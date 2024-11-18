#CMAKE #MCU #NXP
One key element of a CI/CD pipeline is the automatic testing phase: whenever I check in new source code or manually trigger it, I can run a test suite to make sure that the changes do not break anything. For this, I have to run automated tests. For an an embedded target, it means that I have to run some tests on the board itself too.

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/ctest-in-vs-code.jpg?w=1024)

CTest with VS Code

## Outline

In a [TDD](https://en.wikipedia.org/wiki/Test-driven_development) (Test Driven Development) world you would first write the tests, then develop the software. That would be ideal, but as we all know: the world is not ideal. Instead, in many cases you might have to work on legacy code, and adding tests later on. In this article I describe, how tests can be added to a CMake based project, using the Unity test framework and **CTest as testing driver**.

For example: in a embedded project where LPC55S16 is used to communicate over LoRaWAN, we need to run a lot of tests on the actual hardware:

![](https://mcuoneclipse.com/wp-content/uploads/2021/12/lorawan-with-lpc55s16-evk.png?w=1024)

LoRaWAN with LPC55S16

[CI/CD](https://en.wikipedia.org/wiki/CI/CD) (Continuous Integration / Continuous Delivery) means running lots of tests, and fast. Ideally you would run all the tests on the host. But certain tests needs to run on the hardware and can not run on the host or in a simulator or emulator. Here I show how to run tests **on-target** with the help of a debug probe.

You can find an example project using the approach presented here on [GitHub](https://github.com/ErichStyger/MCUXpresso_LPC55S16_CI_CD). I recommend that you have a read at my previous topic here: [CI/CD for Embedded with VS Code, Docker and GitHub Actions](https://mcuoneclipse.com/2023/10/02/ci-cd-for-embedded-with-vs-code-docker-and-github-actions/).

In this article I’m using [VS Code](https://mcuoneclipse.com/2023/10/14/semihosting-with-vs-code-on-rp2040/) with the [NXP LPC55S16-EVK](https://mcuoneclipse.com/2021/12/30/lorawan-with-nxp-lpc55s16-and-arm-cortex-m33/), but the concepts can be easily used for any other environment:

- How to structure test files with CMake and Presets
- Integrating the Unity test framework
- Writing tests with Unity
- Adding tests with CTest
- Configuring test properties with CTest
- Running Tests with JRun to program it, passing arguments and producing output with RTT
- Passing arguments to the application and producing test output
- Executing tests with CTest
- Configuring CTest with regular expressions for pass and fail
- Running tests with Visual Studio Code

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/testing-pipeline.jpg?w=841)

Test Pipeline

## Structuring Tests

There are many different ways to organize the test files. My preference is to have them (Unit Tests, System Tests, …) are organized in separate sub-folders, inside the project sources:

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/tests-folder.jpg?w=314)

Tests Folder

So for example the test files for the ‘`src`‘ folder are located in ‘`src/tests`‘, and so on. Each tests folder has its own `CMakeLists.txt` which is only used if I’m building for testing.

## Unity Testing Framework

As testing framework, I’m using [Unity from ThrowTheSwitch](http://www.throwtheswitch.org/unity). This is a very easy-to-use and powerful unit testing framework, but easily can be used for any other testing like integration or system level testing. The Unity test framework has been recently added to the [McuLib](https://github.com/ErichStyger/McuOnEclipseLibrary) library of embedded software.

The framework only has a few source files. Unity.h is the header file to be included in the test sources, and unity.c contains the framework to ramp-up and ramp-down the tests:

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/unitytestframework.jpg?w=347)

Unity Test Framework

Similar to other frameworks, Unity provids a rich set of assert macros. I can write with asserts like this:

| 1  2  3  4  5  6 | `void` `TestLeds_OnOff(``void``) {`  `Leds_On(LEDS_BLUE);`  `TEST_ASSERT_MESSAGE(Leds_Get(LEDS_BLUE), ``"Blue LED shall be on"``);`  `Leds_Off(LEDS_BLUE);`  `TEST_ASSERT_MESSAGE(!Leds_Get(LEDS_BLUE), ``"Blue LED shall be off"``);`  `}` |
| --- | --- |

For a full list of possible assertions see [https://github.com/ThrowTheSwitch/Unity/blob/master/docs/UnityAssertionsReference.md](https://github.com/ThrowTheSwitch/Unity/blob/master/docs/UnityAssertionsReference.md).

And then run multiple tests like this:

| 1  2  3  4 | `UNITY_BEGIN();`  `RUN_TEST(TestLeds_OnOff);`  `RUN_TEST(TestLeds_Toggle);`  `nofFailures = UNITY_END();` |
| --- | --- |

The UNITY\_BEGIN() starts the testing (initializes status, etc), then I run the tests with RUN\_TEST() and finally UNITY\_END() returns the number of failed tests.

Unity does not have any hardware dependency, so I need to configure it how it shall write the output messages. The framework is configured by the **McuUnity** module:

| 1  2  3  4  5  6 | `#include "McuUnity.h"`  `#define UNITY_OUTPUT_CHAR(a)                        McuUnity_putc(a)`  `#define UNITY_OUTPUT_FLUSH()                        McuUnity_flush()`  `#define UNITY_OUTPUT_START()                        McuUnity_start()`  `#define UNITY_OUTPUT_COMPLETE()                     McuUnity_complete()`  `#define UNITY_OUTPUT_COLOR                          /* use colored output */` |
| --- | --- |

For example it is configured to send all the output to the SEGGER [RTT](https://mcuoneclipse.com/2015/07/07/using-segger-real-time-terminal-rtt-with-eclipse/) communication channel in McuUnity.c:

| 1  2  3 | `void McuUnity_putc(int c) {`  `McuRTT_StdIOSendChar(c); /* using JRun with RTT */`  `}` |
| --- | --- |

I could use other ways than RTT, for example writing to a serial port or using semihosting.

## CMake Presets

I’m using a dedicated CMake Preset for testing (see [Building with CMake Presets](https://mcuoneclipse.com/2023/12/03/building-with-cmake-presets/)):

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/cmake-test-configuration.png?w=615)

CMake Test Preset

So I have configurations both for the ‘debug’ and ‘release’. For some systems it might make sense only to test the release build, but having a debug build makes it easier to locate testing errors or any other kind of issues.

The ‘test’ presets are setting a CMake variable **ENABLE\_UNIT\_TESTING**:

```
    {      "name": "debug-test",      "displayName": "Config Debug-Test",      "description": "Debug build with tests enabled",      "inherits": "debug",      "cacheVariables": {        "ENABLE_UNIT_TESTING": "ON"      }    },
```

That variable is used in the CMakeLists.txt to turn on or off testing support.

## Build with Testing Support

The CMake variable **ENABLE\_UNIT\_TESTING** is then used in the main CMakeLists.txt to set a compiler define:

```
# Check if ENABLE_UNIT_TESTING is set to ON in the CMake presetif (ENABLE_UNIT_TESTING)  message(STATUS "ENABLE_UNIT_TESTING is ON")  add_compile_options(-DENABLE_UNIT_TESTS=1) # used to enable tests in codeelse()  message(STATUS "ENABLE_UNIT_TESTING is OFF")  add_compile_options(-DENABLE_UNIT_TESTS=0) # used to disable tests in codeendif()
```

The #define is used in the application to check if it has to build and include the test functions:

| 1 | `#define PL_CONFIG_USE_UNIT_TESTS        (1 && defined(ENABLE_UNIT_TESTS) && ENABLE_UNIT_TESTS==1) /* if using unit tests. ENABLE_UNIT_TESTS is set by the CMake file */` |
| --- | --- |

| 1  2  3 | `#if PL_CONFIG_USE_UNIT_TESTS`  `Tests_Init();`  `#endif` |
| --- | --- |

CTest is part of the CMake software suite of tools: with added CTest support to the CMakeLists.txt, I can run tests at the end of a successful build:

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/cmake_ctest_workflow.png?w=1024)

CMake with CTest (Source: o3de.org)

CTest is a driver to kick off a test runner like [Google Test](https://github.com/google/googletest) or [PyTest](https://docs.pytest.org/en/7.4.x/), or anything you like to run:

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/ctest_to_runners.png?w=1024)

CTest (Source: o3de.org)

CTest gets enabled for CMake based projects by including the following in the `CMakeLists.txt:`

```
include(CTest)
```

I prefer to have verbose output, and I can do this with setting the CMake CTest arguments:

```
list(APPEND CMAKE_CTEST_ARGUMENTS "--output-on-failure")list(APPEND CMAKE_CTEST_ARGUMENTS "--verbose")
```

Finally I need to make sure that I include the Unity framework, as used by my tests:

```
add_subdirectory(${MCULIB_DIR}/unity   unity)
```

In a similar way I reference the tests for the src folder from its CMakeLists.txt and link with it:

```
if (ENABLE_UNIT_TESTING)  message(STATUS "Adding src tests")  add_subdirectory(./tests                  tests)endif()...if (ENABLE_UNIT_TESTING)  message(STATUS "Adding src tests library")  target_link_libraries(    ${THIS_LIBRARY_NAME}    PUBLIC srcTestsLib  )endif()
```

## JRun

To test the binary on the target, I have to deploy it, run it and analyze the output:

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/testdeploycommunication.png?w=951)

Running that on the host is easy: just run the executable:

```
$ .\testApp --test=1
```

On an embedded target things are a bit more complex, as I have to program the binary for example with a debugger or boot loader.

And unlike for running an executable on the host, I cannot easily pass arguments to the application under test (`--test=1`). Moreover, there is typically no standard input or output, so the application cannot simply `printf()` the test results.

A solution to this problem is using a debugger with [Semihosting](https://mcuoneclipse.com/2023/10/24/implementing-file-i-o-semihosting-for-the-rp2040-and-vs-code/) or [RTT](https://mcuoneclipse.com/2015/07/07/using-segger-real-time-terminal-rtt-with-eclipse/). But there is even a simpler way: SEGGER JRun: In this project I’m using the [SEGGER JRun](https://www.segger.com/products/debug-probes/j-link/tools/j-run/) with the [Unity](http://www.throwtheswitch.org/unity) testing library. JRun is part of the J-Link software suite.

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/ctest_to_jrun.png?w=191)

JRun with Unity and CTest

JRun is a command-line utility which can program a file like a debugger, but much easier to use.

```
J-Run compiled Dec 13 2023 17:09:30(c) 2019-2019 SEGGER Microcontroller GmbH    www.segger.comSyntax:  JRun [option option ...] elf-file  Option                     Default       Description  --usb <SerialNo>           not set       Set serial number of J-Link to connect to via USB.  --ip <str>                 not set       Set host name of J-Link to connect to via TCP/IP.  --device <str>, --d <str>  STM32F407IE   Set device name to str.  --if SWD | JTAG            SWD           Select SWD or JTAG as target interface.  --speed <kHz>              4000          Set interface speed to n kHz.  --rtt                      Auto          Explicitly enable RTT.  --nortt                    Auto          Explicitly disable RTT.  --semihost                 Auto          Explicitly enable semihosting.  --nosemihost               Auto          Explicitly disable semihosting.  --x str, --exit str        *STOP*        Set exit wildcard to str.  --quit                     On            Automatically exit J-Run on application exit.  --wait                     Off           Wait for key press on application exit.  --2, --stderr              Off           Also send target output to stderr.  --s, --silent              Off           Work silently.  --v, --verbose             Off           Increase verbosity.  --dryrun                   Off           Dry run. Parse elf-file only.  --jlinkscriptfile <str>    not set       Set path of J-Link script file to use to str. Further info: wiki.segger.com/J-Link_script_files
```

JRun can do Semihosting and RTT output and redirects it to the console. For example:

```
JRun --verbose --device LPC55S16 build/debug-test/LPC55S16_Blinky.elf    
```

JRun monitors the output itself, and it can be asked to exit if it gets a special “\*STOP\*” string. So we can write the overall test result (passed or failed) and then exit the application:

| 1  2  3  4  5  6 | `if` `(nofFailures==0) {`  `McuShell_SendStr((unsigned ``char``*)``"*** PASSED ***\n"``, McuRTT_stdio.stdOut);`  `} ``else` `{`  `McuShell_SendStr((unsigned ``char``*)``"*** FAILED ***\n"``, McuRTT_stdio.stdOut);`  `}`  `McuShell_SendStr((unsigned ``char``*)``"*STOP*\n"``, McuRTT_stdio.stdOut); ` |
| --- | --- |

Which then writes the output to the console:

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/vscodeconsoleoutput.jpg?w=820)

JRun Output on Console

## Parameter Passing

In the above case, both tests were executed together. But sometimes it can make sense to run separated tests, with something like passing an argument to the running application. One way to do this is to pass a parameter to the application using a [J-Link Script file](https://wiki.segger.com/J-Link_script_files).

The script file then looks like this which writes the value 1 at address 0x2000’0000:

| 1  2  3  4  5 | `int` `HandleAfterFlashProg(``void``) {`  `int` `res;`  `res = JLINK_MEM_WriteU32(0x20000000, 1);`  `return` `0;`  `}` |
| --- | --- |

The variable at address 0x2000’0000 is allocated in a ‘no-init’ section (see [this article](https://mcuoneclipse.com/2014/04/19/gnu-linker-can-you-not-initialize-my-variable/)):

| 1 | `uint32_t` `program_arg __attribute__((section (``".uninit"``)));` |
| --- | --- |

The script is passed on the command-line to JRun:

```
JRun --verbose --device LPC55S16 --jlinkscriptfile src/tests/test_1.JlinkScript build/debug-test/LPC55S16_Blinky.elf
```

With this I can select and choose tests at runtime:

| 1  2  3  4  5  6  7  8 | `int` `test_arg = McuUnity_GetArgument(); `  `UNITY_BEGIN();`  `switch``(test_arg) {`  `case` `1:   RUN_TEST(TestLeds_OnOff); ``break``;`  `case` `2:   RUN_TEST(TestLeds_Toggle); ``break``;`  `default``:  RUN_TEST(TestArgFailed); ``break``;`  `}`  `nofFailures = UNITY_END();` |
| --- | --- |

With this, I can select which test to run, or pass any other parameters to the application.

I have a feature request pending at SEGGER for JRun to have an argument for it on the command line, so I can pass a string or something else to it which then can be read by semihosting or RTT. That would make it more portable (currently I have a script creating the J-Link Script files). For example:

```
JRun --arg="test=5" .....
```

Crossing fingers that this gets implemented sometimes in the near future.

## Adding Tests

Adding tests with CTest uses the following pattern:

```
add_test(  NAME <testname>  COMMAND <command to be executed>)
```

For example I can add two tests like this:

```
set (JRUN_CTEST_COMMAND "JRun" --verbose --device LPC55S16 --rtt -if SWD)add_test(  NAME Led_1  COMMAND ${JRUN_CTEST_COMMAND} --jlinkscriptfile "${CMAKE_CURRENT_SOURCE_DIR}/test_1.JLinkScript" ${TEST_EXECUTABLE})add_test(  NAME Led_2  COMMAND ${JRUN_CTEST_COMMAND} --jlinkscriptfile "${CMAKE_CURRENT_SOURCE_DIR}/test_2.JLinkScript" ${TEST_EXECUTABLE})
```

Additionally I can set properties for the tests, like setting a timeout:

```
set_tests_properties(Led_1 Led_2 PROPERTIES TIMEOUT 15) 
```

## Pass or Fail

Finally, CTest needs to know if the tests were successful or not. CTest expects the test application to return 0 for no failure, and a negative return code for ‘test failed’. Unfortunately, JRun always returns 0 (that’s yet another pending feature request from my side). But I can tell CTest to use a Regular Expression instead on the output:

```
set (passRegex "\\*\\* PASSED \\*\\*\\*")set (failRegex "\\*\\*\\* FAILED \\*\\*\\*")set_property(TEST PROPERTY PASS_REGULAR_EXPRESSION "${passRegex}")set_property(TEST PROPERTY FAIL_REGULAR_EXPRESSION "${failRegex}")set_tests_properties(Led_1 Led_2 PROPERTIES PASS_REGULAR_EXPRESSION "${passRegex}") set_tests_properties(Led_1 Led_2 PROPERTIES FAIL_REGULAR_EXPRESSION "${failRegex}") 
```

With this, CTest properly detects failed tests. I can run CTest from the command-line:

```
ctest --extra-verbose --test-dir build/debug-test --timeout 120
```
![](https://mcuoneclipse.com/wp-content/uploads/2023/12/running-ctest-manually.jpg?w=829)

Running CTest on the Command Line

To run a single test or a subset of tests, I can use the -R option:

```
ctest --test-dir build/debug-test -R Led_2
```

## VS Code

So far, everything was with CMake, CTest and command line tools. But CTest is nicely integrated in VS Code.

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/ctest-in-vs-code.jpg?w=1024)

CTest with VS Code

## Summary

With this, I have a complete setup and system to run on-target tests on the real hardware: from using VS Code, running CMake and CTest, configuring tests, downloading and running it on the target hardware, up to collecting test results. Clearly one does not need to test everything on the hardware, but in embedded systems there are always things which only can be tested on hardware. I hope with the presented approach and with the project and files shared on GitHub you can explore unit or system testing on hardware in a modern environment.

Happy testing 🙂

### Links

- Project on GitHub: [https://github.com/ErichStyger/MCUXpresso\_LPC55S16\_CI\_CD](https://github.com/ErichStyger/MCUXpresso_LPC55S16_CI_CD)
- Unity Test-Framework: [http://www.throwtheswitch.org/unity](http://www.throwtheswitch.org/unity)
- McuLib: [https://github.com/ErichStyger/McuOnEclipseLibrary](https://github.com/ErichStyger/McuOnEclipseLibrary)
- [CI/CD for Embedded with VS Code, Docker and GitHub Actions](https://mcuoneclipse.com/2023/10/02/ci-cd-for-embedded-with-vs-code-docker-and-github-actions/)
- [LoRaWAN with NXP LPC55S16 and ARM Cortex-M33](https://mcuoneclipse.com/2021/12/30/lorawan-with-nxp-lpc55s16-and-arm-cortex-m33/)
- TDD: [https://en.wikipedia.org/wiki/Test-driven\_development](https://en.wikipedia.org/wiki/Test-driven_development)
- CI/CD: [https://en.wikipedia.org/wiki/CI/CD](https://en.wikipedia.org/wiki/CI/CD)
- CMake: [https://cmake.org/](https://cmake.org/)
- CTest: [https://cmake.org/cmake/help/latest/manual/ctest.1.html](https://cmake.org/cmake/help/latest/manual/ctest.1.html)
- NXP LPC55S16-EVK: [https://www.nxp.com/design/design-center/software/development-software/mcuxpresso-software-and-tools-/lpcxpresso-boards/lpcxpresso55s16-development-board:LPC55S16-EVK](https://www.nxp.com/design/design-center/software/development-software/mcuxpresso-software-and-tools-/lpcxpresso-boards/lpcxpresso55s16-development-board:LPC55S16-EVK)
- SEGGER J-Run: [https://www.segger.com/products/debug-probes/j-link/tools/j-run/](https://www.segger.com/products/debug-probes/j-link/tools/j-run/)
- J-Link Script Files: [https://wiki.segger.com/J-Link\_script\_files](https://wiki.segger.com/J-Link_script_files)
- How not to initialize variables in startup code: [https://mcuoneclipse.com/2014/04/19/gnu-linker-can-you-not-initialize-my-variable/](https://mcuoneclipse.com/2014/04/19/gnu-linker-can-you-not-initialize-my-variable/)