а#DOCKER #MCU 
Docker is a relatively new technology, only appearing just over four years ago. The core building blocks have always been part of Unix; but the significant support, Linux containers (LCX), [first appeared back in 2008](https://searchservervirtualization.techtarget.com/feature/A-brief-history-of-Docker-Containers-overnight-success).

Initially Docker was only supported on Linux, but more recently native support for OSX (my development OS of choice) and Windows (albeit Windows 10 Pro) suddenly opens up some interesting workflow choices.

Contents

Toggle

- [The “What”](https://blog.feabhas.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/#The_%E2%80%9CWhat%E2%80%9D "The “What”")
- [The “Why”](https://blog.feabhas.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/#The_%E2%80%9CWhy%E2%80%9D "The “Why”")
- [Using existing Docker images](https://blog.feabhas.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/#Using_existing_Docker_images "Using existing Docker images")
- [Docker Command Line Search](https://blog.feabhas.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/#Docker_Command_Line_Search "Docker Command Line Search")
- [Pull the Image](https://blog.feabhas.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/#Pull_the_Image "Pull the Image")
- [Running a container](https://blog.feabhas.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/#Running_a_container "Running a container")
- [Compiling a C file](https://blog.feabhas.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/#Compiling_a_C_file "Compiling a C file")
- [Cleaning up](https://blog.feabhas.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/#Cleaning_up "Cleaning up")

- [Summary](https://blog.feabhas.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/#Summary "Summary")

## The “What”

So, first, what is Docker? I’m always trying to find the right words here that does Docker justice but doesn’t over simplify the technology. A one-liner is:

> “A lightweight Virtual Machine”

The danger of this over-simplified statement is the natural follow-on questions trying to compare Docker to, say, a hypervisor technology such as [VirtualBox](https://www.virtualbox.org/).

Another one-liner I try is:

> “It’s like a Linux process with its own file system and network connections”

However, a [fuller description](https://www.infoworld.com/article/3204171/linux/what-is-docker-linux-containers-explained.html) is:

> “Linux containers are self-contained execution environments—with their own, isolated CPU, memory, block I/O, and network resources—that share the kernel of the host operating system. The result is something that feels like a virtual machine, but sheds all the weight and startup overhead of a guest operating system.”

So, Docker allows me to wrap up a program and all its dependences (e.g. python tools, libraries, etc.) into a single, isolated executable environment. The wrapping up of the program and its dependences is called a Docker *image*; when image is executed it runs as a Docker *container*.

## The “Why”

Probably more importantly is *“Why would I use Docker as an Embedded developer?”*. Most of the current Docker development is in the field of [DevOps](https://en.wikipedia.org/wiki/DevOps) called “[microservices](https://en.wikipedia.org/wiki/Microservices)” where Docker is being used to deploy applications. This is, currently, far removed from most embedded systems and I won’t address here. \[ See [Getting started with Docker on your Raspberry Pi](https://blog.hypriot.com/getting-started-with-docker-on-your-arm-device/) if that floats your boat\]

How, then, can Docker help an Embedded developer?

Some of the key benefits are:

- For anyone not developing on Linux it opens up the world of Free Open Source Software (FOSS) that quite often are not available on other platforms (or difficult to install)
- It allows developers to use tools in their local development environment without having to install them (even if there is a build available).
- It allows code to be checked against variants of toolchains without the struggle of tools co-existing
- It ensures all team members are using exactly the same tools and build environment
- It ensures my build server (e.g. Jenkins on Linux) is building against the same tools used in development and vice-versa.
- It allows me to create a virtual TCP/IP network of separate applications on a single machine
- It allows me to experiment with support technology without having pollute my development machine (e.g. run a local nginx HTTP server as a target for my IoT development in a Docker container)

There are plenty more good reasons to look at Docker which I’m sure you’ll start to see as you get use to it.

To get started first we must [install Docker](https://docs.docker.com/engine/installation/) relative to our local operating system (here I have used the Docker Community Edition). Once installed we now have access to a vast array of tools to help during software development.

Before building our own Docker images we’ll look at using a pre-existing one.

## Using existing Docker images

When developing C or C++ code locally on a Mac, the default compile is [clang/LLVM](https://clang.llvm.org/) rather than the GNU Compiler Collection (GCC).

```
$ gcc -v
Configured with: --prefix=/Applications/Xcode.app/Contents/Developer/usr --with-gxx-include-dir=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.12.sdk/usr/include/c++/4.2.1
Apple LLVM version 8.1.0 (clang-802.0.42)
```

In theory, if I stick to “Standard” C++ then any code I develop should build and run on both OSX and Linux without modification (assuming I’m building from source). But, as you can guess, this isn’t always the case. So how can I ensure my codebase will build both locally and remotely?

Of course, I could go ahead and install GCC locally on my development machine. But that means getting IT involved; also, I now must manage two toolchains on my local machine as I still want clang (I find the diagnostic reporting better on clang than gcc). And yet GCC is the default tool used on most projects, therefore it installed on our Linux build server and a standard part of the build workflow.

This is where Docker starts to show its usefulness. Instead of having to battle various IT people to get local and remote installs done, we can utilise a pre-built Docker image that already has GCC installed. This same image can be run both locally on my development machine, by all members of the team on their machines, and remotely on the build server ensuring both builds are using the same configuration and is OS agnostic.

Docker has the concept of official [*Base* images](https://docs.docker.com/engine/userguide/eng-image/baseimages/), such as Ubuntu and Microsoft Windows Server 2016 Core. Further images are then built on these base images, e.g. the official Nginx open source reverse proxy server. Derived images can also come from the community, which typically have more specific tools installed (i.e. *cppcheck*).

In addition, the images have tagged versions, e.g. Ubuntu:14.04, Ubuntu:16.04, nginx:1.12, nginx:1.13, etc. which indicate the containerised tools (e.g. nginx:1.13 supports version 1.13 of nginx). If no tag is present then the *latest* image is always used.

[Dockerhub](https://hub.docker.com/) is a cloud-based repository, that, for among other things, stores Docker images that can be searched. To use existing images you’ll need a Dockerhub account (it’s free) as we’ll need to be logged in to pull images.

There are two ways of looking for an Official GCC Docker image:

- Using a web-browser, log into hub.docker.com and search for gcc
- Use a command line search

### Docker Command Line Search

The quickest and easiest method is using the command line, e.g.

```
$ docker search gcc
```

As of the time of writing, there were 25+ hits for gcc. Choosing “the best” image is based on several criteria, but initially we want to work with “Official” images. We can filter the search results for only official images, e.g.:

```
$ docker search --filter "is-official=true" gcc
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
gcc       The GNU Compiler Collection is a compiling...   211       [OK]
```

As you get to know Docker better it is worth visiting the Dockerhub page for the image, which should show details of the image construction and may have notes on how to use it (which library/gcc helpfully does).

### Pull the Image

To start, we can download the image we want to use from Dockerhub to our local machine:

```
$ docker pull gcc
```

Once downloaded we can check for the image by doing:

```
$ docker image ls
REPOSITORY             TAG               IMAGE ID           CREATED            SIZE
gcc                    latest            855a4f4d1cd9      4 weeks ago        1.64GB
```

You can see this is a large image at 1.64GB; something to factor in when choosing/building images as it may have a knock-on effect to your build servers performance.

### Running a container

Now we have the gcc Docker image locally, we can build code against it. This ensures our code has been checked against the latest GCC compiler on Linux. First off, we verify the compiler installed in the Docker image by simply doing:

```
$ docker run gcc gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/local/libexec/gcc/x86_64-linux-gnu/7.1.0/lto-wrapper
Target: x86_64-linux-gnu
Configured with: /usr/src/gcc/configure --build=x86_64-linux-gnu --disable-multilib --enable-languages=c,c++,fortran,go
Thread model: posix
gcc version 7.1.0 (GCC)
```

The command runs the Docker image “gcc” with the command line command `gcc -v`. The output clearly shows the target build is for *x86\_64-linux-gnu* using GCC v7.1.0.

### Compiling a C file

However, as we are planning to run gcc in a container we must transfer the code we want to compile into the container’s file system. There are a couple of ways of doing this, but, for now, the simplest approach is using Dockers capability to pass (mount) a local directory into the container when we run it.

Assuming we are in the development machines source directory containing the traditional `main.c` file:

```
$ ls
main.c

$ cat main.c
#include <stdio.h>
int main(void) 
{
  puts("Hello from Docker!");
  return 0;
}
```

To run the gcc image as a Docker container:

```
$ docker run -v $(pwd):/usr/src/myapp -w /usr/src/myapp gcc gcc -o hello main.c
```

The `-v $(pwd):/usr/src/app` is mounting the present working directory (pwd) as the `/usr/src/myapp` directory in the container. The `-w /usr/src/myapp` sets the working directory of the container to `/usr/src/myapp` where any commands will be executed. Finally, the `gcc -o hello main.c` compiles `main.c` and creates the executable `hello`. If we now list our local file system we’ll see we now have the additional (executable) file `hello` as well as `main.c`

```
$ ls
hello main.c
```

We can confirm that the executable generated is based on a Linux-x86-64 bit platform, e.g.

```
$ file hello
hello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, not stripped
```

To execute this file we must also run it within the container (it won’t run locally on OSX):

```
$ docker run -v $(pwd):/usr/src/myapp -w /usr/src/myapp gcc ./hello
Hello from Docker!
```

If we build a Makefile, e.g.

```
$ cat Makefile
hello : main.c
   gcc -o hello main.c 
clean :  
   rm hello
```

Then simply we could run make from within the container:

```
$ docker run -v $(pwd):/usr/src/myapp -w /usr/src/myapp gcc make
gcc -o hello main.c
$ docker run -v $(pwd):/usr/src/myapp -w /usr/src/myapp gcc ./hello
Hello from Docker!
$ docker run -v $(pwd):/usr/src/myapp -w /usr/src/myapp gcc make clean
rm hello
```

And that’s all there is too it. You now have the capability to run the latest version of gcc and g++ against any C/C++ codebase without installing the tool. It also means everyone can be using a common tool with having deal with IT departments (never a bad thing).

### Cleaning up

By default, each time `docker run` is executed it leaves the container (predominately the file system) resident on your local machine, e.g.

```
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                 CREATED              STATUS                          PORTS               NAMES
e5964012946a        gcc                 "make"                  5 seconds ago        Exited (0) 3 seconds ago                            mystifying_neumann
2605bb29adb6        gcc                 "make clean"            7 seconds ago        Exited (0) 6 seconds ago                            heuristic_poincare
a22fe8b524ae        gcc                 "make"                  12 seconds ago       Exited (0) 11 seconds ago                           kind_payne
559ac67b4ebb        gcc                 "./hello"               18 seconds ago       Exited (0) 16 seconds ago                           happy_panini
61d18d59e323        gcc                 "gcc -o hello main.c"   30 seconds ago       Exited (0) 29 seconds ago                           lucid_wilson
2b6d472cb596        gcc                 "gcc -v"                About a minute ago   Exited (0) About a minute ago                       condescending_ride
0d6b0ebbea13        gcc                 "gcc -v"                About a minute ago   Exited (0) About a minute ago                       laughing_curie
```

Don’t worry about the container names, these are automatically generate using pseudo-random lookup model – the surnames are notable scientists and hackers!

As you do more with Docker, having the used containers available can be useful as sometimes you’d like to review the process execution (e.g. look at any logs or exit status). The exited container also stores any filesystem changes, which can be commit as a new image. However, in our case once we’ve used the container we no longer needed it (in this model we don’t re-run the container).

To remove an individual container, we use the `docker rm` command, e.g.:

```
$ docker rm 61d18d59e323
61d18d59e323
```

Alternatively, the quickest way to remove all previous Docker containers is to run the slightly convoluted command:

```
$ docker rm $(docker ps -a -q)
e5964012946a
2605bb29adb6
a22fe8b524ae
559ac67b4ebb
2b6d472cb596
0d6b0ebbea13
```

A better approach, for this particular workflow where we’re not saving the containers after use, is to get Docker to automatically delete the container once its finished executing. This is achieved by adding the `--rm` flag to our run command, e.g.

```
$ docker run --rm -v $(pwd):/usr/src/myapp -w /usr/src/myapp gcc make
```

Finally, should we want to reclaim further disk space we can delete the original image we pulled using the `docker rmi` command, e.g.

```
$ docker rmi gcc
Untagged: gcc:latest
Untagged: gcc@sha256:43709e8e28ca7a5dd13c3da8b3bb513819064f5ead3f6bed9c7970c992b0b967
Deleted: sha256:855a4f4d1cd9a1fb403a4ef98b067ef26c36fe96de446b695456fb84d43ce4d6
```

## Summary

Should you have followed along, you now have Docker installed, a Dockerhub account and can run gcc/g++ against any C/C++ code on your development machine. This forms the foundations for all further use of Docker.

In the next article, we shall look at creating our own Docker image so we can use a different build system to make, such as Scons.

