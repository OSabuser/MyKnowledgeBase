> “Our highest priority is to satisfy the customer  
> through early and continuous delivery  
> of valuable software.”
> 
> Agile Manifesto, [https://agilemanifesto.org/principles.html](https://agilemanifesto.org/principles.html)

It is interesting to see that modern tools and agile development workflows are getting more and more into the embedded world. CI/CD is a strategy where code changes to an application get automatically integrated, tested and released automatically into a production environment.

![](https://mcuoneclipse.com/wp-content/uploads/2023/10/vs-code-with-ci-cd.png?w=1024)

VS Code with CI/CD

## Outline

In this article I’ll show how to set a CI/CD pipeline, with the example of the NXP LPC55S16-EVK, using Visual Studio Code, Docker and GitHub actions.

I show how to build a simple ‘pipeline’ which gets triggered by a git action, to build the software and to release the firmware binary on GitHub.

For this, I’m using the [NXP LPC55S16-EVK](https://mcuoneclipse.com/2021/12/30/lorawan-with-nxp-lpc55s16-and-arm-cortex-m33/) board, using the NXP MCUXpresso SDK with CMake and VS Code. You can find all the sources on [GitHub](https://github.com/ErichStyger/MCUXpresso_LPC55S16_CI_CD).

![](https://mcuoneclipse.com/wp-content/uploads/2023/10/lpc55s16-evk.jpeg?w=1024)

NXP LPC55S16-EVK

## What is CI/CD?

CI/CD (**C**ontinuous **I**ntegration/**C**ontinuous **D**elivery) means automating the development process from adding changes, to tests up to delivery of the software product.

CI/CD is an important part of DevOps, a combination of development (dev) and operations (ops), where people are working together to build and deliver products.

![](https://mcuoneclipse.com/wp-content/uploads/2023/10/devops.png?w=1024)

DevOps chain (Source: [Wikipedia](https://en.wikipedia.org/wiki/DevOps_toolchain))

Continuous Integration (CI) means that developers check-in their changes into a version control system. This then triggers an automated build of the system on a server which then reports the status back. This ensures that the change by the developer does not break anything.

![](https://mcuoneclipse.com/wp-content/uploads/2023/10/continuous_integration.jpg?w=834)

Continuous Integration (Source: [Wikipedia](https://en.wikipedia.org/wiki/Continuous_integration))

The whole process can be seen as a ‘pipeline’ of automated steps:

![](https://mcuoneclipse.com/wp-content/uploads/2023/10/ci_cd_pipeline.png?w=1024)

CI/CD Pipeline

The idea is to ensure that software can be released at and time, ideally in a fully automated way.

## Software and Tools

Key component of the workflow is a version control system (e.g. git). It requires a way of automatically build the software (command line build tools, cmake/make/ninja, …) and a testing framework ([CTest](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Testing%20With%20CMake%20and%20CTest.html), [unity](http://www.throwtheswitch.org/unity), …), plus a controlled build environment (e.g. [docker](https://www.docker.com/)). [VS Code](https://mcuoneclipse.com/2023/08/25/vs-code-virtual-environments-for-embedded-development-with-conda/) actually is only used here because it nicely integrates with git, cmake and docker through extensions: any other editor will do it too.

So here is what I use in this article:

- Visual Studio Code: [VS Code: Installation](https://mcuoneclipse.com/2023/08/05/vs-code-ide-installation/)
- NXP MCUXpresso SDK: [https://mcuxpresso.nxp.com/en/welcome](https://mcuxpresso.nxp.com/en/welcome)
- CMake, Ninja, Make: [VS Code: MCUXpresso Installer](https://mcuoneclipse.com/2023/08/11/vs-code-mcuxpresso-installer/)
- Docker: [https://www.docker.com/](https://www.docker.com/)
- GitHub: [https://github.com/](https://github.com/)

As project I’m using a ‘blinky’ for the LPC55S16-EVK board. You can find the project on GitHub here: [https://github.com/ErichStyger/MCUXpresso\_LPC55S16\_CI\_CD](https://github.com/ErichStyger/MCUXpresso_LPC55S16_CI_CD)

![](https://mcuoneclipse.com/wp-content/uploads/2023/10/vs-code-with-ci-cd.png?w=1024)

## Using Docker

We are going to use docker both on a GitHub server. It makes sense to test out things locally first, so we have to install it locally too. I have installed Docker Desktop for Windows: [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/), but using it on the command line.

Docker uses ‘images’ (think about a blue-print) and ‘container’ (think about a kind of virtual machine). To create an image, we need to create a file with the instructions how to build it.

The next steps go through the process of creating an image and a container, plus inspecting it.

In the ‘blinky’ project root, I have such a file, named `Dockerfile`:

```
# Fetch ubuntu image
FROM ubuntu:22.04

# Install prerequisites
RUN \
    apt update && \
    apt install -y cmake gcc-arm-none-eabi libnewlib-arm-none-eabi build-essential ninja-build
    
# create a directory for the project
RUN \
    mkdir -p /project/
    
# Copy project sources into image
COPY CMakeLists.txt             /project/
COPY arm-none-eabi-gcc.cmake    /project/
COPY sdk                        /project/sdk/
COPY src                        /project/src/
COPY McuLib                     /project/McuLib/

# Build project
RUN \
    cd /project && \
    cmake -G"Ninja" . -B build && \
    cmake --build build
    
# Command that will be invoked when the container starts
ENTRYPOINT ["/bin/bash"]
```

What it does is:

1. Use an Ubuntu ‘base’ image
2. Perform an update and install the necessary tools (cmake, gcc for ARM Embedded, libraries, build tools, ninja)
3. Create a directory in the image for the project
4. Copy the project source files and CMake files into that project folder
5. Initialize the project (for using Ninja) and build the project
6. Specify that the bash shell is used

To create the docker image, open a console/shell (e.g. in VS Code) and ‘cd’ to where the `Dockerfile` is located.

Run the following command to build the image:

```
docker build -t lpc55s16-image .
```
![](https://mcuoneclipse.com/wp-content/uploads/2023/10/docker-image-building.png?w=1024)

Docker Image building

The option ‘-t’ tags the image with a name, and ‘.’ uses the Dockerfile in the current directory.

**Hint**: To view the available images, use:

```
docker images
```

**Hint**: To delete an image:

```
docker rmi lpc55s16-image
```

To create a container from our image:

```
docker create -i -t --name lpc55s16-container lpc55s16-image
```

‘-i’ creates a container for the interactive mode, ‘-t’ adds a pseudo-terminal, and ‘–name’ gives the container a name.

**Hint**: to list the available container:

```
docker container ls -a
```

**Hint**: to remove the container:

```
docker rm lpc55s16-container
```

**Hint**: to copy a file from the host to the container, use the following

```
docker cp <filename> <container>:<pathInContainer>
```

Now lets start the container and log into it:

```
docker start -i lpc55s16-container
```
![](https://mcuoneclipse.com/wp-content/uploads/2023/10/running-container.png?w=881)

Running the container

Here I can inspect the files and see that the project has been built, with the expected output files in the build folder. Later we want to run docker on GitHub, let it build the output files and publish it as a release.

To leave the container, use:

```
exit
```

So now we have the docker environment, the next step is to build it on the GitHub server using GitHub Actions.

GitHub offers the a workflow for ‘actions’: for example it can trigger an event if someone pushes new content or pushes a tag to the repository. And this is what I want here: If I push a tag labeled e.g. “v1.0.0” then it shall create an event to build the project and publish the resulting binary on GitHub as a release.

To use Github workflows, I need to have a special directory on GitHub/in my project: .github\\workflows wehre I put in my actions as YAML files:

![](https://mcuoneclipse.com/wp-content/uploads/2023/10/github-workflow-directory.png?w=336)

GitHub Workflow Directory

Into that folder, I an put workflow files with a .yml extension. The `deploy.yml` has the following content:

```
# Name of the workflow
name: Deploy new version

# Define the trigger event(s)
# Only deploy when a new tag is pushed ('push:tags:') or manually (with 'workflow_dispatch:')
# If pushing tag, it has to match "v*.*.*"
on:
    workflow_dispatch:
    push:
        tags:
          - "v*.*.*"
          
# Must match the project() name in CMakeLists.txt, variable used below to copy .hex file
env:
    APP_NAME: LPC55S16_Blinky
    
# Allow this workflow to write back to the repository
permissions:
    contents: write

# Jobs run in parallel by default, each runs steps in sequence
jobs:
  # Build binary and send to releases
  build-and-deploy:
    runs-on: ubuntu-latest
    name: Build and deploy
    steps:
      - name: Check out this repository
        uses: actions/checkout@v3

      - name: Build Docker image
        run: docker build -t lpc55s16-image .
        
      - name: Create Docker container
        run: docker create --name lpc55s16-container lpc55s16-image
        
      - name: Copy out Intel Hex file
        run: docker cp lpc55s16-container:/project/build/${APP_NAME}.hex ./${APP_NAME}.hex
        
      - name: Put environment variable into the env context
        run: echo "app_name=$APP_NAME" >> $GITHUB_ENV
        
        # for the push, we need the tag! This step is skipped if we run it manually
      - name: Push to release
        uses: softprops/action-gh-release@v1
        if: startsWith(github.ref, 'refs/tags/')
        with:
            files: ${{ env.app_name }}.hex
            body_path: CHANGELOG.md
```

The workflow has a single job (build-and-deploy). It is possible to run multiple jobs in parallel and for this it runs on a Ubuntu machine (runs-on). It sets an environment variable with the name of the application, so it can be used later during pushing to the ‘release’ section of the repository.

The push to the release section is made with an open source GitHub Action (action-gh-release, [https://github.com/softprops/action-gh-release](https://github.com/softprops/action-gh-release)): It uses the git tag to append to the CHANGELOG.md file and publishes the artifacts.

Because the action has the ‘workflow\_dispatch’ trigger specified, I can run the workflow manually too:

![](https://mcuoneclipse.com/wp-content/uploads/2023/10/run-workflow-manually.jpg?w=886)

Run Workflow manually

To trigger the workflow, I push a new tag with a version, for example:

```
git tag v0.0.1
git push origin v0.0.1
```

On GitHub then I can monitor the running action:

![](https://mcuoneclipse.com/wp-content/uploads/2023/10/running-github-action-1.png?w=1024)

running GitHub Action

Clicking on the action I can get more details:

![](https://mcuoneclipse.com/wp-content/uploads/2023/10/action-details.jpg?w=1024)

Action Details

It has pushed the assets to the ‘Releases’ and added an entry to the Change-Log:

![](https://mcuoneclipse.com/wp-content/uploads/2023/10/changelog-and-assets.jpg?w=908)

New release

Congratulations, you have completed a pipeline :-).

## Summary

It is not that hard to build a CI/CD pipeline even for an embedded application, with the help of GitHub actions and docker. VS Code can be nicely used for the task, as it comes with the necessary support for CMake, Docker and GitHub. The presented framework can be extended with additional things like static code analysis or automated testing.

Happy integrating and delivering 🙂

### Links

- Project on GitHub: [https://github.com/ErichStyger/MCUXpresso\_LPC55S16\_CI\_CD](https://github.com/ErichStyger/MCUXpresso_LPC55S16_CI_CD)
- Wikipedia DevOps: [https://en.wikipedia.org/wiki/DevOps\_toolchain](https://en.wikipedia.org/wiki/DevOps_toolchain)
- Wikipedia Continuous Integration: [https://en.wikipedia.org/wiki/Continuous\_integration](https://en.wikipedia.org/wiki/Continuous_integration)
- Docker: [https://www.docker.com/](https://www.docker.com/)
- Earlier series about VS Code: [Consolidating with VS Code](https://mcuoneclipse.com/2023/08/01/consolidating-with-vs-code/)
- Getting started with docker: [https://www.digikey.ch/en/maker/projects/getting-started-with-docker/aa0d4c708c274ffd975f3b427e5c0ce6](https://www.digikey.ch/en/maker/projects/getting-started-with-docker/aa0d4c708c274ffd975f3b427e5c0ce6)
- Continuous Deployment Using Docker and Github Actions: [https://www.digikey.ch/en/maker/projects/continuous-deployment-using-docker-and-github-actions/d9d18e19361647dbb49070ce6f96c2ea](https://www.digikey.ch/en/maker/projects/continuous-deployment-using-docker-and-github-actions/d9d18e19361647dbb49070ce6f96c2ea)
- [A Modern C Development Environment | Interrupt (memfault.com)](https://interrupt.memfault.com/blog/a-modern-c-dev-env?utm_campaign=Interrupt%20Blog&utm_medium=email&_hsmi=269774827&_hsenc=p2ANqtz--DZCtXhgZ9OkTAB-VZEebBJPL08GCSX1CRtsJm4s7CMaYpZqxtMuqSsCoLk3UBzf5-CRuAm9smBgz7BuvOuG41SUS7_A&utm_content=269774827&utm_source=hs_email)
- Unity: [https://github.com/ThrowTheSwitch/Unity/blob/master/docs/UnityGettingStartedGuide.md](https://github.com/ThrowTheSwitch/Unity/blob/master/docs/UnityGettingStartedGuide.md)
- GitHub actions: [https://github.blog/2021-11-04-10-github-actions-resources-basics-ci-cd/](https://github.blog/2021-11-04-10-github-actions-resources-basics-ci-cd/)
- Solving ‘permission denied’ on config file: [https://dev.to/aileenr/github-actions-fixing-the-permission-denied-error-for-shell-scripts-4gbl](https://dev.to/aileenr/github-actions-fixing-the-permission-denied-error-for-shell-scripts-4gbl)
- GitHub CI/CD: [https://resources.github.com/ci-cd/](https://resources.github.com/ci-cd/)