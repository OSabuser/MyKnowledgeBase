
#MCU #C
The RP2040 Pico board comes with 2 MByte onboard FLASH memory. While this is plenty of space for many embedded applications, sometimes it needed to have more storage space. Having the ability to adding an extra SPI FLASH memory with a useful file system comes in handy in such situations. This makes the RP2040 ideal for data logger applications or otherwise store a large amount of data. In this article I’ll show you how to add an extra 16 MByte of memory to the Raspberry Pi Pico board, running FreeRTOS, a command line shell and using LittleFS as the file system.

![](https://mcuoneclipse.com/wp-content/uploads/2022/12/raspberry-pi-pico-with-winbond-external-flash.png?w=829)

## Outline

I have used the [LittleFS](https://github.com/littlefs-pr) in other projects with the [McuLib](https://github.com/ErichStyger/McuOnEclipseLibrary). For a data logger project I have ported the file system using a Winbond W25Q128 16 MByte SPI FLASH device for the Raspberry Pi Pico RP2040. You can find an example project using it on [Github](https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/RaspberryPiPico/pico_LittleFS_W25Q128).

## Files

The CMake file in the example project pulls in the McuLib files from GitHub: just run the ‘cmake init.bat’ file. It creates as well the needed files to use it in an IDE like Eclipse:

![](https://mcuoneclipse.com/wp-content/uploads/2022/12/littlefs-inside-mculib.png?w=417)

## Build

You can use tone of the make targets in the IDE:

![](https://mcuoneclipse.com/wp-content/uploads/2022/12/make-targets-in-ide.png?w=393)

If desired, the project can be built on the console:

![](https://mcuoneclipse.com/wp-content/uploads/2022/12/build-with-make.png?w=766)

The memory device and file system is configured through a set of configuration macros:

![](https://mcuoneclipse.com/wp-content/uploads/2022/12/file-system-configuraiton.png?w=1024)

## Init

During startup, the needed modules (McuSPI, McuW25 and McuLFS) for the file system get initialized:

![](https://mcuoneclipse.com/wp-content/uploads/2022/12/initialization.png?w=865)

## SPI Pins

Wiring and pins can be configured for the McuSPI module. By default it uses SPI1 with the25 following pins:

```
#define MCUSPI_CONFIG_HW_SCLK_PIN (10)  /* SPI1_SCK */
#define MCUSPI_CONFIG_HW_MOSI_PIN (11)  /* SPI1_TX  */
#define MCUSPI_CONFIG_HW_MISO_PIN (12)  /* SPI1_RX  */
#define MCUSPI_CONFIG_HW_CS_PIN   (13)  /* SPI1_CSn */
```

If other SPI pins are needed, they can be configured in the McuSPIconfig.h file.

## Shell

The modules have implemented a shell command line interface, below the help text:

![](https://mcuoneclipse.com/wp-content/uploads/2022/12/shell-help-output.png?w=813)

The ‘status’ shell command reports both the status of the file system and of the flash memory:

![](https://mcuoneclipse.com/wp-content/uploads/2022/12/status-output.png?w=511)

## Debug

The project includes both Segger J-Link and OpenOCD ([PicoProbe](https://mcuoneclipse.com/2022/09/17/picoprobe-using-the-raspberry-pi-pico-as-debug-probe/)) debug configurations. As you can see, the application runs FreeRTOS on the the RP2040, which makes it ideal for any kind of data logger.

![](https://mcuoneclipse.com/wp-content/uploads/2022/12/debug.png?w=1024)

## Summary

It is always good to have the ability to use an external SPI Flash device. The the Raspberry Pi Pico board already has an onboard 2 MByte Flash, and having an external one with the LittleFS file system is certainly a plus. With the Winbond and LittleFS integration in the McuLib it is now very easy to add some extra storage for every Raspberry Pi Pico project.

To use the example project on GitHub, follow the tutorial I wrote earlier: [Getting Started: Raspberry Pi Pico RP2040 with Eclipse and J-Link](https://mcuoneclipse.com/2022/07/16/getting-started-raspberry-pi-pico-rp2040-with-eclipse-and-j-link/). It both works with Cmake and Eclipse IDE support.

Happy Flashing 🙂

Links

- [Driver and Command Line Shell for Winbond W25Q128 16MByte Serial FLASH Device](https://mcuoneclipse.com/2019/01/06/driver-and-shell-for-winbond-w25q128-16mbyte-serial-flash-device/)
- LittleFS: [https://github.com/littlefs-project/littlefs](https://github.com/littlefs-project/littlefs)
- McuLib: [https://github.com/ErichStyger/McuOnEclipseLibrary](https://github.com/ErichStyger/McuOnEclipseLibrary)
- Example on GitHub: [https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/RaspberryPiPico/pico\_LittleFS\_W25Q128](https://github.com/ErichStyger/mcuoneclipse/tree/master/Examples/RaspberryPiPico/pico_LittleFS_W25Q128)
- Setting up project with Eclipse and CMake: [Getting Started: Raspberry Pi Pico RP2040 with Eclipse and J-Link](https://mcuoneclipse.com/2022/07/16/getting-started-raspberry-pi-pico-rp2040-with-eclipse-and-j-link/)
- [Different Ways of Software Configuration](https://mcuoneclipse.com/2019/02/23/different-ways-of-software-configuration/)