---
layout: post
title: "Understanding Pod Security Admission - PSA"
date: 2024-01-24 10:00:00
categories: [kubernetes]
tags: [kubernetes]
---

## Introduction

Running a container inside a pod without restrictions or isolation in a production environment can be dangerous. Fortunately, Kubernetes offers many ways to achieve isolation and governance within a cluster. There are more complex and flexible options (like OPA), and there are also simpler methods that I believe are good starting points, such as Pod Security Admission (PSA), which fits this description.

![psa](https://rmnobarradev.blob.core.windows.net/rmnobarradev/psa-resized.png)

## Pod Security Admission (PSA)

PSA is about standards, and these standards are defined by Pod Security Standards (PSS).

A PSS defines a set of policies ranging from highly restrictive to highly permissive.

The beauty of PSA is that it's very easy to enable in a cluster. PSA has been enabled by default since Kubernetes version 1.23. To enable it, you simply add a label to a namespace, and all Pods in that namespace must follow the standards in order to execute.

The label is composed of three elements: a prefix, a mode, and a level. The prefix consistently employs the hard-coded value 'pod-security.kubernetes.io', followed by a slash. The mode specifies how a violation is handled. Lastly, the level establishes the extent of security standards to be followed. 

An example of such a label might appear as follows:

```yaml
metadata:
  labels:
    pod-security.kubernetes.io/enforce: restricted
```

Dig a little deep, we have:

### prefix (simple as that):

```yaml
pod-security.kubernetes.io
```

### Mode

| Mode | Description |
|----------|----------|
| enforce  | Policy violations will cause the pod to be rejected.  |
| audit  | Policy violations will trigger the addition of an audit annotation to the event recorded in the audit log, but are otherwise allowed.  |
| warn  | Policy violations will trigger a user-facing warning, but are otherwise allowed.  |

Offical doc [here](https://kubernetes.io/docs/concepts/security/pod-security-admission/)

### Level

| Profile | Description |
|----------|----------|
|Privileged | Unrestricted policy, providing the widest possible level of permissions. This policy allows for known privilege escalations.  |
| Baseline | Minimally restrictive policy which prevents known privilege escalations. Allows the default (minimally specified) Pod configuration.  |
| Restricted	 | Heavily restricted policy, following current Pod hardening best practices.
 |

Priviled: Fully and wide and complety open
Baseline: Policy made to make a psa adoption painless
Restricted: High secure, less privilege approach

More detail in [official documentation](https://kubernetes.io/docs/concepts/security/pod-security-standards/)

## Hands on

Let's begin by creating a namespace as follows:

```yaml
namespace.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: psa
  labels:
    pod-security.kubernetes.io/enforce: restricted
```
```bash
➜  k8s kubectl apply -f namespace.yaml                                                                            
namespace/lab-psa created
➜  k8s kubectl describe ns lab-psa    
Name:         lab-psa
Labels:       kubernetes.io/metadata.name=lab-psa
              pod-security.kubernetes.io/enforce=restricted
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
```


And a regular pod:

```yaml
podv1.yaml

apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: psa
spec:
  containers:
    - image: busybox:1.35.0
      name: busybox
      command: ["sh", "-c", "sleep 1h"]
```
```bash
➜  k8s kubectl apply -f podv1.yaml    
Error from server (Forbidden): error when creating "podv1.yaml": pods "busybox" is forbidden: violates PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "busybox" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "busybox" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "busybox" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "busybox" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
```

Totally busted!


## Let's fix it

A secure pod's manifest:

```yaml
podv2.yaml

apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: psa
spec:
  containers:
    - image: busybox:1.35.0
      name: busybox
      command: ["sh", "-c", "sleep 1h"]
      securityContext:
        allowPrivilegeEscalation: false
        capabilities:
          drop: ["ALL"]
        runAsNonRoot: true
        runAsUser: 2000
        runAsGroup: 3000
        seccompProfile:
          type: RuntimeDefault
```
```bash
➜  k8s kubectl apply -f podv2.yaml
pod/busybox created
➜  k8s kubectl get pods -n lab-psa         
NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          8s
```

There is a healthy and active pod in the 'lab-psa' namespace, operating under the strict rules of a PSA.

## Conclusion

PSA has been enabled by default since Kubernetes version 1.23. It's really simple to activate and adopt, allowing for the implementation of governance mechanisms without complex configurations. It can either block or just log any violations.

Sadly, it's not possible to write custom rules or change any aspect of the PSA implementation, such as messages and so on. However, it can still be a good starting point for implementing security mechanisms in a Kubernetes environment

## Any sugests or doubt? 

Feel free to reach out to me on social media: [twitter](https://twitter.com/rmnobarra)
,[linkedin](https://www.linkedin.com/in/rmnobarra/) and [github](https://github.com/rmnobarra).

You can also email me directly at rmnobarra@gmail.com. 

## Support

Did you really enjoy my content? Consider buying me a coffee through my Bitcoin wallet: 

Bitcoin Wallet: `3GzvpHGd22HDbpnaZjU57jFT6ptXqNrGmT`

[![Donate with Bitcoin](https://img.shields.io/badge/Doar%20com-Bitcoin-orange)](bitcoin:3GzvpHGd22HDbpnaZjU57jFT6ptXqNrGmT)

Bye!















