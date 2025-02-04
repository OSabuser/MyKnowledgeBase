#CMAKE #MCU

- [Introduction](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#Introduction "Introduction")
- [Managing Source Files](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#Managing_Source_Files "Managing Source Files")
- [Source File Wildcards](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#Source_File_Wildcards "Source File Wildcards")
- [Configuring File Dependencies](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#Configuring_File_Dependencies "Configuring File Dependencies")
- [Using Subsystems](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#Using_Subsystems "Using Subsystems")
- [Bare Metal Runtime Object Library](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#Bare_Metal_Runtime_Object_Library "Bare Metal Runtime Object Library")
- [RTOS Shared Library](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#RTOS_Shared_Library "RTOS Shared Library")
- [CMake Options](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#CMake_Options "CMake Options")
- [Dynamic Link Library](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#Dynamic_Link_Library "Dynamic Link Library")
- [Summary](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#Summary "Summary")
- [Postscript – A Simple Build Script](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#Postscript_%E2%80%93_A_Simple_Build_Script "Postscript – A Simple Build Script")
- [Linux Build Script (bash)](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#Linux_Build_Script_bash "Linux Build Script (bash)")
- [Windows Build Script](https://blog.feabhas.com/2021/08/cmake-part-3-source-file-organisation/#Windows_Build_Script "Windows Build Script")

## Introduction

In previous blog posts in this series ([Part 1](https://feabhasblog.wpengine.com/2021/07/cmake-part-1-the-dark-arts/) and [Part 2](https://feabhasblog.wpengine.com/2021/07/cmake-part-2-release-and-debug-builds/)), I looked at using [CMake](https://cmake.org/) to configure a build for a [cross compilation](https://en.wikipedia.org/wiki/Cross_compiler) to target hardware such as the [STM32F4 Series](https://www.st.com/en/microcontrollers-microprocessors/stm32f4-series.html). In this blog post I will look at how to configure project source code, identify subsystems and use CMake to manage the build for each subsystem.

In our training courses, we have identified two shared subsystems: the bare metal code used to initialise the C/C++ run time system and a middleware layer consisting of a real-time operating system (RTOS).

Before we look at configuring subsystems, we’ll briefly discuss managing a project with multiple source and header files.

## Managing Source Files

Any non-trivial project will use separate source files to encapsulate different functional areas of the system. So far, our example project has just used a single **main.cpp** source file, although the supporting GitHub projects use multiple source files to build a usable ELF image.

From the previous blog, you may remember that, for our build, we use a separate toolchain file (**toolchain-STM32F407.cmake**) and a project configuration file (**CMakeLists.txt**). The following is a simplified project configuration file where we have omitted the compiler and linker options as we are now concentrating on source code management:

```
cmake_minimum_required(VERSION 3.16)
project(target-cortexm LANGUAGES C CXX)

set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 17)

add_executable(Application
  src/main.cpp
)
set_target_properties(Application PROPERTIES
  SUFFIX .elf
)
```

You can find the complete configuration files in the [GitHub project accompanying this blog](https://github.com/feabhas/cmake-blog-3).

We can extend our list of source files for the target executable. Let’s say we have two separate modules for our project:

- hardware devices (**devices.cpp** and **devices.h**)
- logic controller (**controller.cpp** and **controller.h**).

We just add these source files to the **add\_executable**() definition:

```
add_executable(Application
  src/main.cpp
  src/gpio.cpp
  src/controller.cpp
)
```

Remember that CMake will scan the source files looking for dependencies to build a dependency tree for the source files and included header files. We don’t specify the header files as part of the source dependencies.

Although we never list the header files as part of the configuration (as discussed in previous blogs), we need to specify the directories to search for header files by adding entries to the **target\_include\_directories**() directive. For example:

```
target_include_directories(Application PRIVATE
  src
  include
)
```

Directory locations are relative to the project root but we can use the CMAKE\_SOURCE\_DIR variable to reinforce this:

```
add_executable(Application
  ${CMAKE_SOURCE_DIR}/src/main.cpp
  ${CMAKE_SOURCE_DIR}/src/gpio.cpp
  ${CMAKE_SOURCE_DIR}/src/controller.cpp
)

target_include_directories(Application PRIVATE
  ${CMAKE_SOURCE_DIR}/src
  ${CMAKE_SOURCE_DIR}/include
 )
```

When using subdirectories to organise source code the generated build commands print out each directory name as it is being processed. You can suppress this directory tracking using the **–no-print-directory** option on the build command line.

```
cmake --build build --no-print-directory
```

### Source File Wildcards

Teams following agile development models based on [Evolutionary Prototyping](https://en.wikipedia.org/wiki/Software_prototyping#Evolutionary_prototyping) where the source file structure can change regularly may prefer to use wildcard patterns to specify multiple source files to simplify project administration.

Teams following a more formal methodology usually prefer to specify every source file to avoid accidentally including unwanted sources and to maintain accurate list modules dependencies.

Software risk analysis usually requires a definitive list of the source files used to build a given project. As part of the build file generation CMake can optionally generate a list of compilation commands by setting the [CMAKE\_EXPORT\_COMPILE\_COMMANDS](https://cmake.org/cmake/help/latest/variable/CMAKE_EXPORT_COMPILE_COMMANDS.html) variable:

```
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

The well-documented proviso in CMake is that wildcards are evaluated when the build files are generated and not when the build takes place. To get proper wildcard support, the CMake build command must be run whenever there is a change to the source file structure.

Many sites using wildcards simply regenerate the build files whenever a build takes place – it doesn’t take that much time compared to the build. The downside is that old artefacts may be left in the output build folder and used in the build (e.g. linking in an object file that is no longer built from a source file). These sort of build problems may not be discovered until a later date when rebuilding the whole project from scratch.

Note that using the  **–target clean** option on the CMake command will only delete artefacts defined by the current build configuration and not older unreferenced artefacts. The simple solution to this problem is to delete the entire output folder and regenerate everything.

General advice is to avoid wildcards in a build configuration as the potential problems are more serious than the extra administrative load.

However, we decided our training projects benefited from using wildcards so that we didn’t have to get everyone to edit the build system as we went through the programming exercises. Our supporting build script always regenerates the build files.

### Configuring File Dependencies

While we’re looking at source files and wildcards it’s worth pointing out that CMake does not have built in rules for all possible source files for a project. Our embedded linker commands are dependent upon a number of hardware configuration files stored in an **ldscripts** subdirectory which we need to add as a dependency for the linker stage.

We do this using LINK\_DEPENDS option to the [set\_target\_properties()](https://cmake.org/cmake/help/v3.3/command/set_target_properties.html) function which is used to configure target specific properties. The related [set\_property()](https://cmake.org/cmake/help/latest/command/set_property.html) command is  used to set other CMake properties. There are a [plethora of properties](https://cmake.org/cmake/help/latest/manual/cmake-properties.7.html#manual:cmake-properties\(7\)) available in CMake and being aware of these, and when to use them is a good example of just how complex and confusing it is to define a CMake build configuration.

We can add our two linker configuration scripts as linker dependencies using the following:

```
set(LINKER_SCRIPTS 
  ${CMAKE_SOURCE_DIR}/ldscripts/mem.ld 
  ${CMAKE_SOURCE_DIR}/ldscripts/sections.ld
)

set_target_properties(Application PROPERTIES
  SUFFIX .elf
  LINK_DEPENDS "${LINKER_SCRIPTS}"
)
```

The LINK\_DEPENDS option requires a single parameter which is a semi-colon separated list of absolute pathnames to the files; relative filenames do not work so we need to use the CMAKE\_SOURCE\_DIR to prefix the relative paths to the files.

It isn’t well documented but when expanding a variable containing a list inside a quoted string the list values will be separated by semi-colons. Hence the need to create a list of linker configuration scripts and expand that list in a quoted string after the LINK\_DEPENDS keyword.

The LINK\_DEPENDS option is used to ensure the linker is run to rebuild the image if any linker configuration files change. There is a related CMAKE\_CONFIGURE\_DEPENDS option to **set\_property**() that can be used to force the build files to be regenerated if one or more files (not known to CMake) have changed.

The CMAKE\_CONFIGURE\_DEPENDS usage is similar to that of LINK\_DEPENDS requiring a semi-colon separated list but this time containing filenames relative to a given directory:

```
set(FILES config.yml) 
set__property(DIRECTORY ${CMAKE_SOURCE_DIR} 
  APPEND PROPERTY CMAKE_CONFIGURE_DEPENDS "${FILES}"
)
```

## Using Subsystems

Now we have looked at managing source files we can look at subsystems. In our embedded training projects we have identified two subsystems:

- the bare metal runtime
- an optional real-time operating system (RTOS)

Each subsystem has its own **CMakeLists.txt** configuration file and is defined in a subdirectory of our project as shown in the following screen image:

![](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2021/07/cmake-project.jpg?resize=291%2C287&ssl=1)**Embedded Project Structure**

In the main **CMakeLists.txt** project file we use  **add-subdirectory**() to add a subsystem to the main build.

The first subsystem to consider is the bare metal runtime and startup code which has an added complexity in it use of [weak linkage](https://feabhasblog.wpengine.com/2013/01/weak-linkage-in-c-programming/).

### Bare Metal Runtime Object Library

We created a subdirectory (**system**) for the [Arm CMSIS files](https://developer.arm.com/tools-and-software/embedded/cmsis), the STMicroelectronics files for the [STM32F407xx chipset](https://github.com/STMicroelectronics/cmsis_device_f4),  and files to support the [newlib C/C++ Standard Library](https://sourceware.org/newlib/).

Each subsystem is a separate project with its own project definition file (**system/CMakeLists.txt**). In a subsystem we list each dependent source file avoiding wildcards as we want to select exactly which files to use in our build:

```
cmake_minimum_required(VERSION 3.16)
project(target-system LANGUAGES C CXX)

add_library(system OBJECT
  src/newlib/_syscalls.c
  src/newlib/_startup.c
  src/newlib/_sbrk.c
  src/newlib/assert.c
  src/newlib/__dso_handle.c
  src/newlib/_exit.c
  src/newlib/_write.c
  src/cortexm/exception_handlers.c
  src/cortexm/_reset_hardware.c
  src/cortexm/_initialize_hardware.c
  src/diag/Trace.c
  src/diag/trace_impl.c
  src/cmsis/vectors_stm32f4xx.c
  src/cmsis/system_stm32f4xx.c
)
```

As with the main project file, we start with the minimum required CMake version and  a unique name for our subsystem project (**target-system**). All our current supporting files are written in C, but we still define this as a C and C++ project in case we decide to add C++ files at a later date. It is worth noting that the default project languages for CMake are C and C++ so this line is redundant, but we think it’s worth including anyway.

We define the library itself (called **system**) with the **add\_library**() function. The first argument is the library name and must be unique within the project (different from all other libraries and *executable* targets created by the project). CMake supports conditional configuration so two libraries with the same name can be defined, so long as only one is included in the generated build.

The first argument to **add\_library** defines the library type. There are several [CMake library types](https://cmake.org/cmake/help/latest/command/add_library.html) which include:

- **SHARED** – dynamically linked libraries (**.so** or .**dll** files) not supported by the [GNU Arm Embedded Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm)
- **STATIC** – statically linked libraries (**.a** or **.lib** files)
- **OBJECT** – not a single library file but a collection of separate object files (**.o** or **.obj** files)

The GNU Arm Linker supports weak linkage but if we use a static library with the **\-ffunction-sections** linker option then all weakly linked functions in the library will linked into the target image file in preference to any strongly linked versions in our code. To ensure the weak linkage mechanism works correctly we have to create an OBJECT library for our target.

The remaining **add\_library** arguments supply a list of source files which CMake will use to generate the build dependencies. In the previous example we have used paths relative to the project directory but the PROJECT\_SOURCE\_DIR variable can be used to make it clear these are relative to the directory containing the **CMakeLists.txt** file (… represents omitted entries):

```
add_library(system OBJECT
  ${PROJECT_SOURCE_DIR}/src/newlib/_syscalls.c
  ${PROJECT_SOURCE_DIR}/src/newlib/_startup.c
...
  ${PROJECT_SOURCE_DIR}/src/cmsis/vectors_stm32f4xx.c
  ${PROJECT_SOURCE_DIR}/src/cmsis/system_stm32f4xx.c
)
```

For an OBJECT library, the output object files are created in a build directory named after the library (**system.dir**). In our case, for a debug build, this is the location **build/debug/system/CMakeFiles/system.dir/** (this output directory store object files as a mirror of  the directory structure of the source files).

Apart from the object files, our subsystem includes several headers files which we need to add to the compiler’s include locations. There are two types of include locations:

- INTERFACE includes are part of the interface to the subsystem
- PRIVATE includes are only needed to compile the subsystem

We use **target\_include\_directories**() to identify the include file folders:

```
target_include_directories(system INTERFACE
  ${PROJECT_SOURCE_DIR}/include/cmsis
)

target_include_directories(system PRIVATE
  ${PROJECT_SOURCE_DIR}/include
  ${PROJECT_SOURCE_DIR}/include/cmsis
  ${PROJECT_SOURCE_DIR}/include/cortexm
  ${PROJECT_SOURCE_DIR}/include/diag
)
```

That’s all we need for our bare metal subsystem. The subsystem compilation will inherit the compiler options and defines from the main project configuration. If we had specific options or defines for the subsystem, we would specify these with [target\_compile\_options](https://cmake.org/cmake/help/latest/command/target_compile_options.html) and [target\_compile\_definitions](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions) . Do not set project-wide compiler (or linker) options and definitions from within a subsystem: you’ll only create trouble for yourself.

We now update the main project’s **CMakeLists.txt** file to add the subsystem configuration and add a dependency to the build target (**Application)**:

```
add_subdirectory(system)
target_link_libraries(Application PRIVATE system)
```

The **add\_subdirectory** is used to add the subsystem configuration to the project build. We have made the subsystem project name the same as the directory name, but this isn’t necessary.

A subsystem project could create multiple libraries (or even target executables), so we need to tell CMake to generate appropriate linker options to add the required library to our **Application** target with **target\_link\_library**.

The **target\_link\_library** needs to know if the library is only required to build our target executable or is part of this project’s interface. In our case, which is the most common, the library is only required to build the target, so we specify the PRIVATE link library argument. The other options are PUBLIC (the library file or files are made available to enclosing projects) and INTERFACE (the include locations are made available to enclosing projects). This approach allows CMake to support complex subsystem hierarchies.

In the top-level project configuration file (**CMakeLists.txt**), the use of PUBLIC/PRIVATE/INTERFACE is normally moot as  other project will not depend on this one. Many CMake examples specify PUBLIC libraries in the top-level configuration file, which can, and has, lead to confusion as to which is the correct approach. If in doubt, make libraries PRIVATE as this is usually the right approach; you’ll soon find out when this is wrong when a compilation or link fails.

One final twist to our training project configuration is that we originally intended to use a common CMake build file for training courses for embedded targets (cross compilation) and hosted courses where we do not add the **\-DCMAKE\_TOOLCHAIN\_FILE=toolchain-STM32F407.cmake** option to the **cmake** command line.

We did this by testing for the presence of the subsystem directory (which is not included with our hosted training courses):

```
if (IS_DIRECTORY ${CMAKE_SOURCE_DIR}/system)
  add_subdirectory(system)
  target_link_libraries(Application PRIVATE system)
endif()
```

There are advantages and disadvantages to this approach. Adopting this approach, if the subsystem directory is not present the build will be generated but the link may fail due to the missing directory.  Without the test for the presence of the directory the code generation stage will fail.

### RTOS Shared Library

Our second training project subsystem is the RTOS shared library. Again, we create a separate directory (**middleware**) for this subsystem with its own **CMakeLists.txt** file defining a separate project (**target-middleware**). We use FreeRTOS for our middleware RTOS and add the required sources files to a STATIC library called **middleware** (a full list of files is included in the accompanying [GitHub project](https://github.com/feabhas/cmake-blog-3)):

```
cmake_minimum_required(VERSION 3.16)
project(target-middleware LANGUAGES C CXX)

add_library(middleware STATIC
  FreeRTOSv202012.00/FreeRTOS/Source/croutine.c
  FreeRTOSv202012.00/FreeRTOS/Source/event_groups.c
  FreeRTOSv202012.00/FreeRTOS/Source/list.c
  FreeRTOSv202012.00/FreeRTOS/Source/queue.c
  FreeRTOSv202012.00/FreeRTOS/Source/stream_buffer.c
  FreeRTOSv202012.00/FreeRTOS/Source/tasks.c
  FreeRTOSv202012.00/FreeRTOS/Source/timers.c
 
  FreeRTOSv202012.00/FreeRTOS/Source/portable/GCC/ARM_CM3/port.c
  FreeRTOSv202012.00/FreeRTOS/Source/portable/MemMang/heap_3.c
)
```

This will create a static library file in the build folder. On Linux with a debug build, this will be in the location **build/debug/middleware/libmiddleware.a**.

As with the object library for the bare metal system we need to specify the locations of the interface header files:

```
target_include_directories(middleware INTERFACE
  cortex_m4_config
  FreeRTOSv202012.00/FreeRTOS/Source/include
  FreeRTOSv202012.00/FreeRTOS/Source/portable/GCC/ARM_CM3
)
```

CMake does not assume interface headers files are required for the build so in our case we need to include the same header files for the build:

```
target_include_directories(middleware PRIVATE
  cortex_m4_config
  FreeRTOSv202012.00/FreeRTOS/Source/include
  FreeRTOSv202012.00/FreeRTOS/Source/portable/GCC/ARM_CM3
)
```

We could have avoided the duplication of header files by using a variable:

```
set (MIDDLEWARE_INC
  cortex_m4_config
  FreeRTOSv202012.00/FreeRTOS/Source/include
  FreeRTOSv202012.00/FreeRTOS/Source/portable/GCC/ARM_CM3
)

target_include_directories(middleware INTERFACE
  ${MIDDLEWARE_INC}
)

target_include_directories(middleware PRIVATE
  ${MIDDLEWARE_INC}
)
```

Note that this is a local variable only used in this subsystem project, so we do not add it to the CMake variable cache (which we did for some variables defined in the toolchain file).

Our middleware subsystem requires header files from the bare metal subsystem, so we add this dependency to our project file. The header files are only required to build the middleware library, not to make use of the library, so we make them PRIVATE to this subsystem project:

```
target_link_libraries(middleware PRIVATE system)
```

That completes the middleware subsystem configuration so back in our main project we add the **middleware** static library using the same approach as the **system** object library:

```
add_subdirectory(middleware)
target_link_libraries(Application PRIVATE middleware)
```

We must add this library dependency after the **Application** target, but it can be placed before or after the **system** subsystem. CMake will ensure the generated build files will take multiple library dependencies into account.

### CMake Options

Not all of our training course exercises use the RTOS features, so we decided to control inclusion of the middleware using a [CMake option](https://cmake.org/cmake/help/latest/command/option.html). We add the controlling variable with its default value as an **option** (not a variable **set** command):

```
option(USE_RTOS "Enable RTOS support" OFF)
```

We then use this option and the presence of the **middleware** directory to control adding the library dependency:

```
if (USE_RTOS AND IS_DIRECTORY ${CMAKE_SOURCE_DIR}/middleware)
  add_subdirectory(${CMAKE_SOURCE_DIR}/middleware)
  target_link_libraries(Application PRIVATE middleware)
endif()
```

We also use this option to define an RTOS macro for the compiler options so that we can use this to conditionally include code when we are linked with the RTOS middleware:

```
add_compile_definitions(
  $<$<CONFIG:DEBUG>:DEBUG>
  $<$<CONFIG:DEBUG>:TRACE_ENABLED>
  $<$<BOOL:${USE_RTOS}>:RTOS>
)
```

Generator expressions (unlike if statements) do not implicitly test the values of variables (options) so we have to test the variable value in a Boolean context.

Back on the command line we need to add a CMake define (**\-DUSE\_RTOS=ON**) to override the default value for this option when we want to include the RTOS middleware:

```
cmake -S . -B build/debug --warn-uninitialized \
    -DCMAKE_BUILD_TYPE=DEBUG \
    -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake \
    -DUSE_RTOS=ON
```

CMake options are used to configure the generated build files and are not used during the build process itself. If a **cmake** command line defines a different value for an option, the build files must be regenerated, which is another good reason for always generating the build files when using CMake to manage a project build.

It’s worth emphasizing the point made earlier that CMake command line defines are not passed through to the underlying build commands. The compiler or linker will not see a USE\_RTOS macro definition.

### Dynamic Link Library

Although the Arm Toolchain does not support dynamic link libraries, it’s worth mentioning that dynamic link libraries are created in the same manner as static libraries but using the SHARED option to add\_library.

The following is a minimal **CMakeLists.txt** for a subsystem creating a shared library called **library-c** from a single file **helper.c**:

```
cmake_minimum_required(VERSION 3.16)
project(host-c-library LANGUAGES C)

add_library(library-c SHARED
  ${PROJECT_SOURCE_DIR}/helper.c
)

target_include_directories(library-c INTERFACE
  ${PROJECT_SOURCE_DIR}/
)
```

Note the inclusion of the current folder (PROJECT\_SOURCE\_DIR) in the interface directories because the header files for the library are in the same subsystem directory as the source files. This target will create a shared library on Linux with a debug build: **build/debug/library-c/liblibrary-c.so**.

To add a shared library to a project use the same approach as object and static libraries:

```
add_subdirectory(${CMAKE_SOURCE_DIR}/library-c)
target_link_libraries(Application PRIVATE library-c)
```

## Summary

Most, if not all, projects for embedded targets will consist of readily identifiable functional areas which can be configured as subsystems: for example, low level hardware access, bare metal runtime system, and so on. CMake supports subsystems by treating these as separate projects used to create libraries.

Each subsystem requires it’s own **CMakelist.txt** and therefore has to be defined in a sub directory: CMake is hard-coded to look for **CMakelist.txt** when generating build files for a project. Typically with embedded projects each subsystem generates a static library (**.lib** or **.a** file) linked into the target image.

CMake options can be used to configure the build process using command line definitions (**\-D** *settings*) for the generation of the project build files. These CMake defines are not added to the compiler defines for the build process but can be used in generator expressions to add compiler defines or compiler/linker options.

In the next blog in this series [CMake Part 4 – Windows Hosts](https://feabhasblog.wpengine.com/2021/09/cmake-part-4-windows-10-host/), I’ll look at how we configured CMake to use the Arm Toolchain on a Windows 10 host system.

A later article on [CMake Presets](https://feabhasblog.wpengine.com/2023/08/cmake-presets/) describes how to use the [presets](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html) feature added at CMake 3.19 in 2020.

## Postscript – A Simple Build Script

The [GitHub project](https://github.com/feabhas/cmake-blog-3) supporting for this blog contains a minimal shell script (**build.sh**) for building debug and release projects under Linux.

### Linux Build Script (bash)

```
set -o errexit
set -o nounset
USAGE="Usage: $(basename $0) [-v | --verbose | --rtos] [ reset | clean | debug | release ]"

CMAKE=cmake
BUILD=./build
TYPE=DEBUG
BUILD_DIR=$BUILD/debug
CLEAN=
RESET=
VERBOSE=
RTOS=

for arg; do
  case "$arg" in
    --help|-h)    echo $USAGE; exit 0;;
    -v|--verbose) VERBOSE='VERBOSE=1' ;;
    --rtos)       RTOS='-DUSE_RTOS=ON' ;;
    debug)        TYPE=DEBUG; BUILD_DIR=$BUILD/debug ;;
    release)      TYPE=RELEASE; BUILD_DIR=$BUILD/release ;;
    clean)        CLEAN=1 ;;
    reset)        RESET=1 ;;
   *)             echo -e "unknown option $arg\n$USAGE" >&2; exit 1 ;;
  esac
done

[[ -n $RESET && -d $BUILD_DIR ]] && rm -rf $BUILD_DIR
$CMAKE -S . -B $BUILD_DIR --warn-uninitialized \
  -DCMAKE_BUILD_TYPE=$TYPE \
  -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake \
  $RTOS
[[ -n $CLEAN ]] && $CMAKE --build $BUILD_DIR --target clean
$CMAKE --build $BUILD_DIR --no-print-directory -- $VERBOSE
```

