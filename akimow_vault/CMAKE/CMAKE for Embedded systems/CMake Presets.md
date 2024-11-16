## Introduction

#CMAKE #MCU

When we developed the CMake based toolchain for our training projects  we used a shell script to simplify invoking the **cmake** command line. CMake 3.19 added a [presets](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html) feature that allows us to define command line parameters in a **CMakeSettings.json** file which can be used in place of using multiple command parameters.

In previous articles about CMake we have shown how we need to specify  command line parameters to use CMake with an embedded target  toolchain (see [CMake Part 3](https://feabhasblog.wpengine.com/2021/08/cmake-part-3-source-file-organisation/)). To generate the build configuration files we use a command line with several options:

```
$ cmake -S . -B build/debug --warn-uninitialized \
        -DCMAKE_BUILD_TYPE=DEBUG \
        -DCMAKE_TOOLCHAIN_FILE=toolchain-STM32F407.cmake
```

Each time we actually want to build the system we have to enter a second command line with multiple options:

```
$ cmake --build build/debug --config Debug -- --no-print-directory
```

These commands are long and complex so we wrote a [Bash script](https://help.ubuntu.com/community/Beginners/BashScripting) to simplify invoking these commands.

With presets available we can now use a much shorter command line by defining a preset configuration so that the following generates the **debug** build configuration:

```
$ cmake --preset debug
```

And the following performs the build itself:

```
$ cmake --build --preset debug
```

If we want a release build we just define a **release** preset and invoke that from the command line in the same manner.

## Presets definition file

The capabilities of the presets have developed over time from supporting configuration and build stages to include testing (the **ctest** command)  and other CMake features. This blog will concentrate on using presets for the configuration and build stages only. An example project with presets can be downloaded from the Feabhas GitHub repo [CMake Presets Blog.](https://github.com/feabhas/cmake-presets-blog)

Presets are defined in a **CMakePresets.json** file stored alongside the **CMakelists.txt** file and consists of the following *outline* [JSON](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON) components:

```
{
  "version": 3,
  "cmakeMinimumRequired": {
   "major": 3,
   "minor": 21,
   "patch": 0
  },
  "configurePresets": [
  ],
  "buildPresets": [
  ],
}
```

The version number determines which level of preset support is required and version 3 is the minimum required to support *configure* and *build* presets.

After the version requirements there are optional arrays of definitions for each category of preset. It is these entries that define the parameters that previously had to be supplied on the command line.

Each preset has a name that is unique within the preset category so the *debug* preset in the *configuration* section is different to the *debug* preset in the *build* section.The presets are hierarchical allowing one preset to inherit definitions from another so that common settings can be reused.

## Defining configure presets

We’ll start with a preset for the embedded toolchain used with our STM32F407 target board:

```
"configurePresets": [
  {
    "name": "stm32-base",
    "hidden": true,
    "generator": "Unix Makefiles",
    "binaryDir": "${sourceDir}/build/${presetName}",
    "cacheVariables": {
      "CMAKE_BUILD_TYPE": "Debug",
      "CMAKE_TOOLCHAIN_FILE": {
        "type": "FILEPATH",
        "value": "toolchain-STM32F407.cmake"
      },
      "EXCEPTIONS": "OFF"
    },
    "warnings": {
      "uninitialized": true,
      "dev": true,
      "deprecated": true
    },
    "architecture": {
      "value": "unknown",
      "strategy": "external"
    }
  }
]
```

This base configuration is not meant to be used directly buts acts like an [Abstract Base Class](https://en.wikipedia.org/wiki/Class_\(computer_programming\)#Abstract_and_concrete) used in [Object Oriented Programming](https://en.wikipedia.org/wiki/Object-oriented_programming). Setting the **hidden** attribute to true will prevent it from appearing in the list of configure presets obtained using the command:

```
$ cmake --list-presets
```

A similar command is used to list the *build* presets:

```
$ cmake --build --list-presets
```

The **stm32-base** preset defines the common configuration parameters for the **debug** and **release** builds. The entries should be self explanatory: they define the build system to use (**generator**), the build directory (**binaryDir**) and variables to add to the cache (**cacheVariables**). The **CMakePresets.json** file is described on the [Presets manual page.](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html)

Presets have their own variables and use a different naming conventions to avoid confusion with other CMake variables so that **${sourceDir}** defines the workspace root in a preset and and has the same value as the CMake variable **${CMAKE\_SOURCE\_DIR}**.

In the **cacheVariables** attribute the CMAKE\_TOOLCHAIN\_FILE entry defines the toolchain file we introduced in the introductory [CMake blog article](https://feabhasblog.wpengine.com/2021/07/cmake-part-1-the-dark-arts/).

At the end of each preset we can add an optional section for vendor specific entries such as this one to support [IntelliSense](https://learn.microsoft.com/en-us/visualstudio/ide/using-intellisense?view=vs-2022) used by Microsoft tools:

```
"vendor": {
  "microsoft.com/VisualStudioSettings/CMake/1.0": {
  "intelliSenseMode": "linux-gcc-arm"
}
```

The **debug** configuration preset used on the command simply inherits from the **stm32-base** preset:

```
"configurePresets": [   
  {
    ...
    {
      "name": "debug",
      "displayName": "Debug",
       "inherits": "stm32-base"
    }
  }
]
```

Similarly a **release** preset inherits from **stm32-base** but we override the CMAKE\_BUILD\_TYPE value:

```
"configurePresets": [   
  {
    ...
    {
      "name": "release",
      "displayName": "Release",
      "inherits": "stm32-base",
      "environment": {
        "CMAKE_BUILD_TYPE": "Release"
     }
  }
]
```

Note that any variable defined in the **environment** section will override its value in the **cacheVariables** section.

Generating the project build files no longer requires any **cmake** command line parameters:

```
$ cmake --preset debug
```

## Defining build presets

The presets for using CMake to build the project follow the same idiom by starting with an abstract base definition:

```
"buildPresets": [
  {
    "name": "build-base",
    "hidden": true,
    "configurePreset": "debug",
    "nativeToolOptions": [
      "--no-print-directory"
    ]
  }
 ]
```

The *build* preset needs to know the target build directory location and the build type used in the configuration so the **configurePreset** entry is used to ensure we pickup the same values as those used in the named configure preset.

The **debug** build preset used on the command line just inherits from this **build-base** definition:

```
"buildPresets": [   
  {
  ...
{
      "name": "debug",
      "inherits": "build-base"
    }
```

We can now build the system using:

```
$ cmake --build --preset debug
```

The **release** build uses the same **build-base** but overrides the **configurePreset** to use the **release** configuration:

```
    {
      "name": "release",
      "inherits": "build-base",
      "configurePreset": "release"
    }
```

We can also provide additional presets to invoke custom build tasks. Our **CMakeLists.txt** configuration has one target to run [clang-tidy](https://clang.llvm.org/extra/clang-tidy/) and a second to run any test cases we have supplied in the project. Presets for these tasks are:

```
{
  "name": "clang-tidy",
  "inherits": "debug",
  "targets": [ "clang-tidy" ]
},
{
  "name": "test",
  "inherits": "debug",
  "targets": [ "test" ],
  "nativeToolOptions": [
    "ARGS=--output-on-failure"
  ]
}
```

The **test** preset also shows how we can pass target options to the underlying command. We can invoke these presets using:

```
$ cmake --build --preset clang-tidy
$ cmake --build --preset test
```

## Conclusion

The addition of presets to CMake to define *configure* and *build* variables and other options in a JSON configuration file has dramatically simplified invoking the **cmake** command. Prior to being able to use presets  the complex CMake command lines typically required supporting Bash or PowerShell scripts to be used to avoid typing long and complex command lines.

With presets the **cmake** command line is noticeably simpler and can be entered directly without the need for a supporting script.

- [About](https://blog.feabhas.com/2023/08/cmake-presets/#abh_about)
- [Latest Posts](https://blog.feabhas.com/2023/08/cmake-presets/#abh_posts)

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