#CMAKE #MCU #NXP
A *Triumvirate* is or *Triarchy* is built by three individuals which lead or rule something. In this article I want to rule a project with **Eclipse CDT**, **Visual Studio Cod**e and with building it from the **command line** for automated builds.

So what if I have an Eclipse project (say MCUXpresso IDE and SDK), and want to build it on a build server, and and I want to use the same time the project with Eclipse IDE and Visual Studio code?

Key to this is CMake: I’m keeping the Eclipse CDT features, adding CMake with Make and Ninja to the fix, and have it ‘ruled’ by three different ’emperor’: Eclipse, Visual Studio Code and from a shell console:

![](https://mcuoneclipse.com/wp-content/uploads/2023/04/cmake-sdk-project-for-eclipse-and-visual-studio-code.jpg?w=1024)

MCUXpresso SDK CDT project with CMake for Eclipse, Visual Studio Code and Command Line Building

## Outline

Eclipse CDT managed build projects greatly simplify project handling from a user perspective: create a project, add or remove source files and hit ‘build’, and everything is taken care of. But this depends on the build plugins used. It makes porting harder to port from one target platform to another, because different vendors have different plugins, with incompatible settings. That’s fine in some environments, but as soon more complex things like remote build servers, independent command-line building or just more control over the build process is needed, then the CDT managed build process is not really scalable. CMake is the solution to that problem.

In an earlier article “[Tutorial: Creating Bare-bare Embedded Projects with CMake, with Eclipse included](https://mcuoneclipse.com/2022/09/04/tutorial-creating-bare-bare-embedded-projects-with-cmake-with-eclipse-included/)” I showed an approach how to create your own CMake project from scratch. In this article I show how an existing MCUXpresso project can be extended with CMake, so I still can use the Eclipse way of building, but the same time I can use CMake and to build build it with make or ninja. As ‘icing on the cake’, I get it working with [Visual Studio Code](https://mcuoneclipse.com/2021/05/01/visual-studio-code-for-c-c-with-arm-cortex-m-part-1/). I have freedom of choice: what IDE I want to use with my build system of choice.

But most important: because everything is command-line driven (git, cmake, ninja, …), it is easy to build a complete CI/CD pipeline, e.g. using git with with a docker environment.

The sources of the project discussed in this article can be found on [GitHub](https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/MCUXpresso/FRDM-K22F/FRDM-K22F_SDK_CMake).

## Approach

In order to have CMake working, I need CMake with Make and/or Ninja installed and present in the path. I’m using the following tools and versions:

- MCUXpresso IDE V11.7.x with its internal toolchain
- MCUXpresso SDK 2.13.0
- CMake 3.22.5
- Ninja 1.11.1
- GNU Make 4.3.90
- Visual Studio Code 1.77.3

To use CMake with an existing Eclipse project, CMakeLists.txt files need to be added to the project, a toolchain defined (arm-none-eabi-gcc.cmake) and the linker files provided (copied from the auto-linker files). Additionally I have added custom build targets plus some batch files to make setup easier.

![](https://mcuoneclipse.com/wp-content/uploads/2023/04/cmake-sdk-project.jpg?w=472)

CMake added files

## SDK CMakeLists.txt

Place a `CMakeLists.txt` into each of the SDK folders. They include all source files of the folder and create a library.

![](https://mcuoneclipse.com/wp-content/uploads/2023/04/sdk-drivers-cmake-file.jpg?w=323)

SDK Drivers CMake File

Below is the example for the ‘`drivers`‘ SDK folder:

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24 | `file(GLOB FILES`  `*.c`  `)`  `# add_library: With this declaration, you express the intent to build a library. `  `# The first argument, here its pico-shift-register, is the name of the library, `  `# the second argument are the files that will be compiled to create your library.`  `add_library(drivers ${FILES})`  `# target_link_libraries: If you link with other libraries, list them here`  `target_link_libraries(`  `drivers `  `)`  `# target_include_directories: Libraries need to publish their header files `  `# so that you can import them in source code. This statement expresses where to find the files `  `# - typically in an include directory of your projects.`  `target_include_directories(`  `drivers `  `PUBLIC`  `./`  `../device`  `../CMSIS`  `)` |
| --- | --- |

The ones for ‘board’, ‘component’, ‘device’ and ‘utilities’ are very similar.

## Toolchain File

To define the toolchain, add the `arm-none-eabi-gcc.cmake` file:

![](https://mcuoneclipse.com/wp-content/uploads/2023/04/cmake-toolchain-file.jpg?w=330)

Toolchain File

Below it uses the toolchain of the IDE 11.7.0:

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24 | `set(CMAKE_SYSTEM_NAME Generic)`  `set(CMAKE_SYSTEM_PROCESSOR ARM)`  `# point ARM_TOOLCHAIN_BIN_DIR to things like`  `# "C:/Program Files (x86)/GNU Arm Embedded Toolchain/10 2020-q4-major/bin")`  `# "C:/NXP/MCUXpressoIDE_11.7.0_9198/ide/tools/bin")`  `set(ARM_TOOLCHAIN_DIR ${ARM_TOOLCHAIN_BIN_DIR})`  `set(BINUTILS_PATH ${ARM_TOOLCHAIN_DIR}) `  `set(TOOLCHAIN_PREFIX ${ARM_TOOLCHAIN_DIR}/arm-none-eabi-)`  `set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)`  `set(CMAKE_C_COMPILER "${TOOLCHAIN_PREFIX}gcc")`  `set(CMAKE_ASM_COMPILER ${CMAKE_C_COMPILER})`  `set(CMAKE_CXX_COMPILER "${TOOLCHAIN_PREFIX}g++")`  `set(CMAKE_OBJCOPY ${TOOLCHAIN_PREFIX}objcopy CACHE INTERNAL "objcopy tool")`  `set(CMAKE_SIZE_UTIL ${TOOLCHAIN_PREFIX}size CACHE INTERNAL "size tool")`  `set(CMAKE_FIND_ROOT_PATH ${BINUTILS_PATH})`  `set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)`  `set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)`  `set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)` |
| --- | --- |

## Linker Files

Create a folder named ‘`ld`‘ for the linker files and copy the linker files from the ‘`debug`‘ (generated by the linker) to the new folder:

![](https://mcuoneclipse.com/wp-content/uploads/2023/04/linker-files.jpg?w=399)

In case your SDK is not using generated linker files, you can simply use the one provided.

## Main CMakeLists.txt

Finally, add the main `CMakeLists.txt` to the project. You can use the one from [GitHub](https://github.com/ErichStyger/mcuoneclipse/blob/master/Examples/MCUXpresso/FRDM-K22F/FRDM-K22F_SDK_CMake/CMakeLists.txt). In that file, we reference and link the SDK libraries we have created earlier:

| 1  2  3  4  5  6  7  8  9  10  11  12 | `add_subdirectory(./board     build/board)`  `add_subdirectory(./drivers   build/drivers)`  `add_subdirectory(./utilities build/utilities)`  `add_subdirectory(./component build/component)`  `target_link_libraries(`  `${EXECUTABLE}`  `board`  `drivers`  `utilities`  `component`  `)` |
| --- | --- |

If your project has a different set of folders, simply extend/change the above structure.

## Generating Build Files

Using CMake means that CMake has first to run to create the Make or Ninja files. In other words: CMake is a ‘build generator’.

One challenge with CMake is that by default it ‘clutters’ the source folders with the generated files. What I prefer is to have everything generated in a ‘build’ folder, so I can easily get rid of it (clean). The ‘magic’ option for making this possible is the **\-B** option:

```
cmake -G"Unix Makefiles" . -B build
```

or

```
cmake -G"Ninja" . -B build
```
![](https://mcuoneclipse.com/wp-content/uploads/2023/04/creating-ninja-files.jpg?w=1024)

Creating Ninja Files with CMake

I recommend using Ninja, as it will build around 10x times faster!

I find the `-B` option very useful, as that way I can execute CMake from the project root directory, otherwise I would have to do it the following way:

```
mkdir build
cd build
cmake -G"Ninja" ..
```

This is of course doable, but why not making everything in a single step?

The next ‘magic’ option to be used with `make `or `ninja `is **\-C**: this makes a ‘change directory’ before executing the build. To start the build from a console in the project root directory, I can use

```
make -C build
```
```
ninja -C build
```
![](https://mcuoneclipse.com/wp-content/uploads/2023/04/build-with-ninja.jpg?w=886)

build with ninja

## Build Targets

To make building with CMake easier, I have created [Build Targets](https://mcuoneclipse.com/2017/07/22/tutorial-makefile-projects-with-eclipse/):

![](https://mcuoneclipse.com/wp-content/uploads/2023/04/build-targets-1.jpg?w=182)

Eclipse Build Targets

For Make it looks like this:

![](https://mcuoneclipse.com/wp-content/uploads/2023/04/make-build-target.jpg?w=330)

Make Build Target

You might notice the `../build`: this is needed because Eclipse uses the ‘build output’ folder as ‘current directory’, so I need to step up one directory (`..`) and then down into the `build `folder.

Very similar the same thing for Ninja:

![](https://mcuoneclipse.com/wp-content/uploads/2023/04/ninja-build-target.jpg?w=330)

Ninja Build Target

Using the Build targets I can easily start a build from the Eclipse GUI.

## Make or Ninja?

I grew up with Make, and I feel comfortable with it. But for larger projects I recommend ninja: it is optimized for speed, in the range of 5-10x faster than make. For the project in this article, a full build with make takes 5 seconds, where a full build with ninja only takes 1 second.

You can measure the time for example with:

```
powershell ("Measure-Command {make -C build | Out-Default}").TotalSeconds
```

or:

```
powershell ("Measure-Command {ninja -C build | Out-Default}").TotalSeconds
```

For larger projects, the speed factor is in the 10x range.

With having the project based on CMake, it is now open to be used with Visual Studio Code too. All what we need is to add a toolkit, as described in [Visual Studio Code for C/C++ with ARM Cortex-M: Part 5 – ToolKit](https://mcuoneclipse.com/2021/05/11/visual-studio-code-for-c-c-with-arm-cortex-m-part-5/): with this we can use the same project with CMake, Eclipse and Visual Studio Code:

![](https://mcuoneclipse.com/wp-content/uploads/2023/04/visual-studio-code-toolkit.jpg?w=1024)

Added Visual Studio Code Toolkit

Finally, I have something I can use with Eclipse, Visual Studio Code and on the command line:

![](https://mcuoneclipse.com/wp-content/uploads/2023/04/cmake-sdk-project-for-eclipse-and-visual-studio-code.jpg?w=1024)

Eclipse, Visual Studio Code and Command Line Shell

## Summary

Adding CMake capabilities to an existing Eclipse CDT based project is not very difficult: it requires adding the needed `CMakeLists.txt` file to the project plus a tool-chain definition file. With CMake I can not only enjoy full control over the build process (both from the IDE and command line), while keeping all the IDE features and capabilities, like configuration tools or debugging features. I can do the build both from the IDE and on the command line too. And it allows me to use a ninja based build, which is amazingly fast. Plus I can use the same project with Visual Studio Code.

Happy ruling 🙂

### Links

- Project on GitHub: [https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/MCUXpresso/FRDM-K22F/FRDM-K22F\_SDK\_CMake](https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/MCUXpresso/FRDM-K22F/FRDM-K22F_SDK_CMake)
- Ninja: [https://ninja-build.org/](https://ninja-build.org/)
- [Building Eclipse and MCUXpresso IDE Projects from the Command Line](https://mcuoneclipse.com/2017/08/03/building-eclipse-and-mcuxpresso-ide-projects-from-the-command-line/)
- Visual Studio Code Series: [Visual Studio Code for C/C++ with ARM Cortex-M: Part 1 – Installation](https://mcuoneclipse.com/2021/05/01/visual-studio-code-for-c-c-with-arm-cortex-m-part-1/)