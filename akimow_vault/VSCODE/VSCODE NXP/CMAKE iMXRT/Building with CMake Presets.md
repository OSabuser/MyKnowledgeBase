#CMAKE #MCU

I’m getting my head more and more around CMake and its features. After having so [many issues with VS Code dealing with CMake Kits](https://mcuoneclipse.com/2023/11/05/no-kit-selected-fixing-vs-code-cmake-kit-assignment/), I have found feature in CMake which really is a game changer for me: **CMake Presets**.

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/working-with-cmake-presets-in-vs-code.jpg?w=1024)

Working with CMake Presets in VS Code

## Outline

In a nutshell, CMake is an open source ‘cross-platform build generator’: Using CMake files, I can orchestrate my build process. CMake is a generator, so it generates build files for builders like for GNU [make](https://www.gnu.org/software/make/) or [ninja](https://en.wikipedia.org/wiki/Ninja_\(build_system\)). The downside is that it takes time to learn CMake and its imperative programming language. In contrast, on Eclipse the ‘managed make’ build process is very easy to use, but less flexible. VS Code does not come with a build system, so I’m using and learning CMake with it.

Recently I had a lot of issues with CMake in VS Code, and I have found a way to make the build process both independent of VS Code, and the same time much better integrated. And here **CMake Presets** were a true game changer, as now I can easily have different build types like ‘**debug**‘, ‘**release**‘ or ‘**coverage**‘.

All this could be done with rather complex command lines running CMake, e.g.

```
cmake . -B build/debug -DCMAKE_BUILD_TYPE=DEBUG     -DCMAKE_TOOLCHAIN_FILE=arm-none-eabi-gcc.cmake
```

but this is hard to remember, or I need to create script/batch files, which makes things really complex with different configurations. CMake Presets replaces all this, with a set o JSON files defining the options and more.

In the next sections I explain how I’m using CMake Presets to make building more flexible and easier. There might be better ways, but at least this is currently working very well for me, and enables me to have an easy and simple [CI/CD pipeline](https://mcuoneclipse.com/2023/10/02/ci-cd-for-embedded-with-vs-code-docker-and-github-actions/) too.

You can find an example how I’m using CMake Presets with VS Code on [GitHub](https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/VisualStudioCode/LPC55S16-EVK/LPC55S16_Blinky).

## CMake Presets Definition File

CMake Presets contain information how to configure, build, test and even package a project. Preset information is stored in a JSON files:

```
{  "version": 6,  "cmakeMinimumRequired": {    "major": 3,    "minor": 23,    "patch": 0  },  "configurePresets": [    {      // ...    }  ],  "buildPresets": [    {      // ...    }  ],  "testPresets": [    {      // ...    }  ]}
```

With the **configurePresets** I specify how the project is configured (paths, architecture, options, …). They are used if I do a `CMake: Configure` in VS Code.

The **buildPresets** describe what build types I have (e.g. debug, release, …). And finally, the **testPresets** define the builds to be used for testing. They are used if I do a `CMake: Build` in VS Code.

The default file is named **CMakePresets.json** and is located in the root folder of the project:

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/cmakepreset.json-in-vs-code.jpg?w=970)

CMakePresets.json in VS Code

It is possible to use a `CMakeUserPresets.json`: this one is meant to specify own local settings and build details for a developer.

## Configure Preset

Below is an example of a preset:

```
"configurePresets": [    {      "name": "config-base",      "hidden": true,      "displayName": "base Configuration",      "description": "Default build using Ninja generator",      "generator": "Ninja",      "binaryDir": "${sourceDir}/build/${presetName}",      "toolchainFile": "${sourceDir}/arm-none-eabi-gcc.cmake"    },    {        "name": "debug",        "displayName": "Config Debug",        "description": "Sets debug build type and cortex arch",        "inherits": "config-base",        "architecture": {            "value": "arm",            "strategy": "external"        },        "cacheVariables": {            "CMAKE_BUILD_TYPE": "Debug"        }      },      {        "name": "release",        "displayName": "Config Release",        "description": "Sets release build type",        "inherits": "debug",        "cacheVariables": {            "CMAKE_BUILD_TYPE": "Release"        }      }  ]
```

The first entry describes the ‘config-base’, which is hidden and is **inherited** in the ‘debug’ and ‘release’ configuration. The inheritance works nicely here, as I can define a some base properties, which then can be used and possibly overwritten in the debug and release configuration.

Notice that with the ‘`toolchainFile`‘ the CMake kit to be used is specified. Using ‘`cacheVariables`‘ I can set variables and settings used later in the `CMakeLists.txt` files.

## Build Presets

In a similar way, the build presets are built up here for debug and release builds:

```
"buildPresets": [    {        "name": "build-base",        "hidden": true,        "configurePreset": "debug",    },    {      "name": "debug",      "displayName": "Build Debug",      "inherits": "build-base"    },    {      "name": "release",      "displayName": "Build Release",      "inherits": "build-base",      "configurePreset": "release"    }  ]
```

Here again I define a (hidden) build-base, and inherit from it for a release and debug build. The build presets point back to a configure preset.

## CMake Command-line Usage

To list the available presets with CMake, I use the following:

```
cmake --list-presets
```

which gives:

```
Available configure presets:  "debug"   - Config Debug  "release" - Config Release
```

To configure a project with CMake using a preset, I use the following:

```
cmake --preset <name-of-preset>
```

for example:

```
> cmake --preset debugPreset CMake variables:  CMAKE_BUILD_TYPE="Debug"  CMAKE_TOOLCHAIN_FILE:FILEPATH="LPC55S16-EVK/LPC55S16_Blinky/arm-none-eabi-gcc.cmake"-- McuLib for MCUXpresso SDK-- Configuring done (0.3s)-- Generating done (0.1s)-- Build files have been written to: LPC55S16-EVK/LPC55S16_Blinky/build/debug
```

In a similar way, I can list the build presets:

```
cmake --build --list-presets
```

To build a project, I use

```
cmake --build --preset <name-of-preset>
```

For example:

```
> cmake --build --preset release[149/149] Linking C executable LPC55S16_Blinky.elfMemory region         Used Size  Region Size  %age Used   PROGRAM_FLASH:       27740 B     228608 B     12.13%            SRAM:       42940 B        64 KB     65.52%         USB_RAM:          0 GB        16 KB      0.00%           SRAMX:          0 GB        16 KB      0.00%   text    data     bss     dec     hex filename         27724      16   42924   70664   11408 LPC55S16-EVK/LPC55S16_Blinky/build/Release/LPC55S16_Blinky.elf
```

With this, I have the ability to build the project. I don’t need an IDE or editor for this, everything is easily managed by command line commands. That way, it it can be integrated into a CI/CD pipeline, and the CMake kits and options are all defined with the Preset, the referenced CMake Kit and the `CMakeLists.txt`.

## CMake Presets with VS Code

While we can build everything from the command line, [CMake Presets are supported in VS Code](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/cmake-presets.md).

To tell VS Code to use CMake Presets, use the following two settings in **settings.json**:

```
"cmake.useCMakePresets": "always","cmake.options.statusBarVisibility": "visible",
```
![](https://mcuoneclipse.com/wp-content/uploads/2023/12/cmake-presets-option-in-vs-code.jpg?w=970)

settings.json for CMake Presets

- **useCMakePresets** tells VS Code to use presets with CMake.
- **statusBarVisibility** makes sure that all the preset options are shown in the bottom status bar.

## VS Code Status Bar

The status bar in VS Code shows the currently selected Presets, and I can click on them to change the preset e.g. from `Release `to `Debug`:

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/vs-code-cmake-status-bar-preset.jpg?w=1024)

VS Code CMake Preset Status Bar

> 💡 In above project I do not have a Test Preset defined: this is a subject of a future article describing how to set up tests and run it with CTest.

With that, I can use the usual CMake commands in VS code to clean, configure and build:

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/vs-code-cmake-commands.jpg?w=621)

