---
layout: post
title: "Hunderthehood: Whats happened when we starts a pod in a kubernetes cluster?"
date: 2024-01-10 10:00:00
categories: [kubernetes]
tags: [kubernetes]
---

## Before beginning

This post assumes a basic understanding of core Kubernetes concepts, such as kubelet, CRI, and so on. It also assumes the use of Kubernetes version 1.18.2 and containerd 1.3.4.

## Introduction

Starting and killing pods are common events that happen hundreds of times every day in a healthy Kubernetes environment, reminiscent of the Circle of Life in "The Lion King."

The process is as simple as the command kubectl run nginx --image nginx, but what exactly happens when you execute this line? Let's check it out together.

![pod birth](https://rmnobarradev.blob.core.windows.net/rmnobarradev/pod-birth-resized.png)
## Who Really Works Behind the Scenes

When I run kubectl run nginx --image nginx, the kubelet creates a Pod Sandbox using the RunPodSandbox method, a part of the RuntimeService. The purpose is to establish a network connection between one or more containers within a pod.

Kubelet also sends all necessary information about the pod, such as metadata, name, ID, Kubernetes namespaces, and DNS configuration.

After the container runtime interface (containerd in this case) finishes this process, it returns a Pod Sandbox ID to kubelet, which then creates containers in the sandbox.

Moving on, once the container sandbox is complete, kubelet checks if the container image is present on the Kubernetes node. If not, kubelet pulls the image. This step uses parts of the ImageService, specifically the ImageStatus and PullImage methods. To delve a bit deeper, after the container runtime pulls an image, it returns the SHA256 image digest so kubelet can create the container.

The next step involves kubelet creating containers in the sandbox using the provided data and the CreateContainer method of RuntimeService. Finally, kubelet provides all information regarding the container, such as sandbox ID, container image digest, commands, arguments, environment variables, volume mounts, and so on. In the meantime, the container runtime generates a container ID and sends it to kubelet.

Finally, kubelet starts the container using the StartContainer method of the RuntimeService, utilizing the container ID received from the container runtime.

From here, a result like the following is generally observed:

```terminal
kubectl get pods -n g04 -o wide
NAME       READY   STATUS    RESTARTS   AGE     IP           NODE                 NOMINATED NODE   READINESS GATES
backend    1/1     Running   0          3m13s   10.244.0.5   kind-control-plane   <none>           <none>
```

And thus, the rest becomes history.

## Some Notes

ImageService and RuntimeService are specifications of the [Open Container Initiative](https://opencontainers.org/).

For more details about specs, they can be accessed here: https://github.com/opencontainers.

I aim to generate a visual interaction with all these pieces using a follow sequence diagram:

![Sequence diagram](https://rmnobarradev.blob.core.windows.net/rmnobarradev/sequence-kubernetes-diagram.png)

## Conclusion

Even simple commands often abstract very complex processes and iterations, reducing cognitive load and effort. This leaves us, the mortal sysadmins, free to carry out our daily tasks, which might include tweet gossips on twitter, playing some video games (league of legends addict here, lol), and so on. 
However, understanding deeply how things really work helps us gain context to make informed choices in cloud architecture and beyond.

## Any sugests or doubt? 

Feel free to reach out to me on social media: [twitter](https://twitter.com/rmnobarra)
,[linkedin](https://www.linkedin.com/in/rmnobarra/) and [github](https://github.com/rmnobarra).

You can also email me directly at rmnobarra@gmail.com. 

## Support

Did you really enjoy my content? Consider buying me a coffee through my Bitcoin wallet: 

Bitcoin Wallet: `3GzvpHGd22HDbpnaZjU57jFT6ptXqNrGmT`

[![Donate with Bitcoin](https://img.shields.io/badge/Doar%20com-Bitcoin-orange)](bitcoin:3GzvpHGd22HDbpnaZjU57jFT6ptXqNrGmT)
Bye!