#MCU #C
Most of the time software needs some way to configure things: depending on the settings, the software will do different things. For example the software running on the microcontroller on top of the Raspberry might have the OLED LCD available or not:

[![Raspberry Pi and tinK22 with OLED LCD](https://mcuoneclipse.com/wp-content/uploads/2019/02/raspberry-pi-and-tink22-with-oled-lcd.png?w=584&h=439)](https://mcuoneclipse.com/wp-content/uploads/2019/02/raspberry-pi-and-tink22-with-oled-lcd.png)

Raspberry Pi and tinyK22 (NXP Kinetis K22FN512) with OLED LCD

How can I deal with this in my application code?

## Variable

For example I could check in my application a variable if there is a certain feature available at runtime:

| 1  2  3  4  5 | `extern` `bool` `configHas_LCD = ``true``;`  `...`  `if` `(configHas_LCD) {`  `showStatusOnLCD();`  `}` |
| --- | --- |

That approach would decide this at runtime. It needs more memory and takes longer to execute.

## File

Having a file system available, it can make sense to store the configuration in a file. There is the very useful  .ini utility (see [Minini](https://mcuoneclipse.com/2014/04/26/frdm-with-arduino-ethernet-shield-r3-part-4-minini/)). With this configuration settings can be stored in \[sections\] in a text file:

```
[Configuration]
LCD = Yes
```

Then I can use it like this:

| 1  2  3  4  5 | `configHas_LCD = MINI1_ini_bool(``"Configuration"``, ``"LCD"``, ``false``, ``"config.ini"``);`  `if` `(configHas_LCD) {`  `showStatusOnLCD();`  `}`  `}` |
| --- | --- |

With that approach I can configure the application from the ‘outside’: very flexible, but of course comes with some overhead.

## Constant

A better approach would be to use a constant if that configuration does not change:

| 1  2  3  4  5 | `const` `bool` `configHas_LCD = ``true``; `  `...`  `if` `(configHas_LCD) {`  `showStatusOnLCD();`  `}` |
| --- | --- |

Better, and hopefully the compiler will do ‘constant-folding’ and no extra compare code will be present in my application.

## #define

What I prefer is to use some configuration macro defines:

| 1  2  3  4  5  6  7 | `#define CONFIG_HAS_LCD   (1)`  `...`  `#if CONFIG_HAS_LCD`  `showStatusOnLCD();`  `#endif` |
| --- | --- |

It makes sense to have these configuration macros in a dedicated header file:

| 1  2  3 | `#define CONFIG_HAS_LCD (1) ` |
| --- | --- |

and then include that header file in the application.

## Compiler -D Option

Another way is to use the compiler -D (Define) option.

```
-DCONFIG_HAS_LCD=1
```

has the same effect as having

in the sources.

In Eclipse (screenshot from [MCUXpresso IDE](https://mcuoneclipse.com/2018/12/16/new-nxp-mcuxpresseo-ide-v10-3-0-release/)) the defines can be added in the project settings:

[![Compiler -D Option](https://mcuoneclipse.com/wp-content/uploads/2019/02/compiler-d-option.png?w=584&h=541)](https://mcuoneclipse.com/wp-content/uploads/2019/02/compiler-d-option.png)

Compiler -D Option

I don’t prefer that way because that way the settings are ‘buried’ in the project settings. With using a version control system, it might be hard to see changes that way.

## \-include Compiler Option

What I prefer is to use the -include compiler option: using that option I can include a header file for each source file I compile.

That option is described in [https://gcc.gnu.org/onlinedocs/gcc-4.3.2/gcc/Preprocessor-Options.html#Preprocessor-Options](https://gcc.gnu.org/onlinedocs/gcc-4.3.2/gcc/Preprocessor-Options.html#Preprocessor-Options):

`-include` file:  
process file as if `#include "file"` appeared as the first line of the primary source file. However, the first directory searched for file is the preprocessor’s working directory *instead of* the directory containing the main source file. If not found there, it is searched for in the remainder of the `#include "..."` search chain as normal.

I can use that option in make files or set it in the IDE:

[![-Include Option](https://mcuoneclipse.com/wp-content/uploads/2019/02/include-option.png?w=584&h=576)](https://mcuoneclipse.com/wp-content/uploads/2019/02/include-option.png)

\-Include Option

Please note that I’m using a fully qualified path to the header file:

```
"${ProjDirPath}/source/config.h"
```

Just using ‘config.h’ would be fine for the compiler. But there is a long outstanding bug in Eclipse CDT how the [Eclipse Indexer](https://mcuoneclipse.com/2012/03/20/fixing-the-eclipse-index/) is dealing with the -include option. Without an expanded path, Eclipse CDT shows annoying warnings about ‘unresolved inclusion’:

[![Unresolved Inclusion](https://mcuoneclipse.com/wp-content/uploads/2019/02/unresolved-inclusion.png?w=584)](https://mcuoneclipse.com/wp-content/uploads/2019/02/unresolved-inclusion.png)

Unresolved Inclusion

> 💡 There is yet another glitch in Eclipse CDT: the option description suggests that there should be a space between -include and the file name, while Eclipse CDT does not issue a space. The gcc compilers accepts the option without a space too, at least in the current version.

## Multi-Level Configuration

Of course there are many ways to implement configurations. Here is my preferred way:

Each driver or module has an external configuration file. For example **LCD.c** and **LCD.h**, and the configuration of the LCD is in **configLCD.h**

| 1  2  3  4  5  6  7  8  9 | `#ifndef CONFIGLCD_H_`  `#define CONFIGLCD_H_`  `#ifndef CONFIG_HAS_LCD`  `#define CONFIG_HAS_LCD (0)`  `#endif`  `#endif` |
| --- | --- |

I can configure the driver with the settings in that configuration header file if I need to.This configuration header file gets included in the LCD driver:

| 1  2  3  4  5  6  7  8 | `#include "configLCD.h`  `#include "LCD.h"`  `...`  `#if CONFIG_HAS_LCD`  `showStatusOnLCD();`  `#endif`  `...` |
| --- | --- |

Note that the configuration #defines in the configuration header file are setting a default value in case the macro is \*not\* already defined (#ifndef). So if don’t want to touch the file I can use the -D compiler option to change the setting.

Or better: use the -include to include a header file like below:

| 1  2 | `#define CONFIG_HAS_LCD (1) /* enable LCD support */` |
| --- | --- |

With this I can overwrite or configure a driver without touching the driver configuration header file itself. This is especially useful if I have many drivers or have them shared by multiple projects: the drivers can be shared together with their default configuration files, while I set and overwrite things with the -include file.

## Summary

There are many ways to configure software at compile or runtime. I prefer using #define macros in combination with the -include option for most of my applications.

I hope this is useful for you and you might use one or the other way to make your software and drivers more versatile.

Happy configuring 🙂

## Links

- gcc include option: [https://gcc.gnu.org/onlinedocs/gcc-4.3.2/gcc/Preprocessor-Options.html#Preprocessor-Options](https://gcc.gnu.org/onlinedocs/gcc-4.3.2/gcc/Preprocessor-Options.html#Preprocessor-Options)
- MCUXpresso IDE: [http://www.nxp.com/mcuxpresso/ide](http://www.nxp.com/mcuxpresso/ide)
- MinINI: [FRDM with Arduino Ethernet Shield R3, Part 4: MinIni](https://mcuoneclipse.com/2014/04/26/frdm-with-arduino-ethernet-shield-r3-part-4-minini/)