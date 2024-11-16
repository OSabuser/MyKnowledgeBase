#CMAKE #MCU

- [Introduction](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Introduction "Introduction")
- [What is CMake](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#What_is_CMake "What is CMake")
- [A Minimal Host Project](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#A_Minimal_Host_Project "A Minimal Host Project")
- [Generate and Build](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Generate_and_Build "Generate and Build")
- [Should we want to use a different build system instead of the host default (GNU Make for Linux) we need to tell CMake which build generator to use using the -G command option.  For example, to generate Ninja build files we would use:](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Should_we_want_to_use_a_different_build_system_instead_of_the_host_default_GNU_Make_for_Linux_we_need_to_tell_CMake_which_build_generator_to_use_using_the_-G_command_option_For_example_to_generate_Ninja_build_files_we_would_use "Should we want to use a different build system instead of the host default (GNU Make for Linux) we need to tell CMake which build generator to use using the -G command option.  For example, to generate Ninja build files we would use:")
- [Source File Dependencies](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Source_File_Dependencies "Source File Dependencies")
- [Toolchains](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Toolchains "Toolchains")
- [Cross Compiling](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Cross_Compiling "Cross Compiling")
- [Toolchain Definition File](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Toolchain_Definition_File "Toolchain Definition File")
- [Toolchain Program Paths](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Toolchain_Program_Paths "Toolchain Program Paths")
- [CMake Functions and Variables](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#CMake_Functions_and_Variables "CMake Functions and Variables")
- [Toolchain Compiler and Linker Options](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Toolchain_Compiler_and_Linker_Options "Toolchain Compiler and Linker Options")
- [Cross Compiler Options](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Cross_Compiler_Options "Cross Compiler Options")
- [Cross Linker Options](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Cross_Linker_Options "Cross Linker Options")
- [Cross Compiler Search Paths](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Cross_Compiler_Search_Paths "Cross Compiler Search Paths")
- [Compilation Options](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Compilation_Options "Compilation Options")
- [Adding a Target](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Adding_a_Target "Adding a Target")
- [Tracing the Build Commands](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Tracing_the_Build_Commands "Tracing the Build Commands")
- [Clean Builds](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Clean_Builds "Clean Builds")
- [Summary](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#Summary "Summary")

## Introduction

In our previous post [Why We Need Build Systems](https://feabhasblog.wpengine.com/2021/06/why-we-need-build-systems/) we examined the need for Build Systems in modern software development. In this post we will examine how to use CMake to mange the build process for a cross compilation project.

CMake can be described as a *marmite* application: you either love it or hate it. Here at Feabhas, we find ourselves falling in the latter category, despite the fact the CMake is widely used within the embedded and deeply embedded development community.

But we also know that many of the C/C++ static analysis and code quality tools integrate well with the CMake build system. For this reason, we’ve put aside our prejudices and reconsidered the way we build our example projects used during training by replacing [scons](https://scons.org/) with [CMake.](https://cmake.org/)

This blog post is a mix of musings and advice when using CMake for [cross-compiling](https://en.wikipedia.org/wiki/Cross_compiler)  to the [STM STM32F407 Discovery board](https://www.st.com/en/microcontrollers-microprocessors/stm32f407vg.html) that we use for our embedded C and C++ training. It is the first of a small series of posts looking at how we build our training projects comprising application code, supporting library code, real-time operating system and bare metal driver code.

The code and examples used in this blog are from CMake 3.16 on Ubuntu 20.04 LTS using the [GNU Arm Embedded Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm) and can be download from the GitHub project [https://github.com/feabhas/cmake-blog-1](https://github.com/feabhas/cmake-blog-1).

## What is CMake

CMake is **not** a build system like [Unix Make](https://en.wikipedia.org/wiki/Make_\(software\)) but a **build system generator**. Its purpose is to take your description of a project and generate a set of configuration files to build that project.

As part of the generation of build configuration files CMake also analyses source code to create a dependency graph of components so that when building the project unnecessary recompilation steps can be omitted to reduce build times. For larger projects this can reduce build times down from tens of minutes or hours, to a few minutes, perhaps even less than one minute.

The following schematic overview shows the complexity of building a modern software system with multiple inputs and output artefacts which will help explain why we need to use a build system to manage the process.

[![](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2021/05/build-process-300x169.jpg?resize=640%2C361&ssl=1)](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2021/05/build-process.jpg?ssl=1)CMake supports several hosted build systems such as [GNU Make](https://www.gnu.org/software/make/),(Linux), [Visual Studio](https://visualstudio.microsoft.com/) (Microsoft Windows), [Xcode](https://developer.apple.com/xcode/) (OSX) and [Ninja](https://ninja-build.org/) (multiple platforms) as well as cross-compilation systems such as [Android Studio](https://developer.android.com/studio) and [IAR Workbench](https://www.iar.com/products/architectures/arm/iar-embedded-workbench-for-arm/).

This plethora of different build systems adds to the confusion about using CMake. At a fundamental level both Visual Studio and Xcode provide a GUI environment that supports multiple build configurations such as Debug and Release. Make, on the other hand, is command-line based and does not support different build configurations. CMake tries hard to hide these differences but doesn’t always succeed.

CMake was originally developed in 1999, but the release of version 3.0 in 2014 introduced a new style of defining a project which is generally referred to as [Modern CMake](https://duckduckgo.com/?q=modern+cmake). This has added to the confusion over using CMake because there are many resources on the web that refer to the *legacy* style of CMake.

While CMake has extensive [documentation](https://cmake.org/cmake/help/v3.20/), it is very much a guide to what (descriptions of function and variables) that lacks the how (examples) and the why. It was difficult for us to access information that helped us understand how CMake works: specifically an overall understanding of how to configure a cross-compilation project.

Having said all that, CMake does work and achieves its purpose for creating a cross-platform build system that will generate build files that optimise the compilation steps.

## A Minimal Host Project

To use CMake, you create a **CMakeLists.txt** file, usually located in the root folder of your project. This file defines the source configuration, compiler and linker options, plus anything else needed to build and, if required, install your project.

The first thing in the file is the minimum CMake version, followed by a name for the project.

```
cmake_minimum_required(VERSION 3.16)
project(simple-host)
```

By default, the project will support a C and C++ [toolchain](https://en.wikipedia.org/wiki/Toolchain), but we could declare this explicitly with:

```
project(simple-host LANGUAGES C CXX)
```

Each CMake configuration requires one or more targets: either an executable program or a library; plus, the source files used to create that target. We’re going to use a single source file, **src/main.cpp**, to create a host-based executable **Application**:

```
add_executable(Application src/main.cpp)
```

That’s it for a minimal host build. CMake will use the default host toolchain to figure out how to generate the required build files. For our Ubuntu Linux build, it will be GNU Make files using **g++**, on Windows it would generate a Visual Studio workspace configuration, and Xcode for OSX.

### Generate and Build

Using CMake is a two-step process:

1. Generate the build files
2. Run the build system

Step one only needs to be run when creating a project, modifying compiler and/or linker options, adding (removing or renaming) source and header files, or making other configuration changes such as inter-file dependencies defined by **#include** statements.

Step two is run every time the project needs building (recompiling and linking).

We can shown this schematically for our project that generates GNU Make files.

[![](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2021/05/cmake-build-300x169.png?resize=640%2C361&ssl=1)](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2021/05/cmake-build.png?ssl=1)Our minimal host **CMakeLists.txt** file looks like:

```
cmake_minimum_required(VERSION 3.16)
project(simple-host LANGUAGES C CXX)
add_executable(Application src/main.cpp)
```

If we were to run **cmake** with no command-line arguments, it will generate the build files in the project root known as an *in-source* build. This build will intermix the object files, dependency files and executables in with the configuration and source files.

The in-source build approach is not a good idea as it is hard to differentiate source files (requiring source code management) from generated files (which should not be added to a source repository).

The best practice is to generate an *out-of-source* build, which we do by specifying the project source root (**\-S** option) and target build location (**\-B** option) on the command line:

```
cmake -S . -B build/
```

With modern CMake also run the build process via **cmake –build** (this was introduced with version 3.12 in 2018):

```
cmake --build build/
```

The older CMake approach was to change to the build folder to explicitly run the build tool (**make**) from that folder:

```
mkdir build
cd build
cmake ..
make
```

Either way, we now have an executable called **Application** (in the build **folder**) that we can run on the host using:

```
build/Application
```

### Should we want to use a different build system instead of the host default (GNU Make for Linux) we need to tell CMake which build generator to use using the -G command option.  For example, to generate Ninja build files we would use:

```
cmake -S . -B build/ -G Ninja
```

### Source File Dependencies

CMake does more than just generate the build files used to create object files and executable programs. It will generate a dependency file for each source file in the project. For example a **main.cpp** file will have a generated **main.cpp.d** file saved in the **build** folder hierarchy honouring the directory structure of the source files (in our case the file path is **build/CMakeFiles/Application.dir/src/main.cpp.d**).

For C/C++ source files  CMake will scan each file for **#include** statements and add these to the list of dependencies for that file. The generated configuration files for the build system (**make** in our case) will include those dependencies in its build rules. This will allow the build system to optimise the compilation steps avoiding recompiling source files that are unaffected by changes to other files.

The following diagram shows an example system with dependencies to illustrate how CMake can generate optimised build steps.

[![](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2021/05/cmake-dependency-300x169.png?resize=640%2C360&ssl=1)](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2021/05/cmake-dependency.png?ssl=1)In this example if we modify the **gpio.cpp** file this is the only file that is recompiled as there is no other file that depends on it. Obviously, we will always need to link the entire project to create the new executable image.

If, in our example, we now modify **gpio.h** then by implication **display.h** is also *out of date* as it depends on **gpio.h**. Now we have to recompile:

- **gpio.cpp** (depends on **gpio.h**)
- **display.cpp** (depends on **display.h** and **gpio.h**)
- **main.cpp** (depends on **display.h** and **gpio.h**)

This is an example the generated **main.cpp.d** file (full path names have been replaced by …):

```
CMakeFiles/Application.dir/src/main.cpp.obj: \
 .../src/main.cpp .../src/display.h .../src/gpio.h
.../stc/display.h:
.../src/gpio.h:
```

These dependency files could be used by other applications such as static analysis tools.

If we were to manually maintain our make system build files without using CMake we would have to specify all of these dependencies ourselves. This will be a tedious and error prone process for large projects due to the number of files and inter-dependencies involved. Failure to record the dependencies correctly can result in  unnecessary compilations taking place slowing the build down, or worse, modules not being recompiled when they should be leading to inconsistencies and potential bugs in the built project.

Furthermore, adding, deleting or modifying **#include** statements in any file requires us to update the build system dependency graph accordingly. Using CMake to manage the build files means we simply regenerate the build when required rather than having manually check and update the affected build configuration files ourselves.

Using CMake to generate the build files is a relatively quick operation compared to  compilation and linking, so many project administrators choose to always regenerate the build files at the start of a system build. That way any new dependencies (changed **#include** statements) will automatically be recorded in the generated dependency files.

This automated management of the build dependencies is a very powerful argument for using CMake, especially on larger projects with multiple source and headers files where dependencies can quickly become very labyrinthine. Even small projects like our training projects with around 40 sources files benefit from using CMake to manage the build process.

### Toolchains

Perhaps the most significant source of confusion we see in articles and questions on web sites is how CMake uses a toolchain when generating the build files.

A toolchain must be defined before CMake starts processing the **CMakeLists.txt** file. Unless you provide a command-line argument to tell CMake which toolchain to use it use the default toolchain for the current host. Any attempt to modify or override the toolchain from within **CMakeLists.txt** typically won’t work or is just plain wrong.

Once the toolchain is defined, CMake will then validate the compiler and linker by building and discarding a simple test application. A toolchain configuration must define all the compiler and linker options necessary to perform a successful test build.

## Cross Compiling

If the default host toolchain is not suitable, as is the case for cross compiling, then the recommended way of specifying the toolchain details is in a separate toolchain file. In fact this is the only reliable way of overriding the default toolchain due to the lifecycle of the CMake processing steps.

To generate a cross compilation build using CMake, we specify the location and command names of the compiler, linker and other build tools (the toolchain). We also have to define compiler and linker options that will ensure the test build works.

To do this, we add a command-line option to **cmake** to tell it to read toolchain information from a file using the CMAKE\_TOOLCHAIN\_FILE variable:

```
cmake -S . -B build/ -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake
```

There are no standard naming conventions for toolchain files, but we’ve followed other examples and included the target specification in the file **toolchain-STM32F407.cmake.**

### Toolchain Definition File

In the **toolchain-STM32F407.cmake** file we define variables for the target system name and  version:

```
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_VERSION Cortex-M4-STM32F407)
```

CMake has a standard set of known system names (Linux, Windows, OSX, Android and others) but we are using **Generic** as there is no predefined name for a bare-metal embedded system.

Setting the system name tells CMake that this is a cross compilation project, and it will define the CROSS\_COMPILING variable as true.

The system version can be anything we want, and we decided to use it to identify the actual target rather than a version number.

### Toolchain Program Paths

The next step is to specify the toolchain programs. We have added the toolchain directory to the search path, so we just need to set the C and C++ compiler command names which are prefixed with **arm-none-eabi-** for the GNU Arm Embedded Toolchain:

```
set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER arm-none-eabi-g++)
```

CMake has rules for finding the location of the other toolchain commands. Typically, a cross compiler toolchain uses a common command prefix (**arm-none-eabi-** for GNU Arm) and CMake uses this convention to generate names for the other tools (**arm-none-eabi-gcc**, **arm-none-eabi-ar,** and so on) if we provide the name of the C compiler. This means we could have just defined the CMAKE\_C\_COMPILER as **arm-none-eabi-gcc** and CMake will have inferred the name of the C++ compiler as **arm-none-eabi-g++.**

CMake will use the full pathnames for the tools rather than the command name so that the actual build tool can be run without adding the build tools directory to the search path.

A downside of using full pathnames in the generated build files is that the build configuration must be regenerated if the build tool location changes. This happens when Arm release a new version of their GNU Toolchain as the version number is part of the path. You cannnot generate build files that use relative pathnames, even if you use relative pathnames in the toolchain definitions.

If you are interested, you can look at the generated configuration variables in the file **CMakeCache.txt** in the output build directory. If your build isn’t working as expected, this is one of the files to examine to look for a misconfiguration. To check for the C++ compiler path look for the line following the comment line containing **CXX  compiler**:

```
//CXX compiler
CMAKE_CXX_COMPILER:FILEPATH=/opt/gcc-arm-none-eabi-10-2020-q4-major/bin/arm-none-eabi-c++
```

All that’s left to do in the toolchain file is to provide sufficient compiler and linker options to ensure the test build will compile and link successfully.

At this point we should digress and explain the syntax for CMake functions, arguments, and strings.

## CMake Functions and Variables

The CMake configuration language is simply a series of function calls with function arguments (parameters) passed in parentheses (round brackets). Flow control constructs such as if statements and loops are also implemented as functions.Parameters are white space separated and long argument lists are usually split across multiple lines (one argument per line) to aid readability.

There is no need to surround arguments with double quotes unless a space or round bracket is needed in the argument.An argument in quotes defines a string and sometimes CMake can be confused by an empty argument and an empty string. It is best to avoid strings except when using the **if()** function to test string values.

Multiple arguments form a list and most functions accept arbitrary sized lists. Some functions use context-sensitive keywords (such as PRIVATE shown later in the **CMakeLists.txt** file) to supply function specific information or partition the list of arguments into different sections.

Variable substitution uses **${…}** (the curly brackets are mandatory) – there is no need to wrap variable substitution in a string (even when the variable value contains white space or round brackets).

## Toolchain Compiler and Linker Options

Resuming our example of a minimal cross compiler build definition we have to supply a some common compiler and linker options for the Arm target. We’ll put these into a custom CMake variable so we can reuse the values:

```
set(ARM_OPTIONS -mcpu=cortex-m4 -mfloat-abi=soft --specs=nano.specs)
```

### Cross Compiler Options

We add our common options along with other cross compiler options using the **add\_compile\_options** function:

```
add_compile_options(
  ${ARM_OPTIONS}
  -fmessage-length=0
  -funsigned-char
  -ffunction-sections
  -fdata-sections
  -MMD
  -MP
)
```

And some required pre-processor defines using **add\_compile\_definitions**:

```
add_compile_definitions(
  STM32F407xx
  USE_FULL_ASSERT
  OS_USE_TRACE_SEMIHOSTING_STDOUT
  OS_USE_SEMIHOSTING
)
```

We could have equally well have added the compiler SEMIHOSTING definitions in our main **CMakeLists.txt** file, but as they are standard for all cross compilations for the target we’ve put them in the toolchain configuration.

## Cross Linker Options

Linker options defined using **add\_link\_options** need to include a minimal bare metal C runtime library specification:

```
add_link_options(
  ${ARM_OPTIONS}
  --specs=rdimon.specs
  -u_printf_float
  -u_scanf_float
  -nostartfiles
  LINKER:--gc-sections
  LINKER:--build-id
)
```

CMake uses the LINKER: prefix to indicate a linker specific directive. On older **gcc** linkers this will generate a **Wl,** option, whereas on  other compilers (later **gcc**, **clang**, etc.), it will generate **\-Xlinker** options.

### Cross Compiler Search Paths

Finally, we need to tell CMake which locations to search when resolving the absolute paths for toolchain components:

```
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

This is a standard definition that basically says the toolchain commands (programs) are outside the project, but libraries, packages and include file locations are within the project folder hierarchy.

We now have a complete toolchain configuration file which, just to remind you, we must add to the **cmake** command line only when generating the build files (it isn’t required when we perform the actual build):

```
cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake
cmake --build build
```

## Compilation Options

In our cross compilation configuration in **CMakeLists.txt**, as with our hosted projects, we need to define the CMake version and project name:

```
cmake_minimum_required(VERSION 3.16)
project(target-cortexm LANGUAGES C CXX)
```

In most projects we will want to override the standard compiler and linker options to configure C/C++ standards compliance and warning levels (at the very least). So, before we define the cross compiler build target using **add\_executable**, we now set C and C++ options to use for all compilations:

```
set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 17)

set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
```

The last four lines ensure we use recommended compiler options **\-std=c++17** instead of the GNU specific versions **\-std=gnu17**; we also enforce ISO C/C++ compiler standards.

We add compiler options and definitions, in the same manner, we used in the toolchain file:

```
add_compile_options(
  -Wall
  -Wextra
  -Wconversion
  -Wsign-conversion
  -g3
  -Og
)

add_compile_definitions(
  DEBUG
)
```

The options and definitions are cumulative. If there are any conflicts, then the values defined in **CMakeLists.txt** take precedence.

### Adding a Target

As with the host project we need to add an executable target:

```
add_executable(Application src/main.cpp)
```

Again it is worth emphasising the **add\_executable** function must define the target before you set any target specific definitions. CMake is generating the build files and must be told what to build first, and then how to define the build steps.

After we have added the project executable, we can set compiler and linker options for the target. For a cross compilation we want the target executable to have a **.elf** suffix. This is achieved using target specific function calls that require the name of the target (**Application**) as the first argument.

```
set_target_properties(Application PROPERTIES
  SUFFIX .elf
)
```

We must define the target hardware configuration for the linker memory allocation, display memory usage after linking, and generate a map file:

```
target_link_options(Application PRIVATE
  -T${CMAKE_SOURCE_DIR}/ldscripts/mem.ld
  -T${CMAKE_SOURCE_DIR}/ldscripts/sections.ld
  LINKER:--print-memory-usage
  LINKER:-Map,${CMAKE_CURRENT_BINARY_DIR}/Application.map
)
```

As a minor digression we’ll point out the duplication of the word **Application** used for **Application.exe** and **Application.map**. We have done this for simplicity while we get the basic concepts sorted. In the next post we’ll look at using CMake generator functions to avoid this repetition.

Although our simple example currently  doesn’t include any user defined header files we normally need to tell CMake which include directories to add to the compiler command line:

```
target_include_directories(Application PRIVATE
  src
)
```

The PRIVATE keyword defines the scope of the include directories when using the target. This is more applicable to a library target (discussed in a later post) where we may want to define INTERFACE or PUBLIC includes to be used with the library. As this is an executable program there is no external dependency on the include files, so we mark these as private.

Note that at this point we haven’t included any driver files for our target board, just a single main application file. Additional source files could be added to the source dependencies on the **add\_executable** definition but this doesn’t capture the architecture of our application. To add support files for the target hardware, and possibly a Real Time OS we will use the **target\_link\_libraries** to define a subsystem in out application in a later blog post.

For now if you want to view the complete project you can do so in out public git repo [https://github.com/feabhas/cmake-blog-1](https://github.com/feabhas/cmake-blog-1)

For completeness, if we had any target-specific compiler configuration requirements that are not included in the toolchain file, we’d have used the **target\_compile\_definitions** and **target\_compile\_options** functions specifying our target name (**Application**) as the first argument.

As an aside, CMake automatically sets several variables that reflect the project build environment. We have used:

- **${CMAKE\_SOURCE\_DIR}** – the project root folder (**\-S** on the **cmake** command line)
- **${CMAKE\_CURRENT\_BINARY\_DIR}** – the output build directory (**\-B** on the **cmake** command line)

This configuration will create the **build/Application.elf** file ready for use by our target loader tools:

```
cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake
cmake --build build
```

## Tracing the Build Commands

CMake prints out information about the build files as they are generated and includes in those generated files print statements about what is being built, but how the compilation and linker commands themselves.

To diagnose problems with the generated commands you can add the **VERBOSE=1** option to the **cmake –build** command to passed into the build. This is not a CMake command option so must be added after **—** option to mark the end of the options:

```
cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake 
cmake --build build -- VERBOSE=1
```

## Clean Builds

Build systems typically optimise the build process by omitting steps that produce artefacts that are already up to date – in simple terms don’t recompile a file if the source and the dependency files have not changed since the last build.

When the build fails, or the generated artefacts are missing or incorrect, a first step is to force a rebuild of the entire system. You can do this by adding the **– -target clean** option to the build command line and then rerun the build step:

```
cmake --build build --target clean
cmake --build build
```

The **clean** target will remove the generated files forcing all build steps to be executed on the next build command (cleaning the build does not automatically initiate a new build).

When changing and updating the build configuration itself inconsistencies can arise in the build folder. Frequently obsolete files that are no longer required can be left around in the build sub-folders. A more dramatic clean build is to remove the entire build folder and regenerate the build files. On our Linux system we’d simply run an **rm** command:

```
rm -r build
cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake
cmake --build build
```

As these CMake build steps start to get more complex many sites will add a front end script to simplify running the different build steps so developers do not have to learn and enter the potentially long CMake build commands.

## Summary

Once you understand that the CMake toolchain must be configured on the command line, problems associated with using a cross compiler should be much easier to resolve.

Cross compiler toolchain configuration is complex enough to require a separate toolchain definition file specified with the -DCMAKE\_TOOLCHAIN\_FILE command-line option.

If you simply wanted to use a different compiler such as [clang](https://clang.llvm.org/) you could *possibly* get away with setting the compiler name or compiler path on the CMake command line:

```
cmake -S . -B build -DCMAKE_CXX_COMPILER=clang++
cmake --build build
```

But this approach will only define the C++ compiler command leaving the C compiler and standard toolchain programs with their default names. To use the full Clang toolchain (often called [binutils](https://en.wikipedia.org/wiki/GNU_Binutils)), you should use a toolchain definition file without defining the CMAKE\_SYSTEM\_NAME variable because this won’t be a cross compilation – the target architecture is still the host.

**NOTE:** if you read about toolchain configuration on some web pages, you may find references to  \_CMAKE\_TOOLCHAIN\_PREFIX or CMAKE\_TOOLCHAIN\_PREFIX variables. This is a common misconception as these variables do not exist in modern CMake and cannot be used to configure a toolchain by defining a common prefix before the command name.

In the next post [CMake Part 2 – Release and Debug builds](https://feabhasblog.wpengine.com/2021/07/cmake-part-2-release-and-debug-builds/), we’ll look at using CMake to configure different debug and release builds.

You can download the complete project from our GitHub repository [https://github.com/feabhas/cmake-blog-1](https://github.com/feabhas/cmake-blog-1).

- [About](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#abh_about)
- [Latest Posts](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/#abh_posts)

[![Martin Bond](https://i0.wp.com/blog.feabhas.com/wp-content/uploads/gravatar/martin-bond-2014-600x600-1.jpg?w=250&ssl=1)](https://blog.feabhas.com/author/martinb/ "Martin Bond")

An independent IT trainer Martin has over 40 years academic and commercial experience in open systems software engineering. He has worked with a range of technologies from real time process controllers, through compilers, to large scale parallel processing systems; and across multiple sectors including industrial systems, semi-conductor manufacturing, telecomms, banking, MoD, and government.

[![Martin Bond](https://i0.wp.com/blog.feabhas.com/wp-content/uploads/gravatar/martin-bond-2014-600x600-1.jpg?w=250&ssl=1)](https://blog.feabhas.com/author/martinb/ "Martin Bond")

Latest posts by Martin Bond ([see all](https://blog.feabhas.com/author/martinb/))

- [CMake Presets](https://blog.feabhas.com/2023/08/cmake-presets/) - August 1, 2023
- [C++20 Coroutine Iterators](https://blog.feabhas.com/2021/09/c20-coroutine-iterators/) - September 23, 2021
- [C++20 Coroutines](https://blog.feabhas.com/2021/09/c20-coroutines/) - September 16, 2021

[![](https://i0.wp.com/blog.feabhas.com/wp-content/uploads/2021/06/Martin-Bond-2014-600x600-1.jpg?resize=150%2C150&ssl=1)](https://blog.feabhas.com/author/martinb/)

##### [Martin Bond](https://blog.feabhas.com/author/martinb/)

An independent IT trainer Martin has over 40 years academic and commercial experience in open systems software engineering. He has worked with a range of technologies from real time process controllers, through compilers, to large scale parallel processing systems; and across multiple sectors including industrial systems, semi-conductor manufacturing, telecomms, banking, MoD, and government.