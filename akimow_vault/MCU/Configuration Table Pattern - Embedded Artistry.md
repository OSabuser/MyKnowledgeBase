---
title: "Configuration Table Pattern - Embedded Artistry"
source: "https://embeddedartistry.com/fieldmanual-terms/configuration-table-pattern/"
author:
  - "[[Embedded Artistry]]"
published: 2022-06-17
created: 2025-02-07
description: "Store configuration and initialization information inside of a table, and pass the table to an initialization routine that iterates over the table entries."
tags:
  - "clippings"
---
[17 June 2022](https://embeddedartistry.com/blog/2022/06/17/) • Last updated 9 February 2024

Store configuration and initialization information inside of a table, and pass the table to an initialization routine that iterates over the table entries.

## Problem

When attempting to create a portable and reusable software design, you need to [decouple](https://embeddedartistry.com/fieldmanual-terms/coupling/) your application code from the underlying platform – the [OS](https://embeddedartistry.com/fieldmanual-terms/operating-system/), the hardware, and any other non-portable constructs. [Abstraction](https://embeddedartistry.com/fieldmanual-terms/abstraction/) layers ([OSAL](https://embeddedartistry.com/fieldmanual-terms/operating-system-abstraction-layer/), [HAL](https://embeddedartistry.com/fieldmanual-terms/hardware-abstraction-layer/)) are typically used to create a decoupling point in the application. You can write drivers and modules that interact only with the provided abstractions, and changes below the abstractions will not require corresponding changes in the application layer.

One challenge in this scheme is handling initialization and configuration. Initialization requirements and available settings vary widely across devices, processors, and OSes. Attempting to create a general [abstraction](https://embeddedartistry.com/fieldmanual-terms/abstraction/) for these settings is a fool’s errand, as you will end up with too many potential options; only a limited set will apply to any given device, rendering the abstraction useless. Hiding this information within the driver or HAL implementation is also risky: each application will need different settings, meaning that you need to change the HAL for each application. How can you properly handle configuration and initialization while still benefiting from abstraction?

A separate problem from the one stated above: configuration and initialization information, such as thread settings and thread creation calls, are often scattered throughout an application. How can you group these settings together in a single, easy-to-find location?

## Solution

One way to address this problem is by defining and maintaining *configuration tables* for each of the various peripheral devices, OS types, and other non-portable constructs. All relevant information required for configuration and initialization is stored in these tables. They are specified at the application level, allowing each application (or the various configurations) to initialize the underlying hardware system according to its needs. Drivers and abstraction layers can remain reusable and do not need to be modified to support different configuration settings.

Specifying configurable settings in this way allows you to maintain a generic interface. There is no need to craft an initialization interface that supports all possible configuration settings. You can create a generic initialization function, such as `OS_thread_init(const OS_thread_config_t* config)`. While the exact definition of the configuration structure may change from one [RTOS](https://embeddedartistry.com/fieldmanual-terms/real-time-os/) to another, the initialization interface will remain the same, and the definitions for each RTOS will be consistent across different applications. This achieves a suitable middle ground for [designing for change](https://embeddedartistry.com/course/designing-embedded-systems-for-change/) while still supporting implementation-specific configuration options.

## Implementation

To implement this pattern, proceed through the following steps:

1. [Identify the Configuration Parameters](https://embeddedartistry.com/fieldmanual-terms/configuration-table-pattern/#Identify-the-Configuration-Parameters)
2. [Create the Configuration Structure and Supporting Types](https://embeddedartistry.com/fieldmanual-terms/configuration-table-pattern/#Create-the-Configuration-Structure-and-Supporting-Types)
3. [Populate the Table](https://embeddedartistry.com/fieldmanual-terms/configuration-table-pattern/#Populate-the-Table)
4. [Create the Initialization Routine](https://embeddedartistry.com/fieldmanual-terms/configuration-table-pattern/#Create-the-Initialization-Routine)

### Identify the Configuration Parameters

First, you need to identify the common configuration parameters that apply for your processor (family). This information can be found by reviewing the datasheet or the chip vendor drivers. Find what registers exist and what the various fields in the registers mean. Looking at different code examples can also be helpful for figuring out what typical configuration patterns are.

As you review this information, make a list of the various configurable parameters as well as possible values for that parameter..

For the purposes of abstraction, especially if supporting multiple processors in a given family, you should focus primarily on common settings that are available. However, configuration is something that typically varies from one processor to another, so do not be surprised if you need to use different configuration table definitions for different processor families.

For example, here is a common set of configurable parameters for a timer:

- Timer mode
- Enabled/disabled status
- Clock source
- Clock pre-scaler
- Count direction (up/down)
- Target count / interval / period
- Periodic / one-shot

Here is a common set of configurable parameters for [GPIO](https://embeddedartistry.com/fieldmanual-terms/general-purpose-i-o/):

- Direction (input/output/tri-state)
- [Pull-up](https://embeddedartistry.com/fieldmanual-terms/pull-up/)/down setting
- Drive options (normal, hi-drive)

You will also want to determine how the different parameters will be “mapped”. Some drivers will be mapped to a specific instance, such as `TIMER0` and `TIMER1`. Others, like [DMA](https://embeddedartistry.com/fieldmanual-terms/direct-memory-access/), may have a combination of a “device” (`DMA0`, `DMA1`) and “channel” (1..8). GPIO will commonly be mapped to a “port” (`GPIOA`, `GPIOB`) and “pin number”

### Create the Configuration Structure and Supporting Types

Once you have identified the various parameters and options, you are ready to create a structure that contains all the configurable settings.

Beningo offers the following timer configuration structure example in *[Reusable Firmware Development](https://amzn.to/3Hkpr9X)*:

While Beningo shows plain `uint32_t` values above, we prefer to use enumerations or vendor-supplied parameters for configuration table values, making them more readable and easier to reference. For Beningo’s GPIO configuration structure example, he uses this approach:

Example definitions for the enumerations referenced above are shown below.

If the chip vendor’s supplied definitions will work for table values, you can certainly use those. You can also use your custom enumeration values as the index in a look-up table kept in the driver implementation (or other file).

These enumeration and structure definitions are best placed within a separate header file instead of in the primary driver abstraction header. For example, if you have a `timer.h` header which provides generic timer functions, the configuration-related definition should go into `timer_config.h`. There are a few different reasons for this:

1. The base abstractions (e.g., `gpio_set_output`, `gpio_read`) should be usable on any processor and are unrelated to the device-specific configuration parameters (and, potentially, vendor-supplied definitions). Excluding timer configuration information from this header ensures that there is no accidental coupling to platform-specific details in code that would otherwise be portable.
2. Tightly coupled application code that sets up the configuration information for the system’s needs can intentionally include `timer_config.h`.
3. The driver implementation can include `timer_config.h` in order to access the full definition of the structure type.
4. Depending on how the driver is structured and how the various configurable parameter values are supplied, you may be able to maintain a common driver definition while supporting multiple processors by changing the configuration definitions. You can maintain different `timer_config.h` definitions for different processors, selecting the right one for use by changing the include directories used to build the project.

### Populate the Table

Once your definitions are in place, you need to create a configuration table and populate it with configuration entries for the various devices in the system.

Beningo provides an example timer configuration table:

The configuration table should be placed either in its own file (e.g., `timer_config.c`) or in a file in the application that is designated for tight coupling and handling initialization/configuration.

Some notes on the declaration:

1. As long as you do not want the table to be changeable during operation, it should be declared `const`.
2. You can declare it `static` to limit visibility to the current file.
3. If you do declare it `static` but need to access the pointer to the table in another module in order to pass it to an initialization function, you can define an access function.

### Create the Initialization Routine

Finally, you need an initialization routine that takes the configuration table *pointer* as a `const` input parameter. The routine will iterate through the entries in the table and configure the devices appropriately by writing to registers.

Here Beningo’s timer initialization example. The registers are stored in [pointer arrays](https://embeddedartistry.com/fieldmanual-terms/pointer-array-pattern/).

## Consequences

- Configuration tables externalize application-specific configuration information, resulting in the following beneficial properties:
- [Decoupling](https://embeddedartistry.com/fieldmanual-terms/coupling/) configuration from driver implementations, keeping drivers generic and reusable. Drivers can be reused from one application to the next, and only minor modifications are required to support new microcontrollers.
- Keeping configuration information in a single, easily found location (rather than scattered throughout the application), making it easier to review and modify.
- Configuration table entries can be easily scaled up or down as needed.
- Configuration tables result in increased binary size due to storing tables in flash. The degree to which this impacts a system depends on the amount of flash memory storage and the number and size of tables. For the tiniest microcontrollers, tables can quickly reduce available flash storage. For most modern systems, the impact is negligible.

## Known Uses

- This pattern is described by Jacob Beningo in [\*Reusable Firmware Development](https://amzn.to/3Hkpr9X)\*. Beningo shows examples of using pointer arrays in combination with a configuration table when implementing the initialization routine for a Timer driver and GPIO driver in his HAL, and his examples are shown above. Configuration tables are used to store initial configuration information for all instances used in an application for any given peripheral type.
- Configuration tables can be used to support different board revisions within a single binary. Each revision can have separate tables that map to the specific hardware configuration. On boot, the [software can identify the hardware revision](https://embeddedartistry.com/fieldatlas/versioning-pcbs/) and load the proper table.
- More advanced implementations might use a base table, tracking subsequent revisions based on the “deltas”, which are then copied into the primary table before implementation occurs.
- Alternatively, an update system can download a separate binary file that contains the tables for the target hardware configuration, loading only those tables into flash. This approach trades increased update complexity for reduced on-device storage requirements, since you no longer need to store tables that will not be used by a given configuration.
- Configuration tables are also useful for specifying the initialization parameters for types in an OSAL. For example, a table can be used to manage the threads, mutexes, or [message queue](https://embeddedartistry.com/fieldmanual-terms/messsage-queue/) configurations for a given application. Beningo gives the following FreeRTOS task configuration example in [his article on the subject](https://www.beningo.com/a-simple-scalable-rtos-initialization-design-pattern/).

The corresponding initialization routine would just loop over the structure and create tasks:

- This pattern can be used in conjunction with the [Pointer Array pattern](https://embeddedartistry.com/fieldmanual-terms/pointer-array-pattern/).

## References

- *[Reusable Firmware Development](https://amzn.to/3Hkpr9X)*, by Jacob Beningo, discusses configuration tables in Chapter 4.  

> A good practice is to place the structure definition within a header file, such as timer\_config.h. An example timer configuration structure can be found in Figure 4-19. Keep in mind that once this structure is created the first time, it will only require minor modification to be used with another microcontroller.

> The initialization function can be written to take the configuration parameters for the clock and automatically calculate the register values necessary for the timer to behave properly so that the developer is saved the painful effort of calculating the register values.

> The initialization can be written to simplify the application developers’ software as much as possible. For example, a timer module could have the desired baud rate passed into the initialization, and the driver could calculate the necessary register values based on the input configuration clock settings. The configuration table then becomes a very high-level register abstraction that allows a developer not familiar with the hardware to easily make changes to the timer without having to pull out the datasheet.

> In my own development efforts, I typically design a new HAL as the need arises. Once designed though, I can reuse the HAL from one project to the next with little to no effort. Application code becomes easily reusable because the interface doesn’t change! I use configuration tables to initialize the peripherals, and once the common features are identified, the initialization structure doesn’t change. A typical peripheral driver using the HAL interface takes less than a day to implement in most circumstances.

> The best place to start is at the configuration table. The configuration table lists the primary features of the driver that need to be configured at startup. Manipulating and automating this table and its configuration is the best bet for testing the initialization code.

> Create configuration tables so that drivers and application modules are easily configurable rather than hard coded. Add enough [flexibility](https://embeddedartistry.com/fieldmanual-terms/flexibility/) so that at a later time the software can be improved without bringing down a house of cards.

> The initialization function should take a pointer to a configuration table that will tell the initialization function how to initialize all the Gpio registers. The configuration table in systems that are small could contain nearly no information at all, whereas sophisticated systems could contain hundreds of entries. Just keep in mind, the larger the table is, the larger the amount of flash space is that will be used for that configuration. The benefit is that using a configuration table will ease firmware maintenance and improve readability and [reusability](https://embeddedartistry.com/fieldmanual-terms/reusability/). On very resource-constrained systems where a configuration table would use too much flash space, the initialization can be hard coded behind the interface, and the interface can be left the same.
- [Using Callbacks with Interrupts](https://www.beningo.com/using-callbacks-with-interrupts/) by Jacob Beningo  

> A configuration table could be used to assign the function that is executed. The advantages here are multifold such as:
> 
> - The function is assigned at compile time
> - The assignment is made through a const table
> - The function pointer assignment can be made so that it resides in ROM versus RAM which will make it unchangeable at runtime
- [A Simple, Scalable RTOS Initialization Design Pattern](https://www.beningo.com/a-simple-scalable-rtos-initialization-design-pattern/) by Jacob Beningo  

> I often find that developers initialize task code in seemingly random places throughout their application. This can make it difficult to make changes to the tasks, but more importantly just difficult to understand what all is happening in the application. It also makes it so that the application is not very scalable or easy to adapt and sometimes results in developers not knowing that a task even exists!

> The [design pattern](https://embeddedartistry.com/fieldmanual-terms/design-pattern/), which I often follow for as much of my code as possible, is to create a generic initialization function that can loop through a configuration table and initialize all the tasks.

> There are certainly several different ways that this can be done, but the idea is to make it so that the driver code is constant, unchanging and could even be provided as a precompiled library. The application code can then still easily change the interrupt behavior without having to see the implementation details.

  

---

[« Back to Glossary Index](https://embeddedartistry.com/fieldmanual-terms/)