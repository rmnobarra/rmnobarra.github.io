---
layout: post
title: "A Golang script to delete namespaces in a Kubernetes cluster using patterns, running as a CronJob."
date: 2024-01-09 16:00:00
categories: [kubernetes, golang]
tags: [kubernetes, golang]
---

## Introduction

Sometimes in a Kubernetes environment, especially in development settings, numerous namespaces are created and deleted throughout the day. Often, a resource that is created is not destroyed after its use, which can unnecessarily overload the Kubernetes cluster.

This project runs a scheduled Golang script to solve this problem.

![namespace cleaner](https://rmnobarradev.blob.core.windows.net/rmnobarradev/ns-resized.png)
## Goals

At the end of this post, you will find:

* A Golang script that removes namespaces in a Kubernetes cluster.
* A Dockerfile to package and run our code.
* A Kubernetes manifest that creates everything we need to run our code."

## main.go

To begin with, let's create our script:

```go

package main

import (
	"context"
	"fmt"
	"os"
	"regexp"

	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/rest"
)

type NamespaceCleaner struct {
	clientSet *kubernetes.Clientset
	regex     *regexp.Regexp
}

func NewNamespaceCleaner(regexStr string) (*NamespaceCleaner, error) {
	config, err := rest.InClusterConfig()
	if err != nil {
		return nil, fmt.Errorf("error building kubernetes in-cluster config: %v", err)
	}

	clientSet, err := kubernetes.NewForConfig(config)
	if err != nil {
		return nil, fmt.Errorf("error creating kubernetes clientset: %v", err)
	}

	regex, err := regexp.Compile(regexStr)
	if err != nil {
		return nil, fmt.Errorf("error compiling regex pattern: %v", err)
	}

	return &NamespaceCleaner{
		clientSet: clientSet,
		regex:     regex,
	}, nil
}

func (n *NamespaceCleaner) CleanupNamespaces(ctx context.Context) error {
	namespaces, err := n.clientSet.CoreV1().Namespaces().List(ctx, metav1.ListOptions{})
	if err != nil {
		return fmt.Errorf("error getting namespaces list: %v", err)
	}

	for _, namespace := range namespaces.Items {
		if n.regex.MatchString(namespace.Name) {
			if err := n.clientSet.CoreV1().Namespaces().Delete(ctx, namespace.Name, metav1.DeleteOptions{}); err != nil {
				return fmt.Errorf("error deleting namespace %s: %v", namespace.Name, err)
			}
		}
	}
	return nil
}

func main() {
	regexStr := os.Getenv("NAMESPACE_SELECTOR")
	if regexStr == "" {
		fmt.Println("NAMESPACE_SELECTOR environment variable not set")
		return
	}

	cleaner, err := NewNamespaceCleaner(regexStr)
	if err != nil {
		fmt.Printf("error creating namespace cleaner: %v\n", err)
		return
	}

	ctx := context.Background()

	if err := cleaner.CleanupNamespaces(ctx); err != nil {
		fmt.Printf("error cleaning up namespaces: %v\n", err)
	}
}

```

## Dockerfile

And now Dockerfile:

```Dockerfile

FROM golang:1.20 AS build

WORKDIR /app

COPY . .

RUN go mod tidy
RUN go mod download

RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o namespace-cleaner main.go

FROM alpine:3.13

RUN apk --no-cache add ca-certificates

COPY --from=build /app/namespace-cleaner /namespace-cleaner

CMD ["/namespace-cleaner"]

```

## Kubernetes manifests

This manifest creates a namespace called 'namespace-cleaner', a ClusterRole, a ClusterRoleBinding, a ServiceAccount, and a CronJob that runs our container image.

*Note: Replace the 'image' and 'NAMESPACE_SELECTOR' values with your own.*

```yaml

apiVersion: v1
kind: Namespace
metadata:
  name: namespace-cleaner
  labels:
    name: namespace-cleaner

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: namespace-cleaner
rules:
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["list", "delete"]

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: namespace-cleaner
  namespace: namespace-cleaner

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: namespace-cleaner
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: namespace-cleaner
subjects:
- kind: ServiceAccount
  name: namespace-cleaner
  namespace: namespace-cleaner

---

apiVersion: batch/v1
kind: CronJob
metadata:
  name: namespace-cleaner
  namespace: namespace-cleaner
spec:
  schedule: "0 * * * *" # runs every hour
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: namespace-cleaner
          containers:
          - name: namespace-cleaner
            image: rmnobarra/namespace-cleaner:1.0 # replace to your Docker image name
            imagePullPolicy: Always
            command: ["/namespace-cleaner"]
            env:
            - name: NAMESPACE_SELECTOR
              value: "dev-*" # replace to your desired value
          restartPolicy: OnFailure


```

## Build

Docker build, tag and push:

```bash
docker build -t namespace-cleaner:1.0 .

docker tag namespace-cleaner:1.0 rmnobarra/namespace-cleaner:1.0

docker push rmnobarra/namespace-cleaner:1.0
```

## Test

I will use a [kind](https://kind.sigs.k8s.io/) cluster for testing. If you're not familiar with kind, now is a good time to learn about it:

```terminal
kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

## Create a few namespaces

Let's create several namespaces using this for loop: for ns in dev-1 dev-2 dev-3 dev-4; do kubectl create namespace $ns; done

```bash
for ns in dev-1 dev-2 dev-3 dev-4; do
  kubectl create namespace $ns
done
namespace/dev-1 created
namespace/dev-2 created
namespace/dev-3 created
namespace/dev-4 created

```

Let's verify the number of namespaces we currently have.

```bash
kubectl get namespaces
NAME                 STATUS   AGE
default              Active   6m9s
dev-1                Active   112s
dev-2                Active   112s
dev-3                Active   112s
dev-4                Active   111s
kube-node-lease      Active   6m9s
kube-public          Active   6m9s
kube-system          Active   6m9s
local-path-storage   Active   6m3s
```

## create kubernetes manifest

I'll configure a CronJob to run every minute using the rmnobarra/namespace-cleaner:1.0 image, along with the 'dev-*' pattern specified in the namespace-selector.yaml file:

```yaml

apiVersion: v1
kind: Namespace
metadata:
  name: namespace-cleaner
  labels:
    name: namespace-cleaner

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: namespace-cleaner
rules:
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["list", "delete"]

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: namespace-cleaner
  namespace: namespace-cleaner

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: namespace-cleaner
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: namespace-cleaner
subjects:
- kind: ServiceAccount
  name: namespace-cleaner
  namespace: namespace-cleaner

---

apiVersion: batch/v1
kind: CronJob
metadata:
  name: namespace-cleaner
  namespace: namespace-cleaner
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: namespace-cleaner
          containers:
          - name: namespace-cleaner
            image: rmnobarra/namespace-cleaner:1.0
            imagePullPolicy: Always
            command: ["/namespace-cleaner"]
            env:
            - name: NAMESPACE_SELECTOR
              value: "dev-*"
          restartPolicy: OnFailure
```

## Install kubernetes manifest

Now, it's time to deploy the Kubernetes manifest:

```bash
kubectl apply -f namespace-selector.yaml

namespace/namespace-cleaner created
clusterrole.rbac.authorization.k8s.io/namespace-cleaner created
serviceaccount/namespace-cleaner created
clusterrolebinding.rbac.authorization.k8s.io/namespace-cleaner created
cronjob.batch/namespace-cleaner created

```

## Check cronjob 

Let's check if all resources were successfully installed.

```bash
kubectl get all -n namespace-cleaner    
NAME                              SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/namespace-cleaner   * * * * *   False     0        <none>          22s
```

## Check if golang script works properly

After installation, a pod is created according to the schedule.

```bash
kubectl get ev -n namespace-cleaner       
LAST SEEN   TYPE     REASON             OBJECT                                 MESSAGE
119s        Normal   Scheduled          pod/namespace-cleaner-28413773-4v6gg   Successfully assigned namespace-cleaner/namespace-cleaner-28413773-4v6gg to kind-control-plane
119s        Normal   Pulling            pod/namespace-cleaner-28413773-4v6gg   Pulling image "rmnobarra/namespace-cleaner:1.0"
102s        Normal   Pulled             pod/namespace-cleaner-28413773-4v6gg   Successfully pulled image "rmnobarra/namespace-cleaner:1.0" in 17.170590812s (17.170612743s including waiting)
102s        Normal   Created            pod/namespace-cleaner-28413773-4v6gg   Created container namespace-cleaner
101s        Normal   Started            pod/namespace-cleaner-28413773-4v6gg   Started container namespace-cleaner
119s        Normal   SuccessfulCreate   job/namespace-cleaner-28413773         Created pod: namespace-cleaner-28413773-4v6gg
99s         Normal   Completed          job/namespace-cleaner-28413773         Job completed
59s         Normal   Scheduled          pod/namespace-cleaner-28413774-wh692   Successfully assigned namespace-cleaner/namespace-cleaner-28413774-wh692 to kind-control-plane
59s         Normal   Pulling            pod/namespace-cleaner-28413774-wh692   Pulling image "rmnobarra/namespace-cleaner:1.0"
57s         Normal   Pulled             pod/namespace-cleaner-28413774-wh692   Successfully pulled image "rmnobarra/namespace-cleaner:1.0" in 1.403255573s (1.403280088s including waiting)
57s         Normal   Created            pod/namespace-cleaner-28413774-wh692   Created container namespace-cleaner
57s         Normal   Started            pod/namespace-cleaner-28413774-wh692   Started container namespace-cleaner
59s         Normal   SuccessfulCreate   job/namespace-cleaner-28413774         Created pod: namespace-cleaner-28413774-wh692
54s         Normal   Completed          job/namespace-cleaner-28413774         Job completed
119s        Normal   SuccessfulCreate   cronjob/namespace-cleaner              Created job namespace-cleaner-28413773
99s         Normal   SawCompletedJob    cronjob/namespace-cleaner              Saw completed job: namespace-cleaner-28413773, status: Complete
59s         Normal   SuccessfulCreate   cronjob/namespace-cleaner              Created job namespace-cleaner-28413774
54s         Normal   SawCompletedJob    cronjob/namespace-cleaner              Saw completed job: namespace-cleaner-28413774, status: Complete
```

And all namespaces based on the specified pattern were removed!

```bash
kubectl get ns                     
NAME                 STATUS   AGE
default              Active   12m
kube-node-lease      Active   12m
kube-public          Active   12m
kube-system          Active   12m
local-path-storage   Active   12m
namespace-cleaner    Active   2m40s

```

The complete project is available for access here: https://github.com/rmnobarra/namespace-cleaner.

## Conclusion

This example can serve as a template for various applications, whether it's for removing namespaces or any other Kubernetes resources. It's quiet, elegant, and simple. Enjoy.

## Any sugests or doubt? 

Feel free to reach out to me on social media: [twitter](https://twitter.com/rmnobarra)
,[linkedin](https://www.linkedin.com/in/rmnobarra/) and [github](https://github.com/rmnobarra).

You can also email me directly at rmnobarra@gmail.com. 

## Support

Did you really enjoy my content? Consider buying me a coffee through my Bitcoins wallets: 

![Donate with Bitcoin](https://img.shields.io/badge/Donate%20with-Bitcoin-orange)

Bitcoin Wallet:

`bc1quuv5hml9fjkf7azgwkt4xp867pzdwzyga33qmj`

![Bitcoin wallet QRCODE](https://rmnobarradev.blob.core.windows.net/rmnobarradev/bItcoin-address.png)

![Donate through Lightning Network](https://img.shields.io/badge/Donate%20with-Lighting-blue)

Lighting Address: 

`lnbc1pjue6mkpp5yj737e7fm6efhlj6sns42a875pmkencqmvdshf4ghlnntaet5llsdqqcqzzsxqrrsssp5d9hxl686w839qkwmkm0m30cf5gp4gnzxh68kss393xqzlsg0wr3q9qyyssqp3933zc3fg46nk3vafh63r3lqd0jn2p04w5xrz77h33rrr3xm7ckegg6s2ss64g4rf4jg87zdjzkl5tup7umqpghy2qnk65tqzsnplcpwv6z4c`

![Lighting wallet QRCODE](https://rmnobarradev.blob.core.windows.net/rmnobarradev/lighting-address.png)

Bye!