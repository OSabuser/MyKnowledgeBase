#DOCKER #MCU
Following on from the [previous post](https://feabhasblog.wpengine.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/), where we spent time reducing the docker image size, in this post I’d like to cover a couple of useful practices to further improve our docker image:

1. Copying local files rather than pulling from the web
2. Simplifying builds using a multi-stage build

Contents

Toggle

- [Copying in Local Files](https://blog.feabhas.com/2018/10/an-introduction-to-docker-for-embedded-developers-part-5-multi-stage-builds/#Copying_in_Local_Files "Copying in Local Files")
- [Multi-stage build](https://blog.feabhas.com/2018/10/an-introduction-to-docker-for-embedded-developers-part-5-multi-stage-builds/#Multi-stage_build "Multi-stage build")
- [Multi-Stage for Applications](https://blog.feabhas.com/2018/10/an-introduction-to-docker-for-embedded-developers-part-5-multi-stage-builds/#Multi-Stage_for_Applications "Multi-Stage for Applications")
- [Multi-Stage GCC-Arm Image](https://blog.feabhas.com/2018/10/an-introduction-to-docker-for-embedded-developers-part-5-multi-stage-builds/#Multi-Stage_GCC-Arm_Image "Multi-Stage GCC-Arm Image")
- [Summary](https://blog.feabhas.com/2018/10/an-introduction-to-docker-for-embedded-developers-part-5-multi-stage-builds/#Summary "Summary")

## Copying in Local Files

So far, when installing the GCC-Arm compiler, we have pulled it from the web using wget. This technique can suffer from two issues:

1. Web links are notoriously fragile
2. https adds complexity to the packages required with smaller base images such as [Alpine-linux](https://hub.docker.com/_/alpine/)

An alternative approach, especially if you are managing your Dockerfiles in a git repository, is to pull the required file (e.g. gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2) to your local file system and then copy this file into the docker image during the build process.

First we need to download to our local filesystem the version of GCC-Arm we want to use. The latest version can be found at: [https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads)

As of today, the latest version is **7-2018-q2-update**.

I happen to be working on a Mac, but as our image is Linux based, I want to download the Linux 64-bit image `gcc-arm-none-eabi-7-2018-q2-update-linux.tar.bz2`.

Once downloaded, the local (build) directory contains two files:

```
.
├── Dockerfile
└── gcc-arm-none-eabi-7-2018-q2-update-linux.tar.bz2
```

We now modify the Dockerfile to copy from the local file system into our base image using the following command:

```
COPY <local file> <destination>
```

So the command (the trailing ‘.’ is to the current container working directory):

```
COPY gcc-arm-none-eabi-7-2018-q2-update-linux.tar.bz2 .
```

will copy the zip file from our local file system into the container. We can now go ahead and un-tar it and configure it as before, e.g.

```
FROM frolvlad/alpine-glibc:latest

ENV REFRESHED_AT 2018-10-08

# Set up a tools dev directory
WORKDIR /home/dev

COPY gcc-arm-none-eabi-7-2018-q2-update-linux.tar.bz2 .

RUN apk add --no-cache \
    git \
    python \
    scons \
    && apk --update --no-cache add --virtual build-dependencies \
    bzip2-dev \
    tar \
    && tar xf gcc-arm-none-eabi-7-2018-q2-update-linux.tar.bz2 \
    && rm gcc-arm-none-eabi-7-2018-q2-update-linux.tar.bz2 \
    && apk del build-dependencies \
    && rm -rf /home/dev/gcc-arm-none-eabi-7-2018-q2-update/share/doc

# Set up the compiler path
ENV PATH="/home/dev/gcc-arm-none-eabi-7-2018-q2-update/bin:${PATH}"

WORKDIR /usr/project

CMD ["scons"]
```

By using a local file, we can drop openssl, ca-certificates and w3m from `build-dependencies` used in our [Dockerfile in Part 4](https://feabhasblog.wpengine.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/).

## Multi-stage build

In [Part 4](https://feabhasblog.wpengine.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/) we covered various techniques to reduce our original docker image from 2.14GB down to a final image size of 457MB. This involved a couple of key steps:

- Starting with a small base image (Alpine-linux)
- Removing one-time-use packages (e.g. tar, openssl, etc.)
- Delete unnecessary files from any libraries installed (e.g. /docs files from GCC-Arm)

Multi-stage builds simplify this further.

### Multi-Stage for Applications

Multi-stage builds are a relatively new feature requiring [Docker 17.05](https://docs.docker.com/release-notes/docker-ce/#17050-ce-2017-05-04) or higher. The goal is to simplify optimisation of application images. The evolution of multi-stage builds is explained here [https://docs.docker.com/develop/develop-images/multistage-build/](https://docs.docker.com/develop/develop-images/multistage-build/).

As a simple example, say we wanted to *deploy* a hello-world Linux C application in a docker container. Assuming we had a project:

```
.
└── src
    └── main.c
```

I can build and run this locally (on my Mac) using gcc, e.g.

```
$ gcc -o hello src/main.c
$ ./hello 
hello, world!
```

To create a portable docker version we could do the following:

```
FROM gcc:latest

WORKDIR /home/dev
COPY src/ src/

RUN gcc -o hello src/main.c
CMD ["./hello"]
```

Now build the docker image:

```
$ docker build .
Sending build context to Docker daemon  13.82kB
Step 1/5 : FROM gcc:latest
 ---> d4d2a8e7d887
Step 2/5 : WORKDIR /home/dev
 ---> Using cache
 ---> 0cdd3e7ec2a3
Step 3/5 : COPY src/ src/
 ---> Using cache
 ---> cd41796ce86b
Step 4/5 : RUN gcc -o hello src/main.c
 ---> Using cache
 ---> 46aa8aeccb2c
Step 5/5 : CMD ["./hello"]
 ---> Running in 2c2548bc9178
Removing intermediate container 2c2548bc9178
 ---> 384a810730e5
Successfully built 384a810730e5
```

And run it:

```
$ docker run --rm 384a810730e5
hello, world!
```

This *application* can now be executed on any platform supporting docker (e.g. Linux, Mac and Windows-10). We could even deploy it as an application on something such as AWS or Google-Cloud.

But the main issue is that the size of the image is 1.68GB, mainly due to having the GCC toolchain as part of the base image:

```
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
<none>                          <none>              384a810730e5        2 minutes ago       1.68GB
```

All we really want is the executable `hello` and any dependent libraries (e.g. glibc).

A multi-stage build allows intermediate docker images to be built, but then offers the functionality to just COPY across any files from the intermediate image to a new, fresh, image. For example, it would be better to deploy our `hello` application using Alpine-linux as the base image rather than the GCC base image.

A Dockerfile to achieve this looks thus:

```
FROM gcc:latest as builder

WORKDIR /home/dev
COPY src/ src/
RUN gcc -o hello src/main.c

FROM frolvlad/alpine-glibc:latest

WORKDIR /usr/project
COPY --from=builder /home/dev/hello .
CMD ["./hello"]
```

Note initial image is tagged as `builder` (this can be any name, builder is not a required name), this then allows the COPY command to select files from that named image instead of the local file system.

Building the image:

```
$ docker build .
Sending build context to Docker daemon  14.85kB
Step 1/8 : FROM gcc:latest as builder
...
Step 5/8 : FROM frolvlad/alpine-glibc:latest
...
Successfully built e0e1640423f4
```

And running it as before:

```
$ docker run --rm e0e1640423f4
hello, world!
```

achieves the same result. However, if we now compare image sizes we see the newly built image is only **11.3MB**:

```
$ docker image ls
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
<none>                          <none>              e0e1640423f4        8 seconds ago       11.3MB
<none>                          <none>              384a810730e5        14 minutes ago      1.68GB
```

## Multi-Stage GCC-Arm Image

With this new knowledge we can adapt our previous GCC-Arm Dockerfile to utilise a multi-stage build.

As we are going to discard the intermediate image, then we can resort back to a generic image which has the packages we require, e.g. [`ubuntu:latest`](https://hub.docker.com/_/ubuntu/). Using this image doesn’t require any packages installing. This means we can simply un-tar and configure the GCC-Arm cross-complier in the ubuntu intermediate image and then just copy across the complier toolchain.

```
FROM ubuntu:latest as builder

WORKDIR /dependencies
COPY gcc-arm-none-eabi-7-2018-q2-update-linux.tar.bz2 .

WORKDIR /home/dev
RUN tar xf /dependencies/gcc-arm-none-eabi-7-2018-q2-update-linux.tar.bz2
RUN rm -rf gcc-arm-none-eabi-7-2018-q2-update/share/doc

FROM frolvlad/alpine-glibc:latest
RUN apk add --no-cache \
    python \
    scons \
    && apk update

COPY --from=builder /home/dev/ /home/dev/
ENV PATH="/home/dev/gcc-arm-none-eabi-7-2018-q2-update/bin:${PATH}"

WORKDIR /usr/project

CMD ["scons"]
```

So now building, we see:

```
$ docker build .
Sending build context to Docker daemon  100.7MB
Step 1/12 : FROM ubuntu:latest as builder
...
Step 7/11 : FROM frolvlad/alpine-glibc:latest
...
Successfully built 7d65a845d8ff
```

With a final image size of **424MB** (slightly smaller than our previous 457MB):

```
$ docker image ls
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
<none>                          <none>              7d65a845d8ff        23 seconds ago      424MB
```

We can go ahead and tag that and push it up to dockerhub to use in our CI builds.

## Summary

Multi-stage builds have significantly simplified optimising Dockerfiles while keeping them easy to read and maintain. As always there are pros and cons regarding using local files verse pulling from the web. One major benefit of local files, is while experimenting they significantly reduce image build times.

[An Introduction to Docker for Embedded Developers – Part 1 Getting Started](https://feabhasblog.wpengine.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/)  
[An Introduction to Docker for Embedded Developers – Part 2 Building Images](https://feabhasblog.wpengine.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/)  
[An Introduction to Docker for Embedded Developers – Part 3 Cross-Compiling for Cortex-M](https://feabhasblog.wpengine.com/2017/11/introduction-docker-embedded-developers-part-3-cross-compiling-cortex-m/)  
[An Introduction to Docker for Embedded Developers – Part 4 Reducing Docker Image Size](https://feabhasblog.wpengine.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/)

- [About](https://blog.feabhas.com/2018/10/an-introduction-to-docker-for-embedded-developers-part-5-multi-stage-builds/#abh_about)
- [Latest Posts](https://blog.feabhas.com/2018/10/an-introduction-to-docker-for-embedded-developers-part-5-multi-stage-builds/#abh_posts)

[![Niall Cooling](https://secure.gravatar.com/avatar/684c2752487144c7b65738b077b88b54?s=250&d=mm&r=g)](https://www.feabhas.com/ "Niall Cooling")

Co-Founder and Director of Feabhas since 1995.  
Niall has been designing and programming embedded systems for over 30 years. He has worked in different sectors, including aerospace, telecomms, government and banking.  
His current interest lie in IoT Security and Agile for Embedded Systems.

[![Niall Cooling](https://secure.gravatar.com/avatar/684c2752487144c7b65738b077b88b54?s=250&d=mm&r=g)](https://www.feabhas.com/ "Niall Cooling")

[![](https://secure.gravatar.com/avatar/684c2752487144c7b65738b077b88b54?s=150&d=mp&r=g)](https://blog.feabhas.com/author/feabhas/)

##### [Niall Cooling](https://blog.feabhas.com/author/feabhas/)

Co-Founder and Director of Feabhas since 1995.  
Niall has been designing and programming embedded systems for over 30 years. He has worked in different sectors, including aerospace, telecomms, government and banking.  
His current interest lie in IoT Security and Agile for Embedded Systems.