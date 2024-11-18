#VSCODE #MCU #C #CMAKE
An important part of every [CI/CD pipeline](https://mcuoneclipse.com/2023/10/02/ci-cd-for-embedded-with-vs-code-docker-and-github-actions/) is having a testing phase. In this article I show how to use GNU gcov (coverage) with an embedded target, using Visual Studio Code as front end:

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/gcov-with-vs-code.jpg?w=1024)

GNU gcov with VS Code

With this, I can run the code on the embedded target which stores the coverage data on the host.

## Outline

Collecting coverage metrics is an essential part of the testing phase. While collecting coverage on a host application is rather simple, it is more challenging with an embedded target. In this article I show I’m doing this with VS Code, GNU gcc and gcov on a NXP LPC55S16 embedded target.

I have published the project used in this article on [GitHub](https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/VisualStudioCode/LPC55S16-EVK/LPC55S16_gcov).

## Requirements

I’m using VS Code 1.84.2 with the [NXP-LPC55S16-EVK](https://mcuoneclipse.com/2021/12/30/lorawan-with-nxp-lpc55s16-and-arm-cortex-m33/) board. If you follow the instructions below you should be able to collect coverage information for any other board.

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/lpc55s16-evk.jpg?w=597)

### Toolchain

The important part is that you need a toolchain and library which has **implemented the gcov library**. Unfortunately, it seems that the original ARM builds of the GNU toolchain after version 10 have dropped that support. GNU ARM Embedded Toolchain **v12.2.1-1.2** in the XPack distribution has everything included, so I recommend using that toolchain from here: [https://github.com/xpack-dev-tools/arm-none-eabi-gcc-xpack/releases/tag/v12.2.1-1.2](https://github.com/xpack-dev-tools/arm-none-eabi-gcc-xpack/releases/tag/v12.2.1-1.2). Other tool chains might work too.

> 💡 Make sure you copy the arm-none-eabi-gcov and name it gcov, as host tools will need it with that name.

### Semihosting

The other aspect is **[file I/O semihosting](https://mcuoneclipse.com/2023/03/09/using-semihosting-the-direct-way/)** which is required to write the coverage data to the host. Here again several bundled tool chains or SDK fail with this. If using the RP2040, check out my article here: [Implementing File I/O Semihosting for the RP2040 and VS Code](https://mcuoneclipse.com/2023/10/24/implementing-file-i-o-semihosting-for-the-rp2040-and-vs-code/). Here again I recommend the XPack v12.2.1-1.2 toolchain mentioned above, as it comes with everything needed.

### Debug Probe

Last but not least: the debug probe. You will need a debug probe which is supported by VS-Code, for example the [NXP MCU-Link (CMSIS-DAP/OpenOCD)](https://mcuoneclipse.com/2020/11/29/new-mcu-link-debug-probe-from-nxp/) and a [SEGGER J-Link EDU](https://mcuoneclipse.com/2023/09/07/new-segger-j-link-ob-lpc4322-based-firmware-with-target-power/).

![MCU-Link debugging the LPC845-BRK Board](https://mcuoneclipse.com/wp-content/uploads/2020/11/mcu-link-debugging-the-lpc845-brk-board.jpg?w=1024)

MCU-Link debug probe

In the next sections I explain what is needed to run gcov and related tools with VS Code, with the example provided on [GitHub](https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/VisualStudioCode/LPC55S16-EVK/LPC55S16_gcov).

## gcov Directory

To make using gcov easier, I have added a folder named ‘gcov’ with support and test files to the project:

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/gcov-directory.jpg?w=318)

gcov directory

That CMake folder is added in the main CMakeLists.txt as below:

```
add_subdirectory(./gcov         gcov)
```

I’m using the [NewLib](https://mcuoneclipse.com/2023/01/28/which-embedded-gcc-standard-library-newlib-newlib-nano/) standard library. Depending on your GNU library version, you might have a missing `[_sbrk()](https://mcuoneclipse.com/tag/sbrk/)` implementation. An implementation of it is provided inside `gcov_support.c`

## Semihosting with rdimon

I need file I/O semihosting, so I had to create an add a custom rdimon library (see [Implementing File I/O Semihosting for the RP2040 and VS Code](https://mcuoneclipse.com/2023/10/24/implementing-file-i-o-semihosting-for-the-rp2040-and-vs-code/)). It gets added to the list of CMake directories in the main CMakeLists.txt:

```
add_subdirectory(${MCULIB_DIR}/rdimon   rdimon)
```

I have configured and enabled the library in IncludeMcuLibConfig.h:

| 1  2  3  4 | `#define McuRdimon_CONFIG_IS_ENABLED       (1)       /* 1: RdiMon is enabled; 0: RdiMon is disabled*/` |
| --- | --- |

Additionally, because the level of semihosting depends on the debug probe, I have to specify he debug probe I want to use, e.g. a SEGGER J-Link:

| 1  2  3 | `#define McuSemihost_CONFIG_DEBUG_CONNECTION         McuSemihost_DEBUG_CONNECTION_SEGGER` |
| --- | --- |

If using the NXP MCU-Link CMSIS-DAP debug probe:

| 1 | `#define McuSemihost_CONFIG_DEBUG_CONNECTION  McuSemihost_DEBUG_CONNECTION_LINKSERVER` |
| --- | --- |

## Linking Libraries

I have to link my application with the gcov (wrapper) library and the rdimon library: Below the list of libraries used:

```
target_link_libraries(
  ${EXECUTABLE}
  srcLib
  sdkLib
  McuLib
  rdimonLib # file I/O with semihosting
  gcovLib   # gcov wrapper library
  gcov      # GNU gcov library
)
```

## Instrumenting Source Files

To instrument the source files, have to compile the source with the `--coverage` flag. I can do this for all files in a directory this way:

```
# Set coverage option only for some files, otherwise use add_compile_options(--coverage)
add_compile_options(--coverage) # all files
```

If I only want or need o instrument some files, I can use the following, where I list the source files:

```
set_source_files_properties(
  main.c
  PROPERTIES COMPILE_FLAGS --coverage
)
```

See the `CMakeLists.txt` in the `src` directory.

## Platform Configuration

I like the idea to be able to turn coverage information on or off. One thing I have defined is a macro in `platform.h` which I can use to turn it on or off:

| 1 | `#define PL_CONFIG_USE_GCOV              (1 && McuRdimon_CONFIG_IS_ENABLED) /* if using gcov */` |
| --- | --- |

In platform.c I initialize the libraries accordingly. First I need the necessary header files:

| 1  2  3  4  5  6 | `#if McuRdimon_CONFIG_IS_ENABLED`  `#include "rdimon/McuRdimon.h"`  `#endif`  `#if PL_CONFIG_USE_GCOV`  `#include "gcov_support.h"`  `#endif` |
| --- | --- |

Then I initialize the libraries:

| 1  2  3  4  5  6 | `#if McuRdimon_CONFIG_IS_ENABLED`  `McuRdimon_Init();`  `#endif`  `#if PL_CONFIG_USE_GCOV`  `gcov_init();  `  `#endif` |
| --- | --- |

## Writing Coverage Data

The application can write the coverage data with the following call:

In a FreeRTOS application, I can exit the scheduler with:

and then write the code at the place after where I have started the scheduler previously:

| 1  2  3  4 | `vTaskStartScheduler();`  `#if PL_CONFIG_USE_GCOV`  `gcov_write_files();`  `#endif` |
| --- | --- |

With this, I can cleanly exit the scheduler and at the end of the application I write the coverage data files.

## .gcno and .gcda Files

When compiling, the `gcc `compiler generates for the instrumented files a file with `.gcno` extension. You can observer this in the build folder:

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/gcno-files-1.jpg?w=433)

GNU gcov instrumented files (.gcno)

Running the application with the debugger and writing the data with semihosting (see above) then adds the data files with `.gcda` extension:

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/gcov-gcda-files.jpg?w=429)

gcov data files (.gdca)

It is possible to run the application multiple times, in multiple test runs. Then the data gets accumulated in the .gdca files.

It is possible to see the coverage data in the sources using the VS Code ‘Gcov Viewer’ from [Jacques Lucke](https://github.com/JacquesLucke/gcov-viewer):

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/vs-code-gcov-viewer.jpg?w=657)

VS Code Gcov Extension

The extension offers commands to reload the data and view the information:

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/gcov-viewer-commands.jpg?w=611)

VS Code Gcov Commands

With this, I see everything covered with green background color:

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/coverage-data-in-vs-code.jpg?w=1024)

Coverage Data shown in VS Code

## gcovr

One can use the `gcov `command to generate text reports. A better approach is to use the [gcovr](https://gcovr.com/) utility. Install it with

```
pip install gcovr
```

To generate a text report, run

```
gcovr .
```
![](https://mcuoneclipse.com/wp-content/uploads/2023/11/text-govr-report.jpg?w=872)

gcovr text report

To generate a HTML report, use

```
gcovr --html-details -o ./cov_report/main.html
```

To simply calling the `gcovr `tool, I have created an entry in `.vscode/tasks.json`:

```
        {
            "type": "shell",
            "label": "gcovr",
            "command": "gcovr --html-details -o ./cov_report/main.html",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "group": {
                "kind": "build",
                "isDefault": false
            },
        },
```

This report can be opened with a web browser or directly in VS Code:

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/gcov-with-vs-code.jpg?w=1024)

Coverage report in VS Code

## Summary

Generating test coverage data can be challenging with an embedded target, file I/O semihosting is a working solution. File I/O semihosting can be intrusive, but because I’m using a [direct semihosting](https://mcuoneclipse.com/2023/03/09/using-semihosting-the-direct-way/) and outside of the scheduler, things work out very well.

I hope this article gets you up and running with `gcov `and an embedded target.

Happy covering 🙂

### Links

- Project on GitHub: [https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/VisualStudioCode/LPC55S16-EVK/LPC55S16\_gcov](https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/VisualStudioCode/LPC55S16-EVK/LPC55S16_gcov)
- xPack project: [https://xpack.github.io/dev-tools/arm-none-eabi-gcc/](https://xpack.github.io/dev-tools/arm-none-eabi-gcc/)
- [CI/CD for Embedded with VS Code, Docker and GitHub Actions](https://mcuoneclipse.com/2023/10/02/ci-cd-for-embedded-with-vs-code-docker-and-github-actions/)
- [Implementing File I/O Semihosting for the RP2040 and VS Code](https://mcuoneclipse.com/2023/10/24/implementing-file-i-o-semihosting-for-the-rp2040-and-vs-code/)
- [GNU Coverage (gcov) with NXP S32 Design Studio IDE](https://mcuoneclipse.com/2022/03/22/gnu-coverage-gcov-with-nxp-s32-design-studio-ide/)
- [Tutorial: GNU gcov Coverage with the NXP i.MX RT1064](https://mcuoneclipse.com/2021/09/19/tutorial-gnu-gcov-coverage-with-the-nxp-i-mx-rt1064/)
- [Code Coverage with gcov, launchpad tools and Eclipse Kinetis Design Studio V3.0.0](https://mcuoneclipse.com/2015/05/31/code-coverage-with-gcov-launchpad-tools-and-eclipse-kinetis-design-studio-v3-0-0/)
- [Tutorial: GNU Coverage with MCUXpresso IDE](https://mcuoneclipse.com/2021/02/01/tutorial-gnu-coverage-with-mcuxpresso-ide/)
- [Debug Probes for RP2040 with VS Code](https://mcuoneclipse.com/2023/10/22/debug-probes-for-rp2040-with-vs-code/)
- VS Code GCov Viewer: [https://github.com/JacquesLucke/gcov-viewer](https://github.com/JacquesLucke/gcov-viewer)
- Gcovr utility: [https://gcovr.com](https://gcovr.com/)