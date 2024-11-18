#MCU #C #NXP

In many of my embedded projects I need persistent data or storage for settings. If using an SD card, then [FatFS](https://mcuoneclipse.com/2021/05/15/using-fatfs-and-minini-with-the-nxp-lpc55s16-evk/) is usually my choice for the file system. But if using an external FLASH memory device, then my preferred choice is usually [LittleFS](https://github.com/littlefs-project/littlefs): it is a little fail-safe filesystem, designed for micro-controllers, which I’m using with external flash memory devices.

In the case where there is enough MCU flash, or if there is no external FLASH device available in a design, it can use the MCU internal FLASH as storage storage too. This is the topic of this article:

![](https://mcuoneclipse.com/wp-content/uploads/2023/07/littlefs-file-system-data-in-internal-flash-memory.png?w=822)

LittleFS File System Data

## Outline

In this project I’ll show you how you can get quickly up and running using LittleFS with FreeRTOS, using the internal flash memory of the MCU. The project used in this article is running with FreeRTOS on the [NXP LPC55S16-EVK](https://mcuoneclipse.com/2020/05/01/nxp-lpc55s16-evk-unboxing-and-first-impressions/) board.

![](https://mcuoneclipse.com/wp-content/uploads/2023/01/lpc55s16-evk.jpg?w=965)

NXP LPC55S16-EVK Board

## Project

First, create a project using the IDE for the device used (in this case, the LPC55S16). You can follow the steps below and/or use the project on [GitHub](https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/MCUXpresso/LPC55S16-EVK/LPC55S16_Blinky) (LPC55S16\_Blinky).

## McuLib

Add the [McuLib](https://github.com/ErichStyger/McuOnEclipseLibrary) to the project. This comes with FreeRTOS plus the littleFS file system. Alternatively you can use the littleFS and FreeRTOS from the NXP SDK, or directly from [freeRTOS.org](https://www.freertos.org/) and [LittleFS Github](https://github.com/littlefs-project/littlefs).

![](https://mcuoneclipse.com/wp-content/uploads/2023/07/blinky-with-mculib.png?w=328)

## Configuration

Next, the LittleFS middleware needs to be configured. I’m usually doing this inside header file (**source/IncludeMcuLibConfig.h**).

First, we need to configure the flash memory: we enable the module, specify the number of flash blocks to be used. The LPC55S16 has 244 KByte of flash, and we allocate the blocks at the end of the memory:

```
#define McuFlash_CONFIG_IS_ENABLED                    (1) /* enable McuFlash module */
#define McuFlash_CONFIG_NOF_BLOCKS                    (32) /* number of flash blocks */
#define McuFlash_CONFIG_MEM_START                     (((0+244*1024)-((McuFlash_CONFIG_NOF_BLOCKS)*McuFlash_CONFIG_FLASH_BLOCK_SIZE)))
```

Below is how to enable the file system:

```
#define LITTLEFS_CONFIG_ENABLED  (1) /* enable the LittleFS file system */
```

Next, we set the system to use a FLASH based file system

```
#define McuLittleFSBlockDevice_CONFIG_MEMORY_TYPE McuLittleFSBlockDevice_CONFIG_MEMORY_TYPE_MCU_FLASH
```

Finally, we have to configure the block size, block count and the block offset. The offset is used as start address inside the memory:

```
#define McuLittleFS_CONFIG_BLOCK_SIZE                 (McuFlash_CONFIG_FLASH_BLOCK_SIZE)
#define McuLittleFS_CONFIG_BLOCK_COUNT                (McuFlash_CONFIG_NOF_BLOCKS)
#define McuLittleFS_CONFIG_BLOCK_OFFSET             ((McuFlash_CONFIG_MEM_START)/(McuFlash_CONFIG_FLASH_BLOCK_SIZE))
```

## Initialization

To use the file system, I have to initialize it. So I have to call the ‘init’ both for the flash module and the file system module:

```
McuFlash_Init(); /* initialize flash module */
McuFlash_RegisterMemory((const void*)McuFlash_CONFIG_MEM_START, McuFlash_CONFIG_NOF_BLOCKS*McuFlash_CONFIG_FLASH_BLOCK_SIZE);
McuLFS_Init(); /* initialize LittleFS */
```

## Mounting the file system

To use the file system, I have to mount it:

```
McuLFS_Mount(McuShell_GetStdio());
```

Mounting the file system is done usually inside a task.

After that, the LittleFS file system can be used for reading and writing.

## Example with Command Line Shell

Everything can be used from a command line [shell](https://mcuoneclipse.com/2016/02/06/tutorial-bare-metal-shell-for-kinetis/). For this, connect with a terminal program to the board (UART or USB CDC/VCOM connection). The ‘help’ command shows the availble commands. Most important for our use here are are the ‘McuLittleFS’ and ‘McuFlash’ sections:

![](https://mcuoneclipse.com/wp-content/uploads/2023/07/shell-interface.png?w=876)

McuShell help output

As the LPC55Sxx use a special type of flash memory, the commands to initialize and erase the memory can be very useful during development. So for example if the flash is only in the initialized state, you have to erase it first:

```
McuFlash erase 0x00039000 0x4000
```

The ‘status’ commands can be used to show the current settings and status:

![](https://mcuoneclipse.com/wp-content/uploads/2023/07/status-commands.png?w=876)

Shell status command

In the case above, the file system has not been mounted yet, and the memory has not been formatted yet. Use the following:

```
McuLittleFS format
```

To mount it, I can use

```
McuLittleFS mount
```

After that, the file system can be used to write and read files, for example:

```
McuLittleFS ls
McuLittleFS bincat test.txt 0x1 0x2 0x3 0x4 0x5
McuLittleFS printhex test.txt
```
![](https://mcuoneclipse.com/wp-content/uploads/2023/07/littlefs-example-session.png?w=670)

LittleFS example session

## Summary

An embedded file system to store and use data on the device is very useful. In many applications I’m using the LittleFS file system: it is optimized for embedded systems, works nicely with FreeRTOS and can be easily used with on-chip and off-chip flash devices. I hope with the provided example and instructions you can easily use it in your projects.

Do not need a file system to store settings? Then have a look at [MinINI](https://mcuoneclipse.com/2021/12/19/key-value-pairs-in-flash-memory-file-system-less-minini/) which can be used file-system-less.

Happy Flashing 🙂

### Links

- LittleFS File System: [https://github.com/littlefs-project/littlefs](https://github.com/littlefs-project/littlefs)
- Project on GitHub: [https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/MCUXpresso/LPC55S16-EVK/LPC55S16\_Blinky](https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/MCUXpresso/LPC55S16-EVK/LPC55S16_Blinky)
- McuLib on GitHub: [https://github.com/ErichStyger/McuOnEclipseLibrary](https://github.com/ErichStyger/McuOnEclipseLibrary)
- FatFS with the LPC55S16: [Using FatFS and MinINI with the NXP LPC55S16 EVK](https://mcuoneclipse.com/2021/05/15/using-fatfs-and-minini-with-the-nxp-lpc55s16-evk/)