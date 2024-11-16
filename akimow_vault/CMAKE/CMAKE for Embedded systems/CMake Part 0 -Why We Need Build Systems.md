#CMAKE #MCU
Build systems were developed to simplify and automate running the compiler and linker and are an essential part of modern software development. This blog post is a precursor to future posts discussing our experiences refactoring the training projects to use the CMake build generator.

- [Using Build Systems](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Using_Build_Systems "Using Build Systems")
- [Source Code Organisation – In Source Builds](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Source_Code_Organisation_%E2%80%93_In_Source_Builds "Source Code Organisation – In Source Builds")
- [Source Code Organisation – Out of Source Builds](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Source_Code_Organisation_%E2%80%93_Out_of_Source_Builds "Source Code Organisation – Out of Source Builds")
- [Source File Dependencies](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Source_File_Dependencies "Source File Dependencies")
- [Compilation Options](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Compilation_Options "Compilation Options")
- [Include File Locations](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Include_File_Locations "Include File Locations")
- [Code generation Options](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Code_generation_Options "Code generation Options")
- [Preprocessor Directives](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Preprocessor_Directives "Preprocessor Directives")
- [Build Configuration](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Build_Configuration "Build Configuration")
- [Program Linker Configuration](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Program_Linker_Configuration "Program Linker Configuration")
- [Post Build Processing](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Post_Build_Processing "Post Build Processing")
- [Managing Testing and Debugging](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Managing_Testing_and_Debugging "Managing Testing and Debugging")
- [Limitations of a  Build System](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Limitations_of_a_Build_System "Limitations of a  Build System")
- [Summary](https://blog.feabhas.com/2021/06/why-we-need-build-systems/#Summary "Summary")

## Using Build Systems

Build systems can be standalone command line applications such as  [Make](https://www.gnu.org/software/make/), [Scons](https://scons.org/) and [Ninja](https://ninja-build.org/); or part of an (Integrated Development Environment [IDE](https://en.wikipedia.org/wiki/IDE)) like [Visual Studio](https://visualstudio.microsoft.com/) , [XCode](https://developer.apple.com/xcode/) or [IAR Workbench](https://www.iar.com/products/architectures/arm/iar-embedded-workbench-for-arm/).

Configuring build systems for a project can be complex and there are a few applications around that will generate the required build files from a simpler project configuration file. The most popular of these tools are [CMake](https://cmake.org/), which generates files for several build systems, and [Meson](https://mesonbuild.com/), which generates Ninja build files.

A 2021 survey by the [Standard C++ Foundation](https://isocpp.org/blog/2021/04/results-summary-2021-annual-cpp-developer-survey-lite) showed that CMake was used by 4 out of 5 of the respondents, while Meson and  scons are each used by less than 1 in 20. The survey also showed that Make/nmake and MSBuild (Visual Studio) are used roughly equally by 2 out of 5 people and Ninja by 1 out of every 3. In many cases the respondents will be using more than one build system across multiple projects.

Interestingly, or worryingly, from our perspective of moving our training projects to CMake, about two thirds of the respondents found CMake to be a major or minor pain point. Managing CMake was the third most frustrating aspect of C++ development behind managing libraries and project build times.

We need to use build systems because compiling an application from source code is no longer as simple as running a single compilation command such as:

```
$ g++ -o Application main.cpp
```

While this works, it relies on default configuration options for the compiler and linker.

We should point out that referring to **g++** as a compiler is misleading. It isn’t very obvious, but **g++** is itself a very simplistic build system responsible for running a number of build phases:

- the preprocessor
- the compiler
- the assembler (code generator)
- the linker

Anyone who has worked with the Microsoft C++ tools will be aware that there is a separate compiler (**cl.exe** which includes the preprocessor and assembler) and linker (**link.exe**).

The build process is complex and involves many stages with different requirements, inputs and outputs and can be summarised in the following diagram.

[![](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2021/05/build-process-1024x576.jpg?resize=640%2C360&ssl=1)](https://i0.wp.com/feabhasblog.wpengine.com/wp-content/uploads/2021/05/build-process.jpg?ssl=1)In reality, we use a build system to manage some or all of the following aspects of software development:

- source code organisation
- source code inter-dependecies
- managing third party libraries
- compilation options
- code generation options
- program linker configuration
- post build processing
- managing testing

It’s worth looking closely at these steps in order to understand the requirements of a build system and the concept of a development [Toolchain.](https://en.wikipedia.org/wiki/Toolchain)

## Source Code Organisation – In Source Builds

Our simple example above generates the intermediary object files and executable program in the current directory. When we store the output files in the same directory as the source files we call this an *In-source build*: this is generally considered a bad idea. Managing the source code will become problematical when our application gets more complex and requires multiple files.

The output files from a build process can be re-created from the source code so do not need to be saved using a backup regime, whereas source code must be saved which these days is usually achieved using a source code repository such as [Git](https://en.wikipedia.org/wiki/Git).

The lifecycle management of source code and build output files are independent and should be stored in different locations. Many build tools and tool generators (like Meson) do not even support in-source builds.

## Source Code Organisation – Out of Source Builds

All projects should store generated artefacts (object files, executable applications, etc.) in a separate location to the source code.

A typical C/C++ application will separate out logical subsystems into different components and these will usually be stored in a hierarchical directories structure representing the application’s architecture. A build system should manage source code (including header files) stored in multiple directory locations.

There is no standard structure for C/C++ source code and a quick browse of open source projects in [GitHub](https://github.com/) shows nearly as many different source code structures as there are projects (a bit of an exaggeration but it does show there is no single standard).

Many projects intermingle the header files with the source files, which can lead to complex compilation options when the source code is stored in hierarchical directories. Header files define the interface to a module’s functionality, and a best practice approach is to separate interface from implementation. Applying this approach to source code organisation implies that header files should be stored separately from the implementation files.

Back when I was developing code using Unix I used directory names **src** for source files and **hdr** for the header files, with both directories stored in the project’s root directory; but the use of **src** and **inc** is the usual practice in embedded system. Not everyone likes the traditional Unix style of shortening words (often by omitting vowels) preferring to use **source** and **include** instead.

What everyone does agree on is that the generated output files go in a separate folder: normally in the project workspace. Typical names are **build** or **target** as used by the [Maven](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html) build system.

No matter how the source code is organised, the compiler must be told where the source and header files are located.

Another aspect of source code organisation is integrating the build process with a source code management system. The ability to download the latest committed version of source code before building means the same build system can be used for local development as well as the centralised pipelines of [Continuous Integration](https://en.wikipedia.org/wiki/Continuous_integration) commonly used in agile development methodologies.

Once the locations of source and headers files are determined, a definition of the component files is also required for a build process. These files can be supplied individually or by using wildcard patterns – a good build system will support both.

The benefit of listing each file individually is that the build system provides a definitive list of source dependencies for the build artefacts, which is helpful for general administration tasks and undertaking a risk analysis for business continuity purposes. The drawback is that this list has to be maintained and the extra administration required can nudge developers into including code in existing modules rather than creating a new module, especially if the build configuration is centrally administered in an overly restrictive manner.

A wildcard approach to filenames (e.g. **src/\*.cpp**) superficially seems more straightforward as it doesn’t require the developer to list each file allowing new files to be easily added. The downside is that the build system does not have a definitive list of the source code files for a given artefact, making it harder to track dependencies and understand precisely what components are required. Wildcards also allow spurious files to be included in the build – maybe an older module that has been superseded but not removed from the source folder.

Best practice says to list all source modules individually despite the, hopefully minor, extra workload involved when first configuring the project or adding additional modules as the project evolves.

## Source File Dependencies

Larger projects will use multiple source files to breakdown a large code base into smaller manageable units, probably using directories to group files in component subsystems. There will be interdependencies between these files. For C/C++ projects the dependencies can be identified through the occurrence of **#include** statements.

Large projects can take a while to compile and link all of the separate files from scratch (a clean build).  A large C project can take 20 or 30 minutes to build even using a fast multi-core server. For C++ projects making heavy use of templates the build times can be measured in hours rather than minutes.

Most build systems will optimise the build process by omitting stages that are already up to date. For C/C++ builds this means omitting the compilation of a source file if neither the source nor any of the files it depends have been changed since the last build.

But a build optimisation only works correctly if the build configuration correctly captures the dependencies between files. For simple build system like GNU Make the developer must specify and maintain these dependencies manually. A build system generator like CMake will scan the source files to maintain the dependencies automatically.

## Compilation Options

Compilation options must specify:

- the source language version, sometimes the source language
- compilation options
- include file locations
- preprocessor symbols (or defines)

Typically, compilation options are provided as command-line parameters but there is no reason why a compiler couldn’t read a configuration from a specification file.

As an example of a compilation option the GNU **g++** compiler uses the **\-std=c++17** for working with C++17 whereas the Microsoft **cl** compiler uses **/std:c++17**. It’s also worth pointing out that the [GNU Compiler Collection](https://gcc.gnu.org/) uses different compiler for C (**gcc**) and C++ ( **g++**), but Microsoft supplies one compiler (**cl**) and uses the filename extension to determine the language (**.c** or **.cpp**).

A build file generator such as CMake or Meson must handle these different approaches.

An essential compilation option is setting the correct level of compiler warnings. A C/C++ compiler will attempt to generate code whenever possible; only if code cannot be generated will a compilation error be issued. This means that the compiler sometimes makes assumptions about what the programmer intended when writing the source code statements. The phrase “never assume because you make an [ASS out of U and ME](https://en.wikipedia.org/wiki/Jerry_Belson)” has some significance here.

Using **g++**, we recommend, as a minimum, using the **\-Wall** and **\-Wextra** options to enable warnings for code use that is generally regarded as questionable (this doesn’t include implicit type conversion). The **\-pedantic** and/or **\-ansi** options enforce strict ISO (formerly ANSI) language compliance which is advisable as it will ensure you don’t make use of **g++** specific features and pre-empt potential problems if you decide to use a different tool chain in the future.

Other compilation options are used to generate warnings when the compiler infers a programmer’s intentions when the source code is not explicit. A good example of the compiler inferring the programmer’s intentions where the source code is not explicit is the implicit conversion of a signed to an unsigned integer value. Implicit sign conversion can cause subtle problems when working with hardware device registers, so we usually add the **\-Wconversion** and **\-Wsign-conversion** warnings on **g++** to identify these situations.

In general, set warnings to the highest level possible and remove all, or at least as many warnings as possible, from the compilation.

Your compilation phase should be augmented by static analysis tools such as clang-tidy, cppcheck or commercial tools such as Coverity which examine code structure and data flow without executing the code. These tools use heuristic rules to identify potential logic flaws and non compliance to coding guidelines such as [MISRA](https://www.misra.org.uk/) widely used in embedded systems.

## Include File Locations

We find that when teaching the use of **#include** preprocessor directive is a common source of confusion, even amongst experienced C/C++ programmers. We’re often asked what is the difference between using angle brackets **< >** and quotes **” “**?

Originally angle brackets were used for header files in standard locations known to the compiler, whereas quotes were used to define a string literal specifying the path to the header file (relative to the project workspace). These days life isn’t quite that clear cut, but the general approach is to use **< >** for library headers and **“”** for user-defined headers.

The ISO C and C++ standards both say that “The named source file is searched for in an implementation-defined manner” for both **< >** and **” “** (see the “Source File Inclusion” section in the relevant standard).

The C++ standard header files are defined with logical module names like iostream whereas in C we use the header filenames like **stdio.h**. When using C header files with C++ the logical name is the base part of the filename prefixed by `c` so <**stdio.h**\> becomes **cstdio**.

The compiler knows where standard header files are located. For example, the Linux host **g++** compiler looks in **/usr/include** whereas the Arm **g++** compiler (located at **/opt/arm-toolchain/bin/arm-none-eabi-g++**) looks in **/opt/arm-toolchain/include**. There is a full description for hosted GNU compilers in the [Search Path](https://gcc.gnu.org/onlinedocs/cpp/Search-Path.html) section of the manual.

We can tell the compiler to look in other locations using the **\-I** directive which specifies an additional directory to search for include headers. Note that this is just the top-level directory and not a recursive directory search.

We can use multiple include path locations on a compilation so we could include nested include directories as separate **\-I** options. For example, when developing out embedded target code for an STM Discovery board ([https://www.st.com/en/evaluation-tools/stm32f4discovery.html](https://www.st.com/en/evaluation-tools/stm32f4discovery.html)) we include standard header files using **\-Isystem/include/cmsis** and **\-Isystem/include/stm32f4xx** separately;  we cannot just use  **\-Isystem/include**. Note that these are relative paths from the project workspace (not an absolute path like **/usr/include**).

When using **#include** directives with string literals we are specifying file pathnames. But this approach can be abused by using a directive such as:

```
#include “../hdr/mylib.h”
```

This approach has coupled the organisation of the files on the file system to the C++ program code structure. We could not rename the **hdr** directory to **include** without modifying every source file that uses this header file. A maintenance nightmare – so this should be avoided.

To solve the file system dependency, we would add **\-Ihdr** to include the **hdr** folder in source include files search (some people prefer to use **\-i./hdr** to make it explicit this is relative to the project workspace). Our include path now becomes **“mylib.h”** without any path information. It is common to organise headers files in sub-directories so it would still be acceptable to use **“lib/mysublib.h”** as the header file organisation is part of the project structure.

The compiler uses the include locations to resolve all include directives so in the previous example we could also have written **#include <mylib.h>** but this would not follow the accepted conventions of using string literals for user defined header files.

In resolving include file paths modern compilers will typically look for the header file relative to the location of the source files containing the **#include** statement. If the header file is not found that way the compiler will search the specified include locations, in the order given on the command line, until it finds an exact filename match (Note, both Windows and OSX are case insensitive when searching for included filenames).

Some compilers may adopt a different approach such as initially looking for include files relative to the current working directory rather than the location of the source file. Defining and using a compiler’s **\-I** option (or equivalent) in a build system removes the dependency on a compiler’s include file lookup strategy.

The specified order of the include directories is therefore important and a possible source of problems if there are multiple header files with the same name: the first filename match is used and the same filename in subsequent include directories are ignored: duplicated include header filenames are not treated as an error. The best practice is to use unique header filenames or, failing that, use hierarchical directories to ensure unique paths. A good example of the directory approach is the standard Linux header **types.h** which is included as **<sys/types.h>**.

## Code generation Options

Code generation or assembler options are specified on the compilation command line and include general concepts like optimisation levels and architecture-specific options. For example, when cross compiling to an [Arm processor](https://gcc.gnu.org/onlinedocs/gcc/ARM-Options.html), we use **\-mcpu=cortex-m4** flag to set the correct architecture and **\-mfloat-abi=soft** to use a software FPU library because the QEMU we use for online training does not include support for a hardware FPU.

Code optimisation is typically disabled during development by using the **\-Og** option (the **g** is short for **gdb** the name of the Gnu debugger), but for a released version of the application (see build types described later) we may want to include some optimisation. For example, using **\-Ofast** will optimise for speed at the expense of a potentially larger memory footprint whereas **\-Os** will optimise for the smallest memory usage usually with slower code execution (the **s** means size).

Usually, we apply the same options to all compilation units (each source file is compiled independently) but sometimes it may be desirable to treat some source files differently. A good example would be targeting an embedded system with a limited amount of memory where we optimise for small size with **\-Os**. However, if one module is a critical performance bottleneck, we may need that compilation unit to be optimised for speed with **\-Ofast**.

Make sure whatever build system you use will support different compilation options for different source files: even if you don’t need it now, you might in the future.

## Preprocessor Directives

We use preprocessor directives to configure our source code so that we can use a single code base (one project) to build potentially different applications.

Examples of standard preprocessor symbols are **\_\_STDC\_VERSION\_** and **\_\_cplusplus** which can be used to verify the compiler options are set to the correct C or C++ language version.

To ensure we are using C++17 we would a check in our code such as:

```
#if __cplusplus < 201703L
#error “__FILE__ requires a C++17 compiler”
#endif
```

Here the symbol **\_\_FILE\_\_** is the filename of the current compilation unit.

If we knew we were always using a Modern C++ compiler (C++11 or later) we could also have used:

```
static_assert(__cplusplus >= 201703L)
```

A good example of user-defined configuration options can be seen in our embedded training projects currently using an STM Discovery board (STM32F407VG) with hardware components configured at specific addresses. We know that we may need to change this to a different board in the future; perhaps STM will stop manufacturing this particular board.

If we need to move to a different board with similar hardware components in the future, these components could be mapped to different physical addresses. We can build this dependency into our source code using conditional compilation based on preprocessor symbol definitions.

To resolve our theoretical problem of differing physical addresses we use pre-processor directives to include the appropriate device header file:

```
#ifdef STM32F407xx
#include “stm32f407xx.h”
#elif defined(STM32F417xx)
#include "stm32f417xx.h"
…
#endif
```

Our build configuration would define the appropriate preprocessor symbol on the command line: in this case using **\-DSTM32F407xx** to select the appropriate hardware configuration.

Similarly, we can use **\-DDEBUG** to define a debug symbol which we use to set options applicable to developing and debugging code such a **\-Og** to optimise for the debugger.

Compiler defines are used to support the concept of different **Build Configurations** which CMake refers to as *build type*, but an IDE usually calls a build configuration or build target.

## Build Configuration

A build configuration can be described as the combination of all the build options that uniquely define the application being built. Many projects have the idea of a **debug** build optimised for development and a **release** build optimised for use.

Each build configuration uses a separate output folder for the build artefacts (object files, executable and additional supporting files). This prevents a one build configuration from overwriting the output from a different configuration.

Build configurations can be used for many purposes. Our previous example of using two different target hardware boards would be separate builds for each hardware target. Actually, including debug and release versions for the two different hardware targets, that’s four different configurations and for separate build output locations.

We can also use build configurations to install our application in a central location or a web server where end-users can access and use our build artefacts.

A good build system will allow us to automate this deployment and/or installation process.

## Program Linker Configuration

Like the compiler, linker options are provided as command line parameters usually augmented by configuration files and run-time libraries. The linking stage for embedded systems is often more complex than the compilation stage because it as at this point that run time libraries, and the physical memory architecture have to be resolved to create a image suitable for the target board.

When using the Gnu toolchain, the use of linker options can be confusing because the same command (**gcc** or **g++**) is used for both compilation and linking. Linker options specified to a compilation are ignored, similarly compiler options are ignored when linking. Working with Arm `g++` we might build a system using a compilation command and a link command:

```
$ g++ -c -o build/main.o -std=c++17 --specs=rdimon.specs src/main.cpp
$ g++ -o build/Application.elf -std=c++17 --specs=rdimon.specs main.o
```

While this works it isn’t clear that the **std=c++17** option is probably not used by the linker. Similarly, the **–specs** option tells the linker to include a debug version of the embedded C standard library, which isn’t required by the compiler.

Microsoft developers have a separate **link.exe** program used to link the application so the two build steps are clearly differentiated.

A good build system will clearly differentiate between compiler and linker options.

On a host compilation the linker is specific to the host and simply needs to be given the list of optional run time libraries used by the project. In our C++ courses we discuss threading which requires the linker to include the POSIX threading library using the **g++** option **\-lpthread**.  The other common linking requirement is for handling libraries: should they be included in the executable image (static linkage), or use dynamic linking to a shared library (**.so** or **.dll** file) at application startup.

Using a development toolchain for an embedded (cross compiled) system has more complex linkage requirements. The physical board memory layout must be supplied to the linker: the [GNU Arm Embedded Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm)  uses **.ld** files for this purpose. There will be runtime kernel or executive code that executes at board reset to initialise the hardware, setup the stack, heap and static data sections before calling the **main** function to start the program.

An embedded system linker needs to include the C/C++ runtime libraries. In the case of the GNU Arm Embedded Toolchain these are provided by the  **–specs** options. For a debug version the **rdimon.specs** is linked to support semi-hosted debugging (I/O streams are mapped onto the serial port used to flash memory), while a release version uses **nosys.specs** which has stubs for standard I/O support and unsupported host system functions.

## Post Build Processing

The last step in our build (if we ignore deployment and installation) is any additional processing required after a successful build.

For a hosted application, we might strip the generated executable to remove all embedded symbols and other redundant information to reduce the size of the executable image.

For a cross-compilation to target hardware, we usually have to generate additional binary (or hex) files containing the image to load into flash memory. If we have embedded debug support, we will need map files to support post-crash analysis it allows the programmer to understand and review the target memory layout.

## Managing Testing and Debugging

Testing and debugging is too extensive a subject to cover in any detail in this post, but a build system should be capable of running automated tests at any point of the build cycle. Test management should include unit testing on a per-source code module, integration testing, and functional testing of the various build artefacts generated by the build process.

Using debugging tools such as [Open OCD](https://openocd.org/)  and [Segger Ozone](https://www.segger.com/products/development-tools/ozone-j-link-debugger/) while a necessary part of development may be too much for a build system to manage. Typically build systems like to create artefacts such as object files, executable images or other output files such as generated web pages for reports and build summaries. Interactive debugging does not fit very well with this approach.

## Limitations of a  Build System

Tools such as CMake have a very extensive “programming language” used to configure the build process. While it may appear attractive to use this programmable capability to incorporate additional steps to the build system this can end up adding huge complexity to the build instructions. It may be better kept some aspect oft the development lifecycle decoupled from the build system.

Moving away from a Build System will introduce problems to do with portability and maintenance of the supporting scripts. Bash scripts are supported on Linux and OSX but on Windows are provided by third party libraries such as [MinGW](https://en.wikipedia.org/wiki/MinGW) or [Cygwin](https://en.wikipedia.org/wiki/Cygwin). Recently Microsoft have been integrating [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/about) more closely with Windows 10 and this provides another source for running shell scripts. Using bash scripts for Windows developers in not straightforward. Similarly PowerShell is standard on Windows 10, but has to be installed on Linux and OSX.

While Python is cross platform some of the standard libraries are only available on Linux like platforms (usually including OSX) while others are only available on Windows. Python’s popularity is partly down to the extensive range of third-party libraries available, which would have to be installed on the developer’s workstations.

Stepping away from the build tool to provide additional development lifecycle support but that loses the inherent portability of a build tool like CMake. Developers can find themselves deciding to use a build tool to configure the build environment before they can even start development work on a project.

