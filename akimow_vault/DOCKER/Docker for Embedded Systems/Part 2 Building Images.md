#DOCKER #MCU
In the [initial post,](https://feabhasblog.wpengine.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/) we covered the basics of getting Docker setup and using an official base image for compilation.  
But let’s suppose the base image doesn’t include all the facilities our company uses for development. For example, we have migrated from make files to CMake, but more lately we have taken to using the python-based [Scons build system](https://www.scons.org/) for C/C++ projects.  
The official gcc base image supports make but not Scons or CMake. As before, we can search for a Scons docker image, but will see no official image exists. We now have two choices:

1. Use an unofficial (user) image
2. Build our own

Contents

Toggle

- [Unofficial Images](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Unofficial_Images "Unofficial Images")
- [Build your own image](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Build_your_own_image "Build your own image")
- [Basic workflow](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Basic_workflow "Basic workflow")
- [Dockerfile](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Dockerfile "Dockerfile")
- [Base image](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Base_image "Base image")
- [Updating the base image](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Updating_the_base_image "Updating the base image")
- [Installing packages](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Installing_packages "Installing packages")
- [Working directory](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Working_directory "Working directory")
- [Default behaviour](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Default_behaviour "Default behaviour")
- [Building the Docker image](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Building_the_Docker_image "Building the Docker image")
- [Running the image](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Running_the_image "Running the image")
- [Removing an Image](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Removing_an_Image "Removing an Image")
- [Naming/Tagging an Image](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#NamingTagging_an_Image "Naming/Tagging an Image")
- [Running using the image name](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Running_using_the_image_name "Running using the image name")
- [Making an Image available via Dockerhub](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Making_an_Image_available_via_Dockerhub "Making an Image available via Dockerhub")
- [Testing the image](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Testing_the_image "Testing the image")
- [One final tweak](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#One_final_tweak "One final tweak")
- [Where next?](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Where_next "Where next?")

- [Summary](https://blog.feabhas.com/2017/10/introduction-docker-embedded-developers-part-2-building-images/#Summary "Summary")

## Unofficial Images

Using an unofficial image is a bit of a hit-or-miss affair. A good unofficial image will have a decent set of instructions about its use and how it was built. Unfortunately, most don’t have any information and you can waste a substantial amount of time trawling through the repositories looking for a suitable image. In general, it may be better to build you own image from a base image.  
It is worth noting there are some gems out there in the user community so it still can be worth a cursory browse.

## Build your own image

How easy build your own image will depend on you experience with using a Linux package manager. If you are familiar with adding packages to a running Linux distribution (e.g. apt-get install), then you’ll see creating an image is straightforward. If you’re not very experienced with this aspect of Linux, then it’s worth spending a little bit of time getting a [foundation understanding](https://www.complete-concrete-concise.com/ubuntu-2/ubuntu-11-10-understanding-sudo-apt-get-install) of the Advanced Packaging Tool.  
It is important to understand that different Linux distributions (Ubuntu, Fedora, etc.) each have their own package manager. I still find this one of the most frustrating aspects when working with Linux (Ubuntu uses APT, Fedora uses yum). You will need to know the base Linux distribution and thus the associated package manager to build Docker images.

### Basic workflow

There are two methods for building a Docker image:

1. Build an image locally on your working machine
2. Use a github/bitbucket repository to automatically build in the cloud

Of course, the choice depends on the intended use, maturity of the image, and how widely it is going to be used. Initially, though, while understanding Docker it is better to use a local build.  
For a local build, the steps couldn’t be simpler:

1. Create a “Dockerfile” file that specifies to image contents and behaviour
2. Build the image (docker build)
3. Use the image (docker run)

### Dockerfile

Our Dockerfile specifies the following items:

1. The base image we want to work from
2. Updating the base image
3. The new packages/files that need installing
4. Setting up working directories/path information
5. Default behaviour when run

Defining a “good” Dockerfile to create an image can be, somewhat, of a hit-or-miss affair.  I’m always interested at how other people’s Dockerfiles are constructed to see alternative models. On the whole this has been useful, but occasionally it has taken me down some dead ends.  
In addition, Docker is evolving, so even what I’m saying here may well be obsolete by better practice as soon as I publish. I have found it difficult to find “best practice” mainly because Docker can be used in some many ways. Therefore, I’m constraining myself to looking at using Docker to support embedded cross-development.  
Let’s start by creating a local working directory:

```
$ mkdir gcc7-scons-docker
$ cd gcc7-scons-docker
$ touch Dockerfile
```

Using your favourite text editor open the Dockerfile.

### Base image

For our base image, we can reuse the official gcc image.  
Choosing a base image mainly depends on what it offers, but also, as we’ll see later, the size. For now, we’ll ignore the size issue and stick with the official gcc image.  
The gcc image used previously was referred to as gcc:latest. The first line of our Dockerfile, therefor is:

```
FROM gcc:latest
```

[It is argued](https://medium.com/@mccode/the-misunderstood-docker-tag-latest-af3babfd6375) that using ‘:latest’ can lead to unexpected behaviour (due to build caching) and it is always better using a specific tag, e.g.

```
FROM gcc:7.2
```

### Updating the base image

To update an Ubuntu distribution, at the command line we would typically type:

```
$ apt-get -qq update
```

For Docker, we use the RUN command to perform the same instruction:

```
RUN apt-get -qq update
```

### Installing packages

If we were running on an Ubuntu system and want to [add Scons](https://scons.org/doc/2.5.1/HTML/scons-user.html#idm139837638113984), we’d enter:

```
$ apt-get install scons
```

The equivalent for Docker would be:

```
RUN apt-get -qq update \
    && apt-get install scons
```

However, because of the base image we’re working with Scons has its own dependences that also need installing. Without getting into the detail here, we need the following command:

```
 RUN apt-get -qq update \
    && apt-get -y install git scons bzr lib32z1 lib32ncurses5
```

The “-y” option assume “yes” as answer to all prompts and allows the APT to run non-interactively.

### Working directory

Though not necessary, it is generally cleaner to specify a working directory for building our intended project:

```
WORKDIR /usr/project
```

We will use this to mount our local files, as explained in the previous posting.

### Default behaviour

Finally, when the Docker image is run, we specify the default behaviour (this can be overwritten using the Docker run command).

```
CMD ["scons"]
```

Here we will run the scons against the working directory.

## Building the Docker image

Our (very simple) Dockerfile is thus:

```
FROM gcc:7.2
RUN apt-get -qq update \
    && apt-get -y install git scons bzr lib32z1 lib32ncurses5
WORKDIR /usr/project
CMD ["scons"]
```

To build, from the command line:

```
$ docker build .
```

Once invoked, if the base image is not found locally, then the Docker build will do an automatically pull, e.g.:

```
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM gcc:7.2
7.2: Pulling from library/gcc
219d2e45b4af: Downloading [========>                                          ]  
7.792MB/45.13MB
ef9ce992ffe4: Downloading [=======================================>           ]  
8.732MB/11.1MB
d0df8518230c: Download complete 
38ae21afde7b: Downloading [==>                                                ]  
2.031MB/50.02MB
baa1c2e15c01: Waiting 
2fb261ca7f61: Waiting 
51c5d3631017: Waiting
```

Now, hopefully, you may appreciate why the size of a Docker image can become important. It we are going to use Docker as part of our automated build process (e.g. using Jenkins) then typically the build server is pulling a fresh Docker image for each build (rather than caching them locally). Large images are going to slow down the build process.  
Once complete (assuming no errors) we get:

```
...Successfully built 5f6bc2fd2b55
$
```

\[The image number is specific to the build\]  
We can list our local image and see our newly created Docker image:

```
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              5f6bc2fd2b55        6 minutes ago       1.7GB
gcc                 7.2                 7d9419e269c3        2 weeks ago         1.64GB

Note the size of our image is now 1.7GB, up from the gcc base image of 1.64GB.
```

## Running the image

To test our new image, we need:

1. A C/C++ source file to compile
2. A SConstruct file for Scons to know how to build the executable

We can reuse out main.c from the previous posting. First, Create a SConstruct file:

```
$ touch SConstruct
```

Add the following line(s) to the file (the line beginning with # is a comment):

```
# SConstruct file
Program('hello','main.c')
```

Now we can use scons to build the source file using Docker:

```
$ docker run --rm -v $(pwd):/usr/project 5f6bc2fd2b55
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
gcc -o main.o -c main.c
gcc -o hello main.o
scons: done building targets.

$ ls
SConstruct hello main.c main.o

$ file hello
hello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, 
interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, not stripped
```

To run the executable:

```
$ docker run --rm -v $(pwd):/usr/project 5f6bc2fd2b55 ./hello
Hello from Docker!
```

Finally, to clean-up:

```
$ docker run --rm -v $(pwd):/usr/project 5f6bc2fd2b55 scons -c
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Cleaning targets ...
Removed main.o
Removed hello
scons: done cleaning targets.

$ ls
SConstruct main.c
```

### Removing an Image

We can remove/delete our image thus:

```
$ docker rmi 5f6bc2fd2b55
Deleted: sha256:5f6bc2fd2b551626b2abe68b00efd10edc50902b8b02bb02e9db287c98f1940d
Deleted: sha256:d240f93859515fd5d1722830f5e91735117371250c9742e0faa6464dabd014eb
Deleted: sha256:e0928c91c1dae92e05b94ac9c09188aa7455b5cf6b146b66064fccfe5b9eab66
 
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
gcc                 7.2                 7d9419e269c3        2 weeks ago         1.64GB
```

Note that the official gcc:7.2 image is still cached locally on our machine. This means any further Docker builds using gcc:7.2 as the base image won’t require a pull from Dockerhub (speeding builds up significantly).

## Naming/Tagging an Image

Obviously using the Image ID for running is not the most user-friendly method. By tagging an image we can not only use it via a user-friendly name, we can also make it available through Dockerhub for online builds.  
To tag our image, when building simply add a tag:

```
$ docker build -t="feabhas/gcc7-scons:1.0" .
```

The “-t” (tag) option allows us to name the image. There are three parts to this:

1. Your Dockerhub account name, i.e. “feabhas” in our case
2. The name of this specific image, e.g. “gcc7-scons” (this is used as part of the docker search criteria)
3. A version indicator “1.0”

```
...Successfully built 174882136a48
Successfully tagged feabhas/gcc7-scons:1.0

$ docker image ls
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
feabhas/gcc7-scons   1.0                 174882136a48        10 seconds ago      1.7GBgcc                  7.2                 7d9419e269c3        2 weeks ago         1.64GB
```

### Running using the image name

```
$ docker run --rm -v $(pwd):/usr/project feabhas/gcc7-scons:1.0
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
gcc -o main.o -c main.c
gcc -o hello main.o
scons: done building targets.
```

## Making an Image available via Dockerhub

Now that we have a named image, we can push this up to Dockerhub and make it available to the wider community and build servers.

```
$ docker push feabhas/gcc7-scons:1.0
```

The push refers to a repository \[docker.io/feabhas/gcc7-scons\]  
On Dockerhub we can now see the image:  
[![](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2017/10/initial-dockerhub-1024x627.jpeg?resize=640%2C392&ssl=1)](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2017/10/initial-dockerhub.jpeg?ssl=1)

### Testing the image

Our final check is to remove the local version and run directly from the Dockerhub image.

```
$ docker image ls
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
feabhas/gcc7-scons   1.0                 174882136a48        19 minutes ago      1.7GB
gcc                  7.2                 7d9419e269c3        2 weeks ago         1.64GB

$ docker rmi 174882136a48
Untagged: feabhas/gcc7-scons:1.0
Untagged: feabhas/gcc7-scons@sha256:893b6b552fefb3514a764225adf3ba7f8db957d9f4f130d22cc32d69dbd5cb74
Deleted: sha256:174882136a48b4210ab35fb22c22fc72d0d5cfc60f96a8de71caa92a0c53528e
Deleted: sha256:bc1ff89714a38d5fe38423719aa06061ce0441b96c40cf35286beff3c01b652f
Deleted: sha256:31ad48318eb623cda9000a11bfc1e3960682fe53eb4b024322dc6d4619dfc36b
Deleted: sha256:49c27c6fa481b53e65f69ecb23f728725ed368533110308c5ba74e33db09bc28
Deleted: sha256:618c31b502c9f65389079f33753a7a680d9050d42900d18c2d079b4b82453790

$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
gcc                 7.2                 7d9419e269c3        2 weeks ago         1.64GB

$ docker run --rm -v $(pwd):/usr/project feabhas/gcc7-scons:1.0
Unable to find image 'feabhas/gcc7-scons:1.0' locally
1.0: Pulling from feabhas/gcc7-scons
219d2e45b4af: Already exists 
ef9ce992ffe4: Already exists 
d0df8518230c: Already exists 
38ae21afde7b: Already exists 
baa1c2e15c01: Already exists 
2fb261ca7f61: Already exists 
51c5d3631017: Already exists 
9b83f87aa2aa: Already exists 
7ac0a3b33eba: Already exists 
1b891e2de301: Downloading [======================================>            ]  
25.62MB/32.99MB
116e50fa975f: Download complete
```

Docker uses a form of stacking for the file system, each new addition to the Docker image creates a new layer. When running feabhas/gcc7-scons:1.0, Docker detects there is no local image, but because I still have gcc:7.2 cached locally it only needs to pull the additional packages for the additional layer.

```
$ docker run --rm -v $(pwd):/usr/project feabhas/gcc7-scons:1.0
Unable to find image 'feabhas/gcc7-scons:1.0' locally
1.0: Pulling from feabhas/gcc7-scons
219d2e45b4af: Already exists 
ef9ce992ffe4: Already exists 
d0df8518230c: Already exists 
38ae21afde7b: Already exists 
baa1c2e15c01: Already exists 
2fb261ca7f61: Already exists 
51c5d3631017: Already exists 
9b83f87aa2aa: Already exists 
7ac0a3b33eba: Already exists 
1b891e2de301: Pull complete 
116e50fa975f: Pull complete 
Digest: sha256:893b6b552fefb3514a764225adf3ba7f8db957d9f4f130d22cc32d69dbd5cb74
Status: Downloaded newer image for feabhas/gcc7-scons:1.0
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
gcc -o main.o -c main.c
gcc -o hello main.o
scons: done building targets.
```

## One final tweak

Docker is very smart about the way it caches files for building. However, sometimes we want to flush the build cache and ensure a complete image rebuild. If, for example, I run:

```
$ docker run --rm -v $(pwd):/usr/project feabhas/gcc7-scons:1.0 scons -v
```

I currently get the following output:

```
SCons by Steven Knight et al.: 
  script: v2.5.1.rel_2.5.1:3735:9dc6cee5c168[MODIFIED], 2016/11/03 14:02:02, by bdbaddog on mongodog 
  engine: v2.5.1.rel_2.5.1:3735:9dc6cee5c168[MODIFIED], 2016/11/03 14:02:02, by bdbaddog on mongodog 
  engine path: ['/usr/lib/scons/SCons']
Copyright (c) 2001 - 2016 The SCons Foundation
```

This version of Scons is now ‘baked’ into the Docker image. If there is a new version of Scons it won’t automatically be updated (a good thing from a build stability perspective). I’d probably want to create a new image with an updated tag (e.g. 2.0) to include the updated version.  
There are a number approaches to this, but the one I prefer (thanks to [James Turnbull](https://dockerbook.com/) ) is to add to the Dockerfile the following line after the FROM <base image> line:

```
ENV REFRESHED_AT 2017-10-01
```

This creates a Docker environment variable (there are lots of cool uses for these), but this approach ensures that by changing the date, Docker resets the cache (due to the change to ENV instruction) and runs every subsequent instruction without using the cache. Thus the update command and the package installs are all refreshed.

## Where next?

My outstanding backlog items are:

1. Build a Docker image using gcc-arm-embedded for cross-development to Cortex-M
2. Reduce the overall image size
3. Automate the build of the image via Github

