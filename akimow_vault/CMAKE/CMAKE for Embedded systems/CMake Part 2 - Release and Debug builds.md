#CMAKE #MCU

- [Introduction](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#Introduction "Introduction")
- [Configuring Debug and Release Builds](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#Configuring_Debug_and_Release_Builds "Configuring Debug and Release Builds")
- [CMake Generator Expressions](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#CMake_Generator_Expressions "CMake Generator Expressions")
- [Toolchain Configuration](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#Toolchain_Configuration "Toolchain Configuration")
- [Build Customisation](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#Build_Customisation "Build Customisation")
- [Post Build Tools](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#Post_Build_Tools "Post Build Tools")
- [Conditional Tests](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#Conditional_Tests "Conditional Tests")
- [Custom Commands](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#Custom_Commands "Custom Commands")
- [Running Post Build Custom Commands](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#Running_Post_Build_Custom_Commands "Running Post Build Custom Commands")
- [Summary](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#Summary "Summary")
- [Postscript – A Simple Build Script](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#Postscript_%E2%80%93_A_Simple_Build_Script "Postscript – A Simple Build Script")
- [Linux Build Script (Bash)](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#Linux_Build_Script_Bash "Linux Build Script (Bash)")
- [Windows Build Script](https://blog.feabhas.com/2021/07/cmake-part-2-release-and-debug-builds/#Windows_Build_Script "Windows Build Script")

## Introduction

In my previous blog post [CMake Part – The Dark Arts](https://feabhasblog.wpengine.com/2021/07/cmake-part-1-the-dark-arts/) I discussed how to configure [CMake](https://cmake.org/) to [cross-compile](https://en.wikipedia.org/wiki/Cross_compiler) to target hardware such as our [STM32F407 Discovery board](https://www.st.com/en/microcontrollers-microprocessors/stm32f407vg.html).

We looked at the minimum requirements to configure the CMake build generator for a cross-compilation project using a project definition file (**CMakeLists.txt**), a toolchain definition file (**toolchain-STM32F407.cmake**). The CMake commands used to generate and build the project are:

```
cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake
cmake --build build
```

In the real world, projects are never as simple as this minimal example, and we try to reflect this in our training. To support the different phases and objectives of a [Software Development Lifecycle](https://en.wikipedia.org/wiki/Systems_development_life_cycle) a project will need to differentiate between developing code, testing (in its various forms) and releasing a version for end-use. We usually do this using build configurations.

Outputs from each type of build configuration are usually different. For example, a developer’s build typically includes metadata used by a [debugger](https://en.wikipedia.org/wiki/Debugger) which is not required for a released version of the project. Therefore, we need to configure our build process to cater for these different output requirements.

Both [Visual Studio](https://docs.microsoft.com/en-us/visualstudio/ide/understanding-build-configurations?view=vs-2019) and [Xcode](https://developer.apple.com/documentation/swift_packages/buildconfiguration)  support multiple build configurations, and CMake can generate appropriate build configuration files for these systems.

On the other hand, the [Unix/Linux/GNU Make](https://www.gnu.org/software/make/) system does not support build configurations. When using CMake to generate different build requirements using make files we take this into account by placing different build configurations in different output directories for each type of build we want to support.

## Configuring Debug and Release Builds

CMake refers to different build configurations as a [Build Type](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html).  Suggested build types are values such as **Debug** and **Release**, but CMake allows any type that is supported by the build tool. The build type specification is case insensitive, so we prefer to be consistent and use all upper case types despite the fact that the CMake documentation refers to capitalised types.

Our underlying build system for training is Make, so we need to create separate output folders for each type of build we require. Unfortunately, this means we have to run two very similar **cmake** commands to generate different configurations:

```
cmake -S . -B build/debug -DCMAKE_BUILD_TYPE=DEBUG \
      -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake

cmake -S . -B build/release -DCMAKE_BUILD_TYPE=RELEASE \
      -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake
```

We also have two separate commands, one for each build type:

```
cmake --build build/debug
cmake --build build/release
```

**Aside**: as a traditional Unix/Linux developer used to typing **make** I find these long and complex commands irksome and I know I’m not alone in this as it is a common source of criticism of CMake.

At this point, using a shell script, or scripts, to encapsulate the underlying **cmake** commands to simplify build the system would be advisable. There is an example shell script **build.sh** in the accompanying GitHub project [https://github.com/feabhas/cmake-blog-2](https://github.com/feabhas/cmake-blog-2).

For developer’s working with build tools supporting multiple build configurations (like Xcode and Visual Studio), the build type is not passed on the generate command line (using **\-CCMAKE\_BUILD\_TYPE=…**) but on the **cmake** build command with the **–config** option. For example:

```
cmake -S . -B build
cmake --build build --config Debug
```

**Note:** when using **Make** builds, the **–config** option is silently ignored, and when using multi-configuration build tools like Visual Studio, the setting for **CMAKE\_BUILD\_TYPE** is also silently ignored. A source of confusion and criticism when first starting to use CMake.

To support multiple build configurations for our training projects we just need to refactor the project and toolchain configuration files to be aware of build types. To do this, we make use of CMake generator expressions, so we need a short digression to discuss this feature of CMake.

## CMake Generator Expressions

A [generator expression](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html) is used to query aspects of the build as the build files are generated giving us a  *dynamic* view of the build generation process.

A *static* view of the build generation process is provided by command line definitions and variables defined in the configuration files, which are saved to the build cache file **CMakeCache.txt** in the build target directory. Note that variables should not change value once the build file generation process begins as this can cause discrepancies in the generated files.

[![](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2021/07/generator-expressions.png?resize=640%2C210&ssl=1)](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2021/07/generator-expressions.png?ssl=1)

Generator expressions are specified using  **$<** *expression* **\>** where the *expression* can take many different forms, whereas variable values are specified using **${** *name* **}**. Variables, once set, can be used at any point in the CMake files, whereas generator expressions query the current build generation environment and are only valid in specific contexts.

The use of the generator expression **$<TARGET\_FILE:Application>** resolves to the path to the output file in the build rule for our main application (**Application** is the target name). This expression is only valid after both the target and the target suffix have been defined:

```
add_executable(Application src/main.cpp)
set_target_properties(Application PROPERTIES
    SUFFIX .elf
)
```

For our project this is the absolute path to **build/debug/Application.elf.**

A generator expression is defined using a **$< : >** syntax with the entry after the colon defining the value of the expression. The first part before the colon takes different forms such as:

- a conditional test such as **$<CONFIG:DEBUG>** is true if this is a debug build type  defined by the command line option **\-DCMAKE\_BUILD\_TYPE=DEBUG**
- a target-dependent query such as **$<TARGET\_FILE:** *name* **\>**
- a string manipulation expression
- a variable query

The [generator expressions](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html) manual page describes the complete range of generator expressions.

## Toolchain Configuration

Refactoring our project toolchain file (**toolchain-STM32F407.cmake**) requires identifying compilation options only applicable to debug builds:

```
add_compile_definitions(
  STM32F407xx
  USE_FULL_ASSERT
  $<$<CONFIG:DEBUG>:OS_USE_TRACE_SEMIHOSTING_STDOUT>
  $<$<CONFIG:DEBUG>:OS_USE_SEMIHOSTING>
)
```

In this example **$<CONFIG:DEBUG**\> is true for a debug build type and similarly **$<CONFIG:RELEASE>** (not used in the example) is true for a release build. Note that the generator expression is all uppercase regardless of the actual value defined for CMAKE\_BUILD\_TYPE. The CMake documentation often refers to **DCMAKE\_BUILD\_TYPE=Debug** but the generator expression is always **$<CONFIG:DEBUG>**.

In our example we have added compiler definitions entries to support using host debugging via a serial port for a debug project.

For our training project we will need to use different runtime support configurations for the debug runtime (**rdimon.specs**) and a bare metal release (**nosys.specs**):

```
add_link_options(
  ${ARM_OPTIONS}
  $<$<CONFIG:DEBUG>:--specs=rdimon.specs>
  $<$<CONFIG:RELEASE>:--specs=nosys.specs>
  $<$<CONFIG:DEBUG>:-u_printf_float>
  $<$<CONFIG:DEBUG>:-u_scanf_float>
  -nostartfiles
  LINKER:--gc-sections
  LINKER:--build-id
)
```

As an alternative to using **$<CONFIG:RELEASE>** we could have tested for the absence of debug mode using the more complex syntax:

```
$<$<NOT:$<CONFIG:DEBUG>>:--specs=nosys.specs>
```

Here we use an inner generator expression to control the inclusion of an enclosing generator expression.

## Build Customisation

With the toolchain correctly configured we will update the project configuration (**CMakeLists.txt**) to refactor the compiler optimisations and symbol definitions for each build type:

```
add_compile_options(
  -Wall
  -Wextra
  -Wconversion
  -Wsign-conversion
  $<$<CONFIG:DEBUG>:-g3>
  $<$<CONFIG:DEBUG>:-Og>
  $<$<CONFIG:RELEASE>:-O3>
)

add_compile_definitions(
  $<$<CONFIG:DEBUG>:DEBUG>
)
```

**Note:** we need to define the compiler **DEBUG** symbol ourselves – it doesn’t happen automatically when we select the debug build type. The build type variable **CMAKE\_BUILD\_TYPE** is a CMake variable and not a linker or compiler defined symbol. The familiar syntax of using **\-D** on the command line to define CMake variables can be confusing when first using CMake as these are not definitions for the underlying compiler.

As an alternative approach for the build type definition we could have simply inserted the **CONFIG** generator expression as a compiler pre-processor definition:

```
add_compile_definitions(
  $<CONFIG>
)
```

This approach would add the pre-processor build type value as a compiler definition. However in this approach the value used would keep the original letter case so that using the CMake approach of **\-DCMAKE\_BUILD\_TYPE=Debug** would define a compiler variable called **Debug** which would not match the expected upper case definition (**DEBUG**).

Our example project does not need any linker options specific to the build type for our example project as these were handled in the toolchain file.

## Post Build Tools

Often when creating a target, such as our executable program, there are additional actions required after a successful build.

In our cross compiler project, we want to use the **objcopy** command to generate the hex file used by some [flash memory programmers](https://en.wikipedia.org/wiki/Programmer_\(hardware\)).

We use **add\_custom\_command**() function calls to run actions after a successful build of a target. CMake automatically generates a variable (**CMAKE\_OBJCOPY**) for the path of the **objcopy** program when the C or C++ compiler is specified in  the toolchain configuration file (in our case it will be **arm-none-eabi-objcopy**) . We should use this  variable preference to the *raw* command name:

```
add_custom_command(
  TARGET Application
  POST_BUILD
  COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:Application> 
          ${CMAKE_CURRENT_BINARY_DIR}/$<TARGET_NAME:Application>.hex
)
```

The use of **POST\_BUILD** command line should be self-explanatory: **CMAKE\_OBJCOPY** is set to the path of the the **objcopy** command (implicitly defined in **toolchain-STM32F407.cmake**) and **CMAKE\_CURRENT\_BINARY\_DIR** is the path to the build folder (**\-B** on the command line).

In building the **objcopy** command line we need to use generator expressions to get the path to the target application ELF file ($**<TARGET\_FILE:Application>**) and the base filename defined by **$<TARGET\_NAME:Application>** because these are specific to that target.

## Conditional Tests

One minor complication to using **CMAKE\_OBJCOPY** in the previous section is that the **objcopy** command may not be part of the toolchain we are using, in which case CMake sets **CMAKE\_OBJCOPY** to the value **CMAKE\_OBJCOPY-NOTFOUND**.

We should test a command path variable to make sure the command exists:

```
if (EXISTS ${CMAKE_OBJCOPY})
  add_custom_command(
  TARGET Application
  POST_BUILD
  COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:Application>
          ${CMAKE_CURRENT_BINARY_DIR}/$<TARGET_NAME:Application>.hex
)
else()
  message(STATUS "'objcopy' not found: cannot generate .hex file")
endif()
```

Note the use of parentheses on the **else()** and **endif()** functions – everything is a function in CMake. The **else()** part is optional, but we have used it to output a message during the build file generation phase, but this won’t be displayed in the actual build.

The first (optional) parameter to **message()** is a type indicator: in our case a **STATUS** message is output prefixed with **—**. In contrast, a **FATAL** message will display the message and stop the build generation at that point. Other message types are described in the [CMake manual](https://cmake.org/cmake/help/latest/command/message.html).

It is worth reinforcing the idea that CMake uses whitespace separated arguments to functions so the **COMMAND** arguments can be given across multiple lines without using a line continuation character (such as **\\**  in shell or Python scripts).

As an aside, you should be aware that CMake does not warn when an undefined variable is used, it simply substitutes nothing. This can be problematic, so we advise using the command line option **–warn-uninitialized**, which will display a warning message but won’t stop the build. So make sure you check the output from the build generation steps carefully in case you’ve mistyped a variable name.

```
cmake -S . -B build --warn-uninitialized -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake
```

There is one downside to adding this warning and that is when CMake generates the build files and the output directory already contains generated files CMake does not usethe toolchain file if the toolchain file has not been recently modified. In this situation the CMAKE\_TOOLCHAIN\_FILE is effectively unused and a warning is issued. To suppress this warning, which implies something is wrong when it isn’t, you can simply read the variable in a message:

```
MESSAGE(STATUS "Using toolchain file: ${CMAKE_TOOLCHAIN_FILE}")
```

## Custom Commands

While the CMake toolchain includes a few commonly used commands like **objcopy** and **ar** there are often additional project or environment specific commands you need to run post (or pre) build. While you can add these to the **CMakeList.txt** file, we think the toolchain file is the right place to configure the custom command paths.

In our cross compilation toolchain file (**toolchain-STM32F407.cmake**) we added logic to locate additional Arm commands not recognised by CMake:

```
find_program(CROSS_GCC_PATH "arm-none-eabi-gcc")
if (NOT CROSS_GCC_PATH)
  message(FATAL_ERROR "Cannot find ARM GCC compiler: arm-none-eabi-gcc")
endif()
get_filename_component(TOOLCHAIN ${CROSS_GCC_PATH} PATH)

set(CMAKE_C_COMPILER ${TOOLCHAIN}/arm-none-eabi-gcc)
set(CMAKE_Cxx_COMPILER ${TOOLCHAIN}/arm-none-eabi-g++)
set(TOOLCHAIN_AS ${TOOLCHAIN}/arm-none-eabi-as CACHE STRING "arm-none-eabi-as")
set(TOOLCHAIN_LD ${TOOLCHAIN}/arm-none-eabi-ld CACHE STRING "arm-none-eabi-ld")
set(TOOLCHAIN_SIZE ${TOOLCHAIN}/arm-none-eabi-size CACHE STRING "arm-none-eabi-size")
```

The **find\_program** function searches the host filesystem for the path to a given program which it stores in the variable name given as the first parameter. If the program isn’t found, the variable is set to *name-NOTFOUND*, in our case **CROSS\_GCC\_PATH-NOTFOUND**. We can check that the ARM compiler has been found by testing  **CROSS\_GCC\_PATH**:variable values ending with **\-NOTFOUND** evaluate to false.

Our search is complicated because we haven’t put the Arm toolchain in the standard Linux folders (such as **/usr/bin**), so we have to extract the directory path part of t**he arm-none-eabi-gcc** command so we can get the toolchain directory location with **get\_filename\_component**.

We have prefixed our custom variables defining the paths to the toolchain commands with **TOOLCHAIN-** to differentiate them from the standard CMake commands.

We need to store these variables where the main project can reference them, so we add them to the cache file using **CACHE STRING** followed by a variable description. Each CMake definition file is a separate processing environment, and variables not added to the cache will be discarded after build file processing is finished.

If you are interested, the variable cache is stored the file **CMakeCache.txt** in the build folder. An entry for the **arm-none-eabi-as** commnd looks like:

```
//arm-none-eabi-as
TOOLCHAIN_AS:STRING=/opt/gcc-arm-none-eabi-10-2020-q4-major/bin/arm-none-eabi-as
```

Note that we don’t use strings for the variable values but use what Perl calls [bare words](https://www.geeksforgeeks.org/barewords-in-perl/) which are values without the quotes (so long as we don’t have whitespace characters in the value). We have chosen to set the variable descriptions as strings because they usually contain spaces: in our case, as we have just used the program name as the description, these too could have been bare words.

## Running Post Build Custom Commands

In the project file (**CMakeLists.txt**) we don’t assume the custom toolchain commands exist because we may be supplying a different toolchain on the command line. As with **objcopy** we verify we can find the required post build commands:

```
if (EXISTS "${TOOLCHAIN_SIZE}")
  add_custom_command(
    TARGET Application
    POST_BUILD
    COMMAND ${TOOLCHAIN_SIZE} --format=berkeley $<TARGET_FILE:Application>
            >${CMAKE_CURRENT_BINARY_DIR}/$<TARGET_NAME:Application>.bsz
  )
  add_custom_command(
    TARGET Application
    POST_BUILD
    COMMAND ${TOOLCHAIN_SIZE} --format=sysv -x $<TARGET_FILE:Application>
            >${CMAKE_CURRENT_BINARY_DIR}/$<TARGET_NAME:Application>.ssz
    )

else()

    message(STATUS "'size' not found: cannot generate .[bs]sz files")

endif()
```

There is nothing in this code that we haven’t seen before.

## Summary

Real-world projects are always more complex than the simple examples used in most tutorials. In this post, we’ve looked at how CMake can be configured to generate two separate makefile build configurations using the same project and toolchain definition. This ability to add build configuration types to the GNU Make system is a good reason to use CMake in conjunction with the **make** command.

We recommend that you use the **–warn-uninitialized** when running CMake to generate the build files check the output from the build generation as this will help identify mistyped variable names.

A prototype project containing the code shown in this blog can be found in the GitHub project [https://github.com/feabhas/cmake-blog-2](https://github.com/feabhas/cmake-blog-2).

In the next blog [CMake Part 3 – Source File Organisation](https://feabhasblog.wpengine.com/2021/08/cmake-part-3-source-file-organisation/), we’ll look at multiple source and header files for a project and discuss how to organise a more extensive project into subsystems and libraries.

## Postscript – A Simple Build Script

The [GitHub project](https://github.com/feabhas/cmake-blog-2) supporting for this blog contains a minimal shell script (**build.sh**) for building debug and release projects under Linux.

### Linux Build Script (Bash)

```
#!/bin/bash
set -o errexit
set -o nounset
USAGE="Usage: (basename $0) [-v | --verbose] [ reset | clean | debug | release ]"

CMAKE=cmake
BUILD=./build
TYPE=DEBUG
BUILD_DIR=$BUILD/debug
CLEAN=
RESET=
VERBOSE=

for arg; do
  case "$arg" in
    --help|-h)    echo $USAGE; exit 0;;
    -v|--verbose) VERBOSE='VERBOSE=1' ;;
    debug)        TYPE=DEBUG; BUILD_DIR=$BUILD/debug ;;
    release)      TYPE=RELEASE; BUILD_DIR=$BUILD/release ;;
    clean)        CLEAN=1 ;;
    reset)        RESET=1 ;;
    *)            echo -e "unknown option $arg\n$USAGE" >&2; exit 1 ;;
  esac
done

[[ -n $RESET && -d $BUILD_DIR ]] && rm -rf $BUILD_DIR

$CMAKE -S . -B $BUILD_DIR --warn-uninitialized -DCMAKE_BUILD_TYPE=$TYPE -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake

[[ -n $CLEAN ]] && $CMAKE --build $BUILD_DIR --target clean

$CMAKE --build $BUILD_DIR -- $VERBOSE
```

### Windows Build Script

Developers working on Windows who install CMake will find that the default build generation targets the Microsoft [Build Tools for Visual Studio](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2019) compilers. Configuring CMake on Windows to cross compile using the Arm Embedded Toolchain is not straightforward and is the subject of a later blog post [CMake Part 4 – Windows Hosts](https://feabhasblog.wpengine.com/2021/09/cmake-part-4-windows-10-host/) including a suitable example PowerShell script.
