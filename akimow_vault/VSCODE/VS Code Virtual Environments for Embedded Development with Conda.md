#VSCODE #MCU

Developing for an embedded target means using a certain version of GNU compiler, debugger and other tools. The challenge gets bigger if working with multiple different tool chains and environments.

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/conda-in-vs-code.jpg?w=1024)

Conda in VS Code

**Conda** is package, dependency and environment management tool. While it is heavily used for Python and Data Science development, it is surprisingly working very well to set up and managing environments for embedded development. Conda is great for managing non-Python dependencies and setups.

## Outline

To have a reliable and isolated environment, I need some kind of ‘package manager’ for the development tools. There are many different solutions to the problem, and each approach has its own pros and cons.

- [vcpkg](https://vcpkg.io/) from Microsoft, uses CMake as scripting language
- [xpack](https://xpack.github.io/) (see [Visual Studio Code for C/C++ with ARM Cortex-M: Part 8 – xPack C/C++ Managed Build Tools](https://mcuoneclipse.com/2021/08/09/visual-studio-code-for-c-c-with-arm-cortex-m-part-8-xpack-c-c-managed-build-tools/))
- [Docker](https://www.docker.com/), allows to run applications in container
- [Conda](https://docs.conda.io/) (or Miniconda): from [https://docs.conda.io/projects/conda/en/stable/](https://docs.conda.io/projects/conda/en/stable/)  
*Conda is an open-source package management system and environment management system that runs on Windows, macOS, and Linux. Conda quickly installs, runs, and updates packages and their dependencies. Conda easily creates, saves, loads, and switches between environments on your local computer. It was created for Python programs but it can package and distribute software for any language.*

I have found **vcpkg** not easy to use, and it would add yet another dependency on Microsoft. I have used **xpack** which is a good solution, but you wont find many material online. **Conda** is a tool for managing environments, especially for Python. And **Docker** is a good tool for packaging and deploying applications in containers. I’m using Docker for CI/CD (e.g. GitHub actions), while I have started using Conda for my development environment setup and environment switching.

With Conda I can:

- Create and use **packages** of development tools: cmake, GNU ARM tools, GNU libraries, …
- Create different **virtual environments**: NXP MCUXpresso, STMCube32, Python, Espressif IDF, RPi RP2040, …, each with its own set of tools and environment
- Managing and **switching environments** inside VS Code, down to each project

Compared to other packaging, I feel Conda (or Miniconda) is easier to use and learn, does faster switching and manages disk space well. It is free and [open source](https://github.com/conda/conda) with a [permissible license](https://docs.conda.io/en/latest/license.html).

## Conda installation

I have Miniconda3 installed with the default settings from [https://docs.conda.io/en/latest/miniconda.html](https://docs.conda.io/en/latest/miniconda.html)

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/miniconda-3-installation.jpg?w=499)

Then launch Conda with the special prompt: The prompt shows the environment I’m in:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/conda-base-environment.jpg?w=724)

## Conda Environment and Package Management

You can find many Conda tutorials on the internet, see as well the Conda Cheat-Sheet: [https://conda.io/projects/conda/en/latest/user-guide/cheatsheet.html](https://conda.io/projects/conda/en/latest/user-guide/cheatsheet.html)

Here are a few things to get started with, see as well the [Conda Getting Started](https://conda.io/projects/conda/en/latest/user-guide/getting-started.html#starting-conda):

To show the version number:

```
conda --version
```

To update Conda:

```
conda update conda
```

To create a new environment

```
conda create -n myenv
```

To list the available environments:

```
conda env list
```

Switch to an environment:

```
conda activate myenv
```

To delete an environment, deactivate it or leave it first:

```
conda deactivate
conda env remove --name myenv
```

Search for a package:

```
conda search cmake
```

Install a package with or without a version number requested:

```
conda install cmake
conda install cmake=3.26.4
```

Uninstall a package:

```
conda uninstall cmake
```

Update the environment using a YAML file:

```
conda env update -f requirements.yml
```

## Creating Your Own Conda Packages

With Conda I can create packages of my build tools.

### Conda Build

To be able to create packages, I have to install the **conda-build**. I recommend to create a dedicated ‘build’ environment for building packages:

```
conda create --name build
conda activate build
conda install conda-build
```

For keeping builds locally and not uploading them on a server:

```
conda config --set anaconda_upload no
```

### Package Files

For a new package, create a new directory and place the following three files into it:

- `meta.yaml:` describes the package
- `build.bat`: bulding the package on Windows
- `build.sh`: building the package on Linux

This builds a ‘recipe’ for the package. You can find [my recipes on Github](https://github.com/ErichStyger/mcuoneclipse/tree/master/conda) (work in progress), or for example here: [https://github.com/memfault/conda-recipes/tree/master/gcc-arm-none-eabi](https://github.com/memfault/conda-recipes/tree/master/gcc-arm-none-eabi)

To build the package, cd into that folder with the meta.yaml and build it:

```
conda-build .
```

The package gets placed into

```
<user>\miniconda3\envs\build\conda-bld
```

The ‘build’ in the name is because we have it built with the environment ‘build’, so environments play nicely here.

To install the package from the current environment, I can use

```
conda install --use-local mypackage
```

Or to test it, create a new environment ‘test’ and install it from the ‘build’ environment space, for example:

```
conda install -c C:\Users\erich\miniconda3\envs\build\conda-bld mypackage
```

Note that I can put easily the whole environment with the packages on git if I want. Or I can just use the package file and install it:

```
conda install --no-deps .\mypackage-1.11.11-0.tar.bz2
```

## Packages for the NXP VS Code Build Tools

NXP provides an [installer for the MCUXpresso build tools](https://mcuoneclipse.com/2023/08/11/vs-code-mcuxpresso-installer/). How to create packages so I can use them with Conda, for example in a CI/CD environment?

Checking the MCUXpresso installer logs, one can see that it goes to [https://www.nxp.com/lgfiles/updates/mcuxpresso/components.json](https://www.nxp.com/lgfiles/updates/mcuxpresso/components.json).

From here I see where it gets for example the NXP RedLib additional libraries:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/package-download-location.jpg?w=530)

That way I can create my packages from that information:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/conda-package-definition.jpg?w=852)

That way I get everything to create my packages :-).

For the SHA256, one can use [https://emn178.github.io/online-tools/sha256\_checksum.html](https://emn178.github.io/online-tools/sha256_checksum.html)

## VS Code Conda Wingman

The [Conda Wingman VS Code extension](https://marketplace.visualstudio.com/items?itemName=DJSaunders1997.conda-wingman) is optional, but comes in handy with Conda:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/conda-wingman.jpg?w=952)

With the Conda Wingman I can activate an environment in VS Code from a YAML file:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/activate-environment-from-yaml.jpg?w=794)

That way I can store the YAML file in the project and switch to that environment from VS Code. Just nice extension.

## Summary

For an embedded development with different vendors and tools I need a way to switch environments. Conda enables me to use virtual environments, without the complexity of other package manager. While Conda has its origin in the Python world, it is a good solution for embedded C/C++ environments, including VS Code. It makes switching environments easy, I can build my own packages and can use it in a CI/CD environment, for example with docker. But that’s something for yet another article.

Happy packaging 🙂

### Links

- Github: [https://github.com/ErichStyger/mcuoneclipse/tree/master/conda](https://github.com/ErichStyger/mcuoneclipse/tree/master/conda)
- NXP extension for Visual Studio Code: [https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-for-visual-studio-code:MCUXPRESSO-VSC](https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-for-visual-studio-code:MCUXPRESSO-VSC)
- Conda Wingman: [https://marketplace.visualstudio.com/items?itemName=DJSaunders1997.conda-wingman](https://marketplace.visualstudio.com/items?itemName=DJSaunders1997.conda-wingman)
- Conda developer environment: [https://interrupt.memfault.com/blog/conda-developer-environments](https://interrupt.memfault.com/blog/conda-developer-environments)