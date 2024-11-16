#DOCKER #MCU
In the [previous posting](https://feabhasblog.wpengine.com/2017/10/introduction-docker-embedded-developers-part-2-building-images) we looked at defining a custom *Dockerfile* where we can add specific tools (and their dependencies). From that we created a Docker image and this allowed us to build C/C++ code in a Docker container, ensuring a consistent build environment.

So far we have to build all our code using the native GCC toolchain which is part of the base Docker image (gcc:7.2). However, I want to be able to build an image I can download and run on a target system (in our case an ARMv7-M, Cortex-M4, STM32F4-based system).

There are three stages required:

1. Create a Docker container with the gcc-arm-embedded compiler installed
2. Have a base “hello world” project for the board
3. A custom Scons file for the cross-build (of course make or CMake could be used here)

Contents

Toggle

- [Pre-built GNU toolchain for Arm Cortex-M](https://blog.feabhas.com/2017/11/introduction-docker-embedded-developers-part-3-cross-compiling-cortex-m/#Pre-built_GNU_toolchain_for_Arm_Cortex-M "Pre-built GNU toolchain for Arm Cortex-M")
- [Install GNU toolchain](https://blog.feabhas.com/2017/11/introduction-docker-embedded-developers-part-3-cross-compiling-cortex-m/#Install_GNU_toolchain "Install GNU toolchain")
- [Build the Docker image](https://blog.feabhas.com/2017/11/introduction-docker-embedded-developers-part-3-cross-compiling-cortex-m/#Build_the_Docker_image "Build the Docker image")
- [Here’s one I prepared earlier…](https://blog.feabhas.com/2017/11/introduction-docker-embedded-developers-part-3-cross-compiling-cortex-m/#Heres_one_I_prepared_earlier%E2%80%A6 "Here’s one I prepared earlier…")
- [Project build](https://blog.feabhas.com/2017/11/introduction-docker-embedded-developers-part-3-cross-compiling-cortex-m/#Project_build "Project build")
- [Using the container](https://blog.feabhas.com/2017/11/introduction-docker-embedded-developers-part-3-cross-compiling-cortex-m/#Using_the_container "Using the container")
- [Bitbucket Pipelines](https://blog.feabhas.com/2017/11/introduction-docker-embedded-developers-part-3-cross-compiling-cortex-m/#Bitbucket_Pipelines "Bitbucket Pipelines")
- [Where next?](https://blog.feabhas.com/2017/11/introduction-docker-embedded-developers-part-3-cross-compiling-cortex-m/#Where_next "Where next?")
- [Summary](https://blog.feabhas.com/2017/11/introduction-docker-embedded-developers-part-3-cross-compiling-cortex-m/#Summary "Summary")

## Pre-built GNU toolchain for Arm Cortex-M

The latest version of the pre-built GNU toolchain for Arm Cortex-M (and Cortex-R) is found [here](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm).

Normally we’d go through the process of selecting the correct package for our development machine (Windows/Mac/Linux), downloading it and installing it (setting up appropriate paths, etc.).

But with Docker we can create a container for this current version and use this to cross-compile our code.

Building on the *Dockerfile* from the previous post (gcc:7.2 + scons) we need too:

1. Download the Linux tarball (tar archive) for the gcc arm cross-complier
2. Extract the code from the tarball
3. Remove the tarball file from the Docker image
4. Set up the PATH to compiler /bin

### Install GNU toolchain

One of the easiest ways to grab a file from a website is to use [GNU Wget](https://www.gnu.org/software/wget/). Helpfully Wget is part of most Linux base distributions so we don’t have to go about installing it. In the following example I pull the tarball into a different directory to the project just to keeps things cleaner.

If extracting at the Linux CLI we’d do the following:

- pull the tarball from the web
- **$ wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/6-2017q2/gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2**
- Untar the file
- **$ tar xvf gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2**
- Remove tarball
- **$ rm gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2**
- Add the compiler path
- **$ PATH=$PATH:/home/dev/gcc-arm-none-eabi-6-2017-q2-update/bin**

Updating the previous *Dockerfile*, we have the following commands:

```
FROM gcc:7.2

ENV REFRESHED_AT 2017-11-21

RUN apt-get update \
    && apt-get -y install git scons bzr lib32z1 lib32ncurses5

# Set up a tools dev directory
WORKDIR /home/dev

# pull the gcc-arm-none-eabi tarball
RUN wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/6-2017q2/gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2 \
    && tar xvf gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2 \
    && rm gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2

# Set up the compiler path
ENV PATH $PATH:/home/dev/gcc-arm-none-eabi-6-2017-q2-update/bin

WORKDIR /usr/project

CMD ["scons"]
```

## Build the Docker image

As in the previous post, let’s build the image (with a different tag):

```
$ docker build -t="feabhas/gcc-arm-scons:1.0" .
```

What you’ll notice this time is it takes significantly more time to build due to pulling the tarball down from the web (and of course your internet speed will dictate that).

During the build, the pull should display something like this:

```
Step 5/8 : RUN wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/6-2017q2/gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2 && tar xvf gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2 && rm gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2
---> Running in 7d9e29641e3e
--2017-11-21 16:39:52-- https://developer.arm.com/-/media/Files/downloads/gnu-rm/6-2017q2/gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2
Resolving developer.arm.com (developer.arm.com)... 52.178.214.248
Connecting to developer.arm.com (developer.arm.com)|52.178.214.248|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-rm/6-2017q2/gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2 [following]
--2017-11-21 16:39:53-- https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-rm/6-2017q2/gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2
Resolving armkeil.blob.core.windows.net (armkeil.blob.core.windows.net)... 191.235.193.40
Connecting to armkeil.blob.core.windows.net (armkeil.blob.core.windows.net)|191.235.193.40|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 100554551 (96M) [application/octet-stream]
Saving to: 'gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2'

0K .......... .......... .......... .......... .......... 0% 1.01M 95s
50K .......... .......... .......... .......... .......... 0% 443K 2m38s
...
...
98150K .......... .......... .......... .......... ....... 100% 4.51M=60s

2017-11-21 16:40:53 (1.60 MB/s) - 'gcc-arm-none-eabi-6-2017-q2-update-linux.tar.bz2' saved [100554551/100554551]
```

Thankfully our internet connection was behaving itself! Now comes the extraction of the toolchain files:

```
gcc-arm-none-eabi-6-2017-q2-update/
gcc-arm-none-eabi-6-2017-q2-update/arm-none-eabi/
gcc-arm-none-eabi-6-2017-q2-update/arm-none-eabi/include/
...
...
gcc-arm-none-eabi-6-2017-q2-update/lib/gcc/arm-none-eabi/6.3.1/lto-wrapper
gcc-arm-none-eabi-6-2017-q2-update/lib/libcc1.so.0.0.0
gcc-arm-none-eabi-6-2017-q2-update/lib/libcc1.so.0
```

then the path setup

```
Step 6/8 : ENV PATH $PATH:/home/dev/gcc-arm-none-eabi-6-2017-q2-update/bin
```

and finally a built image:

```
---> 6187455a4bfe
Successfully built 6187455a4bfe
Successfully tagged feabhas/gcc-arm-scons:1.0
```

## Here’s one I prepared earlier…

We now need a project to test a container build against. Rather than get into the details here of setting up a project I’ll just use an existing one to test the Docker image.

If you want to follow along, the example project can be cloned from [here](https://bitbucket.org/nscooling/blog-test-project/overview)

The project is designed to generate a simple executable for the [STM32F4-Discovery board](https://www.st.com/en/evaluation-tools/stm32f4discovery.html)

## Project build

Assuming you are in the base directory of the project (the one containing the SConstruct file) then we can test locally by using the following command:

```
$ docker run --rm -v $(pwd):/usr/project feabhas/gcc-arm-scons:1.0
```

This should generate the following build output:

```
...
arm-none-eabi-size --format=berkeley build/debug/c-application.elf
text data bss dec hex filename
22792 2544 788 26124 660c build/debug/c-application.elf
arm-none-eabi-objcopy -O ihex build/debug/c-application.elf build/debug/c-application.hex
scons: done building targets.
```

This generated *.elf/.hex* file can be tested either using [QEMU](https://www.qemu.org/) or downloaded to a real target board.

## Using the container

We now have a Docker image that can be run as a container to build code for our target system. This means that anyone with Docker installed can now reproduce the build (there should be no more “*well it builds for me!*” discussions), whether they are running Linux (different flavours), Mac OSX or Windows-10.

However, and more significantly, we can use the Docker image as part of a Continuous Integration (CI) build. Many companies are now running [Jenkins](https://jenkins.io/) locally as a build server; but there are also many good options for running in the cloud. For example, if you’re using [GitHub](https://github.com/), then there is good integration with [Travis-CI](https://travis-ci.org/) .

However, over at Feabhas we use [Bitbucket](https://bitbucket.org/) from Atlassian for our git repository management. One recent (well, 2016) addition to Bitbucket is [Pipelines](https://bitbucket.org/product/features/pipelines) allowing users to build a repository directly within Bitbucket.

### Bitbucket Pipelines

To use Pipelines, we must first push the Docker image to our DockerHub account as before:

```
$ docker push feabhas/gcc-arm-scons:1.0
```

Next we simply need to add a *bitbucket-pipelines.yml* file to our project (if you’re not familiar with *.yml* files, these are based on [YAML](https://yaml.org/) (as in ‘*YAML Ain’t Markup Language*‘ ). This is easily setup through the Bitbucket web interface (I use the default C/C++ option of a *make* pipeline and edit that).

In our YAML file we need to specify:

1. The Docker build image
2. The shell command(s) to execute

This makes our YAML file very simple (for now):

```
image: feabhas/gcc-arm-scons:1.0

pipelines:
    default:
        - step:
            script:
                - scons
```

This tells Bitbucket: each time there is a new commit to the repository, to clone the master branch into a new container using the specified image (we don’t have to concern ourselves with mounting volumes as we do when we use Docker locally).

Then the Pipeline tool executes the script (*scons* in our case)

As you can imagine, the pipelines can be configured to do much more than this (branch based build, multi-stage builds, timed-build, etc.), but I’m not intending to cover that here.

The major benefit of this is you will quickly find if there happened to be any missing dependences, because Docker working locally has different permissions and access to our underlying file system than a server-based build may have.

In addition, assuming the build is successful you get a nice green banner[![](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2017/11/pipeline.jpeg?resize=640%2C83&ssl=1)](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2017/11/pipeline.jpeg?ssl=1)

I’ve found that if a project builds using Pipelines the configuration is pretty rock-solid.

## Where next?

In my previous post, the backlog items were:

1. Build a Docker image using gcc-arm-embedded for cross-development to Cortex-M
2. Reduce the overall image size
3. Automate the build of the image via Github

Okay, so I used Bitbucket instead of GitHub/Travis-CI, but I can assure you it is a very similar setup but using a *.travis.yml* file for configuration. It is also worth looking at [GitLabs](https://gitlab.com/) and their CI builds using (unsurprisingly) a *.gitlab-ci.yml* configuration file. There are also options to use other services, such as AWS, to set up your own build servers.

The backlog, however is now:

1. Reduce the overall image size
2. Automating Docker image builds
3. Multi-stage builds
4. Testing embedded code using QEMU Docker image

