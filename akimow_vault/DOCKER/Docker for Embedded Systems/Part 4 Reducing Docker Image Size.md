#DOCKER #MCU 
In [Part 3](https://feabhasblog.wpengine.com/2017/11/introduction-docker-embedded-developers-part-3-cross-compiling-cortex-m/)  we managed to build a Docker image containing the tools required to compile and link C/C++ code destined for our embedded Arm target system. However, we’ve paid little attention to the size of the image. Doing a quick Docker image listing we can see its grown to a whopping 2.14GB:

```
$ docker image ls
REPOSITORY              TAG     IMAGE ID       CREATED             SIZE
feabhas/gcc-arm-scons   1.0     6187455a4bfe   8 days ago          2.14GB
gcc                     7.2     7d9419e269c3   2 months ago        1.64GB
```

In your day-to-day work the size of a Docker image may not bother you as Docker caches images locally on your machine. But after a while you’ll certainly need to prune them.

Apart from freeing up disk space, why else look to reduce the size of an image?

Contents

Toggle

- [Continuous-Integration (CI)](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Continuous-Integration_CI "Continuous-Integration (CI)")
- [Generating Smaller Images](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Generating_Smaller_Images "Generating Smaller Images")
- [Minimal Base Image](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Minimal_Base_Image "Minimal Base Image")
- [Dependences](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Dependences "Dependences")
- [Scons](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Scons "Scons")
- [Wget](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Wget "Wget")
- [Unzipping/Untarring](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#UnzippingUntarring "Unzipping/Untarring")
- [Running the compiler](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Running_the_compiler "Running the compiler")
- [Installing the packages](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Installing_the_packages "Installing the packages")
- [Build the Alpine Based Image](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Build_the_Alpine_Based_Image "Build the Alpine Based Image")
- [Further Optimisations](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Further_Optimisations "Further Optimisations")
- [Removing one-time-use packages](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Removing_one-time-use_packages "Removing one-time-use packages")
- [Some final tweaks](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Some_final_tweaks "Some final tweaks")
- [Pushing to Dockerhub](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Pushing_to_Dockerhub "Pushing to Dockerhub")
- [Summary](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#Summary "Summary")

## Continuous-Integration (CI)

As previously mentioned, the **overriding** benefit of using a Docker-based build is consistency and repeatability of the build. But, for modern CI to be effective, we want the build/test cycle to be as quick as possible.

A local [provisioned build server](https://en.wikipedia.org/wiki/Provisioning#Server_provisioning) (such as a Linux server running Jenkins) will also cache images after the first build, so less of an issue.

Cloud-based CI services (such as the previously-mentioned Travis-CI, Bitbucket, Gitlabs, etc.) are typically costed on build-minutes for a given period,  e.g. build-minutes per month.  In these cases pulling or building larger images naturally takes longer; and costs more.

## Generating Smaller Images

There is plenty of good guidance around, and I’m sure what I’m doing here can be improved up on. Our basic approach to minimise our image consists of these steps:

1. Start with a minimal Base image
2. Only install what we need
3. Remove anything we only needed to help install what we needed!
4. Reduce the number of [Docker layers](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/)

### Minimal Base Image

Our base image is gcc:7.2 which comes in at 1.64GB. To that we added Scons and the gcc-arm cross compiler, but for cross-compilation we don’t require the host GCC (x86) compiler.

The most widely used minimal image is call [Alpine](https://wiki.alpinelinux.org/wiki/Main_Page). Alpine Linux is a security-oriented, lightweight Linux distribution based on [musl libc](https://www.musl-libc.org/) and [Busybox](https://busybox.net/FAQ.html#whatis).

If we download the latest Docker image of Alpine:

```
$ docker pull alpine
```

and list the Docker images, we can see Alpine is significantly smaller (only 3.97MB) than both gcc:7.2 (1.64GB) and a base image of Ubuntu (123MB):

```
alpine  latest  053cde6e8953    3 weeks ago     3.97MB
ubuntu  latest  20c44cd7596f    12 days ago     123MB
```

We simply can change the first line of our Dockerfile from:

```
FROM gcc:7.2
```

to

```
FROM alpine:3.6
```

All done? Err no…

### Dependences

In our previous Dockerfile we had the following package dependencies:

1. Scons:
- scons and lib32ncurses5
2. gcc-arm-none-eabi
- gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2
- lib32z1 (.bz2 uncompressor)
3. general
- git
- bzr (this was unnecessarily – accidentally included due to an old project!)

As we are intending to use Alpine, we need additional packages that *come for free* with gcc:7.2. This list of available Alpine packages can be found at [https://pkgs.alpinelinux.org/packages](https://pkgs.alpinelinux.org/packages).

It also needs to be noted that Alpine uses the [`apk-tool`](https://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management) package manager, rather than Ubuntu’s [`apt`](https://help.ubuntu.com/lts/serverguide/apt.html).

### Scons

Scons is written in Python, so when using Alpine we explicitly must install the package `python`.

### Wget

The next stage in our Dockerfile build is to grab the `gcc-arm-none-eabi` zipped tarball file from `https://developer.arm.com`. This involved installing the following packages:

- w3m – this gets us access to wget

However, as the site is `https` then we need to access using TLS. This involves installing packages:

- openssl – The toolkit for SSL v2/v3
- ca-certificates – Common CA certificates PEM files (PEM files are the standard format for OpenSSL containing key information)

### Unzipping/Untarring

Previously, we’d installed the package `lib32z1` to support the unzipping part of the untarring of the file `gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2` (a .bz2 file is a file compressed using BZIP2). `lib32z1` is not supported under Alpine, instead we use the package `bzip2-dev` (or we could have used `unzip`, but `bzip2-dev` is 1/5 size).

When trying to untar the unzipped-downloaded file, I kept getting a strange error of:

```
tar: invalid tar magic
```

which I’d never encountered before.

After a bit of Googling I found found a posting on [StackExchange](https://unix.stackexchange.com/questions/302192/how-to-solve-tar-invalid-magic-error-on-linux-alpine) that answered the problem.

As mentioned, Alpine uses Busybox, which itself is commonly used in Embedded Linux builds. Although Busybox supports a version of tar (a smaller, simplified version) it unfortunately doesn’t support all the features required to extract the files from our tarball..

To extract the Arm cross compiler from the zipped-tarball we, therefore, need the following two packages:

- tar
- bzip2-dev

### Running the compiler

Finally, coming to compile the code I ran into another (obvious in hindsight) problem. Alpine is supplies the `musl`standard library (it’s lightweight and fast), in preference to default GNU C library (glibc).

As the compiler is, after all, just an executing program, when running a compilation the executing process requires certain features from the C Standard Library. It turns out that the GCC-Arm complier requires `glibc` be installed.

There are various options to add glibc to Alpine, but the *path-of-least-resitance* is to utilise a base container that someone else had already created. In the end, I opted to change my base image from `alpine` to [`frolvlad/alpine-glibc:latest`](https://hub.docker.com/r/frolvlad/alpine-glibc/). This is an Alpine-based image, only 5MB in size, that contains glibc and enables  
proprietary projects to be compiled against it.

As you can imagine, there was a certain amount of trial-and-error (supported by Google!) to finally get all the needed packages defined.

## Installing the packages

Revisiting out Dockerfile, the top of the file was:

```
FROM gcc:7.2

ENV REFRESHED_AT 2017-11-21

# Set up a tools dev directory
WORKDIR /home/dev

RUN apt-get -qq update \
    && apt-get -y install git scons lib32z1 lib32ncurses5
```

This now becomes:

```
FROM frolvlad/alpine-glibc:latest

ENV REFRESHED_AT 2017-12-01

RUN apk add --update \
    bzip2-dev \
    ca-certificates \
    git \
    openssl \
    python \
    scons \
    tar \
    w3m 
```

You might have spotted that we also have not added the package `lib32ncurses5` in Alpine as the Python install brings in the `ncurses-libs` used by Scons.

## Build the Alpine Based Image

We can now build the Docker image:

```
$ docker build .
```

and inspect the size:

```
$ docker image ls
REPOSITORY  TAG                 IMAGE ID            CREATED              SIZE
<none>      <none>              23e03fd9fb54        About a minute ago   612MB
```

We can now test it against our default project:

```
$ docker run --rm -v $(pwd):/usr/project 23e03fd9fb54
```

This all build successfully, and as a first stab we have reduced the image to nearly 1/4 of the original size.

## Further Optimisations

It would be very easy to stop here, but there are a few good practices we can employ when building our Docker images.

### Removing one-time-use packages

Several of the packages we install (w3m, openssl, ca-certificates, tar and bzip2-dev) are only needed during the installation of GCC-Arm and Scons. Once installed, in our built image we have no reason to use them again.

The akp package manager supports a concept called [*virtual packages*](https://github.com/gliderlabs/docker-alpine/blob/master/docs/usage.md#virtual-packages). This enables you to group together a set of packages and then remove them during the Docker image construction, so they play no part in the final image size.

In the Dockerfile, we collect together all the intermediate packages using the directive `apk --no-cache add --virtual` as a group called `build-dependencies`

```
RUN apk add --no-cache \
    git \
    python3 \
    scons \
    && apk --no-cache add --virtual build-dependencies \
    bzip2-dev \
    ca-certificates \
    openssl \
    tar \
    w3m 

RUN wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/6-2017q2/gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2 \
    && tar xvf gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2 \
    && rm gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2 \
    && apk del build-dependencies
```

Then after we’ve used the files we use the command `apk del` to remove them.

Also, note the use of the [`--no-cache`](https://github.com/gliderlabs/docker-alpine/blob/master/docs/usage.md#disabling-cache) as it saves removing the local cache of packages.

This brings our image size down to a very respectable 586MB.

```
$ docker image ls
REPOSITORY  TAG                 IMAGE ID            CREATED             SIZE
<none>      <none>              dabcc749dc26        4 minutes ago       586MB
```

## Some final tweaks

There is a [**Best Practice**](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/) guide for Dockerfile. Here are a couple of minor improvements:

1. Combine the RUN commands into one
2. Delete unnecessary files from GCC-Arm

Each use of RUN in a Dockerfile creates an [intermediate layer](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/). It is good practice, wherever possible to reduce the number of layers.

As part of the `gcc-arm-none-eabi-6-2017-q2-update-linux` install, there are superfluous files. For example, the project documentation (`gcc-arm-none-eabi-6-2017-q2-update/share/doc`) is approximately 59MB.

Our final Dockerfile becomes:

```
FROM frolvlad/alpine-glibc:latest

ENV REFRESHED_AT 2017-12-01

# Set up a tools dev directory
WORKDIR /home/dev

RUN apk add --no-cache \
    git \
    python \
    scons \
    && apk --update --no-cache add --virtual build-dependencies \
    bzip2-dev \
    ca-certificates \
    openssl \
    tar \
    w3m \
    && wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/6-2017q2/gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2 \
    && tar xvf gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2 \
    && rm gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2 \
    && apk del build-dependencies \
    && rm -rf /home/dev/gcc-arm-none-eabi-6-2017-q2-update/share/doc

# Set up the compiler path
ENV PATH="/home/dev/gcc-arm-none-eabi-6-2017-q2-update/bin:${PATH}"

WORKDIR /usr/project

CMD ["scons"]
```

This brings our final image size to 457MB down from the original 2.14GB.

```
$ docker image ls
REPOSITORY  TAG                 IMAGE ID            CREATED             SIZE
<none>      <none>              9491310da352        6 minutes ago       457MB
```

## Pushing to Dockerhub

After testing it on our default project, we can now rebuild with a tag, test and push to Dockerhub for use with our CI model.

```
$ docker build -t="feabhas/gcc-arm-scons-alpine:1.0" .

$ cd ../blog-test-project
$ docker run --rm -v $(pwd):/usr/project feabhas/gcc-arm-scons-alpine:1.0

$ docker push feabhas/gcc-arm-scons-alpine:1.0
```

Finally we can update our test project’s `bitbucket-pipelines.yml` file to build with our new, leaner, Docker image:

```
image: feabhas/gcc-arm-scons-alpine:1.0

pipelines:
  default:
    - step:
        script: 
          - scons
```

## Summary

As mentioned at the outset, the most compelling reason to start using Docker is consistency of build and test environment.

Spending the time and effort to reduce the image size is worthwhile for automated builds, ultimately reducing costs (the Alpine based image has halved the build time on Bitbucket).

I’m sure the example shown can be further optimised by pruning unnecessary files (especially the GCC-Arm install), please drop me a comment if you spot obvious ones.

Next up; using Docker images in Multi-stage builds…

- [About](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#abh_about)
- [Latest Posts](https://blog.feabhas.com/2017/12/introduction-docker-embedded-developers-part-4-reducing-docker-image-size/#abh_posts)

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