CMake Commands in VS Code

## CMake Project Status

Additionally, the CMake extensions shows the project status with all the settings:

![](https://mcuoneclipse.com/wp-content/uploads/2023/12/cmake-project-status.jpg?w=318)

CMake Project Status

I can directly edit the settings from that view, e.g. change the ‘Debug’ build to a ‘Release’ one.

## Summary

CMake Presets are probably one of the most powerful and useful features in CMake. Instead using complex command lines, I can structure my builds and options for it. The same time it nails down the CMake Kits which has been a constant source of frustration and problems with VS Code for me. With CMake Presets I can configure and build my projects on the command line. The same time CMake presets are much better supported and used with VS Code. My CMake and VS Code projects have been dramatically simplified with using Presets.

Most VS Code users or even many CMake users might not be aware of CMake Presets: I hope with this article I can help you to get to the next level of using CMake. I’m using Presets for my test builds too, more about this in future article.

Happy presetting 🙂

### Links

- Project on GitHub: [https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/VisualStudioCode/LPC55S16-EVK/LPC55S16\_Blinky](https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/VisualStudioCode/LPC55S16-EVK/LPC55S16_Blinky)
- [CI/CD for Embedded with VS Code, Docker and GitHub Actions](https://mcuoneclipse.com/2023/10/02/ci-cd-for-embedded-with-vs-code-docker-and-github-actions/)
- CMake Presets: [https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html)
- CMake Presets in VS Code: [https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/cmake-presets.md](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/cmake-presets.md)
- [“No Kit Selected”: Fixing VS Code CMake Kit Assignment](https://mcuoneclipse.com/2023/11/05/no-kit-selected-fixing-vs-code-cmake-kit-assignment/)