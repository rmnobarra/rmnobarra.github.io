---
layout: post
title: "Some notes about Docker multi stage build"
date: 2024-01-17 10:00:00
categories: [docker]
tags: [docker]
---

## Introduction

Containers are a staple in the cloud-native era, and they result from a series of instructions within a file, namely a Dockerfile.

Typically, to build our containers, we use an instruction in the Dockerfile called “FROM.” This instruction tells Docker which base image will be used to build our application, such as Golang, Java, Python, Linux, etc.

The problem arises when we provide suboptimal instructions to a Dockerfile, which may result in a large container image that is cumbersome to build, push, and load.

A feature that helps solve this issue is called multistage build.

![Multistage docker](https://rmnobarradev.blob.core.windows.net/rmnobarradev/multistage-resized.png)
## Multistage Build

Multistage builds are a Docker capability that allows the building of a container image in multiple stages. Each stage can use a different base image, and only the necessary artifacts from previous stages are carried over to the next stage.

This approach is quite useful for creating lightweight and clean images, optimizing resources like storage and network bandwidth.


## Hands on

Consider a simple Golang "Hello World" application:

```golang
package main

import (
	"fmt"
	"net/http"
)

func helloWorld(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, World!")
}

func main() {
	http.HandleFunc("/", helloWorld)
	http.ListenAndServe(":8080", nil)
}
```

Here is a typical Dockerfile:


```Dockerfile
FROM golang:1.18

WORKDIR /app

COPY . .

RUN go build -o helloworld

CMD ["./helloworld"]
```

Let's build it then:

![Build a image with a typical Dockerfile](https://rmnobarradev.blob.core.windows.net/rmnobarradev/build-dockerfile.png)

Now let’s take a look of image size:

```bash
➜  go-hello-world docker image ls hello_world
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
hello_world   latest    a8a66b650d58   45 minutes ago   972MB
```

Now a multistage build approach

```Dockerfile

FROM golang:1.18 as builder

WORKDIR /app

COPY . .

RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o helloworld .

FROM alpine:latest

WORKDIR /root/

COPY --from=builder /app/helloworld .

CMD ["./helloworld"]
```

Build:

![Build a image with a multistage Dockerfile](https://rmnobarradev.blob.core.windows.net/rmnobarradev/build-dockerfile-multistage.png)

At first glance, comparing the two files, it might seem like the multistage approach combines two Dockerfiles into one, right? In truth, that's exactly what it does, but there's more to it.

In the FROM command, we use an alias, in this case 'builder,' which will be referenced later.

The beauty of multistage builds is highlighted in the COPY line. We copy a binary directly from the previous step (builder) to the current image. This process results in a much more lightweight image, as shown below:

```bash
➜  go-hello-world docker image ls hello_world
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
hello_world   latest    78199c3b8083   58 minutes ago   13.6MB
```

Reducing the size from 972MB to 13.6MB is a tremendous achievement!

## Conclusion

Of course, there are many improvements that can be applied to a typical Dockerfile, but using a multistage approach is an excellent pattern for production. With this method, we don't need to worry about build dependencies or any additional resources for building container images. Everything is contained within the Dockerfile, making it highly repeatable in a CI/CD workflow.

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















