#CMAKE #MCU

- [Introduction](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#Introduction "Introduction")
- [CMake on Windows](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#CMake_on_Windows "CMake on Windows")
- [Toolchain Configuration](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#Toolchain_Configuration "Toolchain Configuration")
- [Project Configuration](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#Project_Configuration "Project Configuration")
- [CMake Command Line](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#CMake_Command_Line "CMake Command Line")
- [WSL and Docker](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#WSL_and_Docker "WSL and Docker")
- [VirtualBox and VMware](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#VirtualBox_and_VMware "VirtualBox and VMware")
- [Summary](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#Summary "Summary")
- [Postscript – Simple Build Scripts](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#Postscript_%E2%80%93_Simple_Build_Scripts "Postscript – Simple Build Scripts")
- [Windows Configure Script](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#Windows_Configure_Script "Windows Configure Script")
- [Windows Build Scripts](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#Windows_Build_Scripts "Windows Build Scripts")

## Introduction

In previous blog posts in this series ([Part 1](https://feabhasblog.wpengine.com/2021/07/cmake-part-1-the-dark-arts/),  [Part 2](https://feabhasblog.wpengine.com/2021/07/cmake-part-2-release-and-debug-builds/) and [Part 3](https://feabhasblog.wpengine.com/2021/08/cmake-part-3-source-file-organisation/)), I looked at using [CMake](https://cmake.org/) on a Linux host to configure a build to [cross compile](https://en.wikipedia.org/wiki/Cross_compiler) to target hardware such as the [STM32F4 Series](https://www.st.com/en/microcontrollers-microprocessors/stm32f4-series.html).

In this post, we’ll work with the [GNU Arm Embedded Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm) on a Windows 10 Host.

The first part of this blog discusses running the Windows hosted versions of CMake, GNU Arm Embedded Toolchain and GNU Make. An alternative approach, briefly discussed at the end of the blog, is to use container technology such as Windows Subsystem for Linux (WSL2) or Docker, or use a full-blown Linux Virtual Machine  hosted in VirtualBox or VMWare.

## CMake on Windows

The first point to make about CMake on Windows is that it defaults to generating build files for [Visual Studio](https://visualstudio.microsoft.com/) and assumes you will be using the [Microsoft Visual Studio Toolset](https://docs.microsoft.com/en-us/cpp/build/building-on-the-command-line?view=msvc-160).

The second point to make is that the CMake command line is subtly different. Not by much, but enough to confuse  us, as some options that work under Linux are not available under Windows.

The third point is around command and file naming conventions. Microsoft uses a **.exe** suffix to identify executable programs, but there is no requirement to include this suffix when invoking an executable from the command line. Running the C/C++ compiler is a matter of entering the command CL or CL.EXE (case is ignored but is usually shown as uppercase in documentation). CMake may require the full pathname to the compiler executable, including the **.exe** suffix. This isn’t used with Linux executables, leading to a minor difference between the command name (CL) and the executable file name (CL.EXE) not found under Linux.

The fourth and last point is that CMake generates build files for Microsoft [NMake](https://docs.microsoft.com/en-us/cpp/build/reference/nmake-reference?view=msvc-160). Running an NMake build requires a custom environment to be set up by running the **vcvarsall.bat** supplied with the Microsoft VS Toolset. This isn’t a big problem, but any automated build script must include this.

We need to modify our Linux CMake configuration and supporting build script to address these portability problems. I’ll assume you’re familiar with the Linux based embedded system project that we’ve used in previous posts and just focus on changes required for Windows.

## Toolchain Configuration

By default, The GNU Arm Toolchain for Windows is installed in the **C:\\Program Files (x86)\\GNU Arm Embedded Toolchain\\** as a subfolder named after the release version. The Arm toolchain does not include the GNU Make command, so we must download this separately, either as a standalone program or as part of a suite of GNU development tools.

Installing GNU development tools on a Linux host is achieved using the system package management commands (**apt** for Debian/Ubuntu and **dnf** or **yum** for Fedora/CentOS/RHEL). However, it’s a little more complex on Windows as there is no official Microsoft package of GNU tools.

We, therefore, need to rely on third-party providers for the GNU development tools, of which [MinGW](https://mingw-w64.org/doku.php) and [Cygwin](https://www.cygwin.com/) are the most popular.

Additionally there are various standalone versions of GNU Make ported to Windows: but none of the one’s I’ve come across appear to be supported or updated on a regular basis which makes me wary of using them.

I’m not going to digress into the details of installing GNU Make under Windows but assume that a Windows version of the **make** command is available. In my case, I use the [MSYS2](https://www.msys2.org/) installer which includes the Mingw-w64 development tools (these must be added to the base MinGW installation).

We already have a working CMake toolchain file for our embedded project  (**toolchain-STM32F407.cmake**), but we will need to make some minor modifications to handle the **.exe** suffix on the toolchain filenames (not found on Linux). The [GitHub project](https://github.com/feabhas/cmake-blog-4) supporting this blog has all the configuration files for Windows.

For CMake we will assume that our Windows environment path variable (%PATH%) is configured to include the directories containing the GNU Arm toolchain (as we have done for the Linux build). This simplifies the toolchain file changes and avoids hard coding filesystem paths into the configuration file. We use a PowerShell script (shown later) to configure the Windows program search path (%PATH%).

In our sample project’s toolchain file (**toolchain-STM32F407.cmake**) we just add conditional code for including executable suffixes when locating the toolchain executable files:

```
if (CMAKE_HOST_WIN32)
  set (SUFFIX .exe)
else()
  set (SUFFIX "")
endif()

find_program(CROSS_GCC_PATH arm-none-eabi-gcc${SUFFIX})
get_filename_component(TOOLCHAIN ${CROSS_GCC_PATH} PATH)

set(CMAKE_C_COMPILER ${TOOLCHAIN}/arm-none-eabi-gcc${SUFFIX})
set(CMAKE_Cxx_COMPILER ${TOOLCHAIN}/arm-none-eabi-g++${SUFFIX})
set(TOOLCHAIN_AS ${TOOLCHAIN}/arm-none-eabi-as${SUFFIX} CACHE STRING "arm-none-eabi-as")
set(TOOLCHAIN_LD ${TOOLCHAIN}/arm-none-eabi-ld${SUFFIX} CACHE STRING "arm-none-eabi-ld")
set(TOOLCHAIN_SIZE ${TOOLCHAIN}/arm-none-eabi-size${SUFFIX} CACHE STRING "arm-none-eabi-size")
```

If we are on Windows, The CMAKE\_HOST\_WIN32 variable is set to **true**, allowing the script to set a variable with the required host filename suffix. No other changes to the toolchain file **toolchain-STM32F407.cmake** are required.

As an aside, if we had decided to adopt a simpler approach for our toolchain configuration, where we only required the compiler and linker without the other build tools, then we could have just specified the compiler command names in the appropriate CMAKE variables:

```
set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER arm-none-eabi-g++)
```

In this case, we are working with command names and not the filenames, so there is no requirement to include a **.exe** suffix. Configuring the toolchain using this approach means no changes are required to the toolchain file to work with Windows rather than Linux. Our changes are required because we are working withe filenames not commands.

## Project Configuration

No changes are required to the project file **CMakeLists.txt,** which shows that CMake can be configured to work with both Linux hosts (including macOS) and Windows with a single version of the configuration files.

But we do need to look at changes required to the **cmake** command line to generate the build files and run the build itself.

## CMake Command Line

On Linux, we typically add our development commands to the standard search path (defined by the $PATH variable). In contrast, on Windows, we tend not to extend the Windows search path to include development tools mainly because we are less likely to be working at a Windows command line.

By not extending the Windows search path,  we must use full path names for each toolchain command or temporarily extend the path to include the toolchain folder location. As with Linux, a script to manage the build process is essential.

The first change to the **cmake** command is to specify using the GNU Make code generator instead of Visual Studio Tools by adding a **\-G “Unix Makefiles”** option.

Not only do we need to use the **-G** option, but we need to specify the location of the **make** command otherwise we will default to using **nmake** which we haven’t installed as we are not using the Microsoft host toolchain.

To make it easier to read and maintain a command script, we define a Windows variable for the location of the **make** program rather than include it on the search path. But we will extend the search path (%PATH%) to include the required GNU Arm toolchain folder.

A simple set of Windows commands to generate a debug build using **msys64** Make and GNU Arm toolchain version **2020-q4-majo**r looks like the following (the **^** symbol is the command continuation character for Windows):

```
set CMAKE=”C:\Program Files\CMake\bin\cmake.exe”
set MAKE=”C:\msys64\usr\bin\make.exe”
set ARMTOOLS=C:\Program Files (x86)\GNU Arm Embedded Toolchain\10 2020-q4-major\bin
set PATH=%PATH%;%ARMTOOLS%

%CMAKE% -S . -B build/debug   ^
  -G “Unix Makefiles”         ^
  -DCMAKE_MAKE_PROGRAM=%MAKE% ^
  -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake
```

Currently, the Windows version of the **cmake** command does not support the **–build** option, so we have to invoke the **make** command directly from the command line. Using the **\-C** option, we can avoid changing directories to work within our project root folder. We can optionally add a VERBOSE=1 to the end of the command to see the build commands as they are executed.

To build the debug version of our project we use:

```
set MAKE="C:\msys64\usr\bin\make.exe"
%MAKE% -C build/debug VERBOSE=1
```

A clean build requires adding the **clean** target to the **make** command:

```
set MAKE="C:\msys64\usr\bin\make.exe"
%MAKE% -C build/debug clean
```

That’s it. All the other CMake command options we used in the Linux build work under Windows. At the end of the blog is an example of this simple command script and a more functional PowerShell script that wraps up the CMake build commands.

An alternative to using the Windows hosted ARM toolchain is to use a virtual environment or container to perform the build under Linux.

## WSL and Docker

Both [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) (WSL2) and [Docker](https://www.docker.com/) are containers or self contained execution environments that run Linux and have access to the Windows file system but typically don’t provide a desktop environment (but could do so). We discuss using Docker containers in out blog [An Introduction to Docker for Embedded Developers – Part 1 Getting Started](https://feabhasblog.wpengine.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/).

For both WSL2 and Docker, the host development toolchain (for the **make** command) and the GNU Arm Embedded Toolchain (for Linux) need to be installed in the container.

These days both WSL2 and Docker can be accessed from [Visual Studio Code](https://code.visualstudio.com/) running on the Windows host. Microsoft extensions (**Remote – WSL** and **Remote – Containers**) are required to access the virtual environments, and there are additional third-party VS Code extensions mainly for Docker but also for WSL. This means you can store and edit the code on Windows filesystem but run the build commands in the WSL2 or Docker container, all from within the Visual Studio Code IDE.

I have found using WSL2 from VS Code an effective mechanism for developing and building our training projects. The resultant ELF file in the Windows filesystem can be downloaded to our target hardware (we use [Segger Ozone](https://www.segger.com/products/development-tools/ozone-j-link-debugger/)) or run in our customised version of the [XPack QEMU emulator](https://xpack.github.io/qemu-arm/) from within windows.

## VirtualBox and VMware

At Feabhas, we use VirtualBox to build self-contained Linux VMs and distribute these for online training as [Open Virtualization](https://en.wikipedia.org/wiki/Open_Virtualization_Format) or OVA files. Both VMWare and VirtualBox can import OVA files and be configured to access folders on the host through their *Shared folder* settings. But both products require additional software to be installed in the Linux guest operating system to gain access to the Windows host filesystem.

We use VirtualBox (without shared folders) to build our training projects and run the compiled ELF image in our custom version of QEMU on the Linux guest. We can also map the [JLink](https://www.segger.com/downloads/jlink/) USB port from the Windows host into the VM in order to use Ozone (in the VM) to download the ELF images to our target hardware.

VirtualBox and VMware both provide a good environment for developing embedded projects on a Windows Host.

**Note:** WSL2 (and Docker on Windows 10 Pro) use the Microsoft Windows Hypervisor Platform (Hyper-V) feature which, up until recently, has prevented VirtualBox and VMWare VMs from running correctly (see [WSL2 FAQ](https://docs.microsoft.com/en-us/windows/wsl/wsl2-faq)). Recent versions of VMWare and VirtualBox (July 2021) can now coexist with WSL2 and the Hyper-V platform and hopefully will continue to do so. There does appear to be a noticeable drop off in the performance of VirtualBox emulation when the Hyper-V platform is enabled, so my personal preference is to work with WSL2.

## Summary

While I prefer Linux and macOS, I use Windows; it is my primary development environment. I generally work directly with the Windows version of the Arm and Segger tools for embedded system development. Recent improvements to WSL and VS Code means that I now find this combination as good as Windows hosted tools and easier to use than working with VirtualBox (or VMWare).

Initially, getting CMake to build an embedded (cross compiler) project under Windows was painful: a term often used when discussing CMake, and Windows for that matter. Once I’d worked out that switching to the GNU Toolchain did not also default to using GNU Make rather than NMake, it turned out to be straightforward to create a portable configuration. If we hadn’t wanted to find the paths to the additional GNU Arm build commands (like **as** and **ld**) we would not have had to make any changes to the CMake configuration files whatsoever.

But, as with Linux, it is the complex and necessary command line options used to set up the toolchain and build configurations that are the main problem.

It’s a shame that the Windows version of CMake does not support the **–build** option so that we have to revert to the old-fashioned approach of running **make** directly.

It would have been easier to work on Windows if the GNU Arm Embedded Toolchain (for Windows) included a version of **make** so that we didn’t have to find and install it from elsewhere.

And finally, using a Windows command or PowerShell script to simplify using the **cmake** and **make** commands is essential.

A later article on [CMake Presets](https://feabhasblog.wpengine.com/2023/08/cmake-presets/) describes how to use the [presets](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html) feature added at CMake 3.19 in 2020.

## Postscript – Simple Build Scripts

The [GitHub project](https://github.com/feabhas/cmake-blog-3) supporting this blog contains a command script (**configure.bat**) containing the build commands in this blog. The repo also includes a more functional PowerShell script (**build.ps1**) with a supporting Command script (**build.bat**) for building debug and release projects under Linux.

### Windows Configure Script

A simple command script (**configure.bat**) based on the examples in the blog:

```
set CMAKE=”C:\Program Files\CMake\bin\cmake.exe”
set MAKE=C:\msys64\usr\bin\make.exe
%CMAKE% -S . -B build/debug   ^
  -G "Unix Makefiles"         ^
  -DCMAKE_MAKE_PROGRAM=%MAKE% ^
  -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake
%MAKE% -C build/debug VERBOSE=1
```

### Windows Build Scripts

A more complex PowerShell script (**build.ps1)** supports command line options, but this must be invoked via a command script (**build.bat**) to configure the security permissions.

Windows applies security restrictions to prevent running unsigned PowerShell scripts from the command prompt (or from the Visual Studio Code tasks). The supporting **build.bat** script is used to start the PowerShell build script without security checks:

```
powershell.exe -noprofile -executionpolicy bypass -file build.ps1 %*
```

The **build.ps1** script is a port of the shell script (**build.sh**) for Linux:

```
Set-StrictMode -version latest

$SCRIPT = Split-Path $PSCommandPath -Leaf;
$USAGE = "Usage: $SCRIPT [-v | --verbose | --rtos] [ reset | clean | debug | release ]"

$CMAKE = 'C:\Program Files\CMake\bin\cmake.exe'
$MAKE = 'C:\msys64\usr\bin\make.exe'
$ARM_TOOLCHAIN = 'C:\Program Files (x86)\GNU Arm Embedded Toolchain\10 2020-q4-major\bin'

$env:PATH += ";$ARM_TOOLCHAIN"

$BUILD= 'build'
$BTYPE = 'DEBUG'
$BUILD_DIR = "$BUILD\debug"
$CLEAN = ''
$RESET = ''
$VERBOSE = ''
$RTOS = ''

switch -regex ($args)
{
  '^(--help|-h|)$'    { Write-Output "$USAGE"; exit 0 }
  '^(--verbose|-v)$'  { $VERBOSE = 'SHELL="/bin/sh -x"'  }
  '^--rtos$'          { $RTOS = '-DUSE_RTOS=ON'  }
  '^debug$'           { $BTYPE = 'DEBUG';   $BUILD_DIR = "$BUILD\debug" }
  '^release$'         { $BTYPE = 'RELEASE'; $BUILD_DIR = "$BUILD\release"  }
  '^clean$'           { $CLEAN = '1'  }
  '^reset$'           { $RESET = '1'  }
  default             { Write-Error "Unknown option $arg"; Show-Usage }
}

if ( $RESET -and (Test-Path $BUILD_DIR -PathType Container) ) {
  Remove-Item $BUILD_DIR -Recurse
}

$TOOLCHAIN = '-DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake'
$CMAKE_ARGS = '-G', 'Unix Makefiles', "-DCMAKE_MAKE_PROGRAM=$MAKE"
$BUILD_TYPE = "-DCMAKE_BUILD_TYPE=$BTYPE"

&$CMAKE -S . -B $BUILD_DIR $CMAKE_ARGS \`
  --warn-uninitialized $BUILD_TYPE $TOOLCHAIN $RTOS

if ( $CLEAN  -ne '' ) {
  &$MAKE -C $BUILD_DIR clean
}

&$MAKE -C $BUILD_DIR $VERBOSE
```

- [About](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#abh_about)
- [Latest Posts](https://blog.feabhas.com/2021/09/cmake-part-4-windows-10-host/#abh_posts)

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