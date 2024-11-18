#MCU #C
When developing with C or C++ an application, then you mostly focus on your own code. You don’t want to bother with the details how input/output functions like printf() or scanf(), and you might just use these functions and helpers and that’s it.

The implementation is part of the ‘C Standard Library’ (or C++ Standard Library). In the world of Linux, this is usually the ‘glibc’ or ‘GNU C Library, and one usually link with ‘libc’. That provides the implementation of printf(), or use ‘libm’ if using math functions like sin() or cos().

In the embedded world, things are much more complex, with plethora of choices, for example in the [MCUXpresso IDE](https://mcuoneclipse.com/2023/01/25/nxp-mcuxpresso-ide-11-7-0/):

![](https://mcuoneclipse.com/wp-content/uploads/2023/01/different-libraries.jpg?w=807)

Library Selection in MCUXpresso IDE

## Outline

When the [C programming language](https://en.wikipedia.org/wiki/C_\(programming_language\)#) was developed by Dennies Ritchie back in the 1970s, it was about the programming language, and not so much about things like ‘printf’ and the like, which is commonly referred as the ‘[C Standard Library](https://en.wikipedia.org/wiki/C_standard_library)‘. The ‘Standard’ has been evolved over time, and has been extended with the C++ libraries.

Good developers should be fully aware not only about the consequences of the programming language used, but as well about the implications of using the associated library.

Using the Standard Library can be problematic in many ways: from code and memory bloat (see [Why I don’t like printf()](https://mcuoneclipse.com/2013/04/19/why-i-dont-like-printf/)) up to many reentrancy issues for example with malloc (see [How to make sure no Dynamic Memory is used](https://mcuoneclipse.com/2022/11/06/how-to-make-sure-no-dynamic-memory-is-used/) or [Steps to use FreeRTOS with newlib reentrant Memory Allocation](https://mcuoneclipse.com/2020/11/15/steps-to-use-freertos-with-newlib-reentrant-memory-allocation/)).

## Runtime Routines

As for the GNU gcc, the libraries bundled with the build tools do not only include the ‘standard’ functions link printf(), but as well so called ‘runtime routines’. Runtime routines are called and inserted by the compiler if the target machine is not able to do something directly. For example an ARM Cortex-M0+ has no FPU for floating point operations, so if you have something like this:

```
float add(float f, float t) {
  return f+g;
}
```

There is no hardware ‘FADD’ instruction, so the compiler as to call little (or larger) helper routines to perform the operation:

![](https://mcuoneclipse.com/wp-content/uploads/2023/01/aeabi_fadd.jpg?w=701)

\_\_aeabi\_fadd

In the above code example and assembly listing, the compiler is calling the \_\_aeabi\_fadd() runtime routine in the library to do the floating point addition. There are many more of such help routines, from math operations up to things like selecting an entry in a switch statement. The number and naming of these runtime routines depends on the compiler used, and can change from one compiler version to another.

The GNU Library (glibc) used for Linux has two issues: first it is licensed under [GPL](https://en.wikipedia.org/wiki/GNU_General_Public_License) which is a problem for many embedded applications (you would have to make your application open source too). Second, the glibc is very large and most likely will not fit in the the limited memory of an embedded device. This is why [Newlib](https://en.wikipedia.org/wiki/Newlib) had been created by Cygnus around the year 2000 is now maintained by Red Hat: an open source port of the Standard Library to bring portability with a permissive BSD-like license and with a smaller footprint. As such, the Newlib is what you usually get delivered with a GNU gcc distribution from NXP, STM, Espressif and many others.

### **When using Newlib?**

I recommend using newlib if:

- Plenty of RAM and FLASH available, e.g. can easily spend 50 KByte of Flash and 25 kByte of RAM for it
- Speed is more important than memory footprint, because I/O operation are buffered
- Need a fully compliant standard library, e.g. supporting all printf() formatting strings
- Need to use full C++ including exception
- Wants to keep cross-platform compatibility across different CPU architectures (ARM, RISC-V, …)
- Wants a free-of-charge library
- Needs for extensive documentation and tutorials
- Want to use advanced tools like [gprof](https://mcuoneclipse.com/2015/08/23/tutorial-using-gnu-profiling-gprof-with-arm-cortex-m/) or [gcov](https://mcuoneclipse.com/2021/09/19/tutorial-gnu-gcov-coverage-with-the-nxp-i-mx-rt1064/) with file I/O
- Prefers to use an open source library

## Newlib-nano

The Newlib standard library was a success and widely used for embedded targets, and still is used. But the library code and RAM footprint was not small. ARM realized that too, that Newlib is a problem for smaller ARM Cortex-M devices, say with 4 KByte of FLASH and 1 KByte of RAM. That’s why back in 2013 with the [GCC ARM Embedded 4.7 Release](https://community.arm.com/arm-community-blogs/b/embedded-blog/posts/shrink-your-mcu-code-size-with-gcc-arm-embedded-4-7) the ‘Newlib-nano’ was introduced. ARM developed a stripped down version of Newlib with focus on memory size. It simplifies the code with removing features which have been added after the C89 standard. For example there is no wide-character support, or printf() has removed floating point support. The library is built as well without C++ exception support.

What library is linked with the application is configured by the [Spec-File](https://gcc.gnu.org/onlinedocs/gcc/Spec-Files.html). Newlib-nano for example is linked with

```
--specs=nano.specs
```

### When using newlib-nano?

I recommend using newlib-nano if:

- Values smaller code size and smaller RAM usage over performance
- Wants faster and smaller heap usage compared to Newlib
- Wants to keep cross-platform compatibility within the ARM ecosystem
- Does not need C++ exception handling
- Wants to use GNU tools like [gcov](https://mcuoneclipse.com/2021/09/19/tutorial-gnu-gcov-coverage-with-the-nxp-i-mx-rt1064/) or [gprof](https://mcuoneclipse.com/2015/08/23/tutorial-using-gnu-profiling-gprof-with-arm-cortex-m/)
- Wants a free-of-charge library
- Prefers to use an open source library

## Floating Point Options

Newlib-nano greatly benefits from not having floating point support in printf() and scanf(). But if this is still needed, it can be [enabled with linker options](https://mcuoneclipse.com/2014/07/11/printf-and-scanf-with-gnu-arm-libraries/) or with a setting in the MCUXpresso IDE:

![](https://mcuoneclipse.com/wp-content/uploads/2023/01/float-for-printf-and-scanf-in-newlib-nano.jpg?w=798)

Newlib-nano with float enabled for printf and scanf

The options can be turned on with spec files too:

I recommend turning on these options only if using float with printf and scanf cannot be avoided.

## Vendor Specific Libraries

ARM was not the only one trying to shrink the Standard Library. For example NXP and others provide their own custom library, aiming at an even lower footprint than the Newlib-nano. For example NXP provides the ‘Redlib’ with different variants:

![](https://mcuoneclipse.com/wp-content/uploads/2023/01/redlib.jpg?w=634)

Redlib

> 💡 NXP acquired ‘Code Red’ back in [2013](https://mcuoneclipse.com/2013/04/20/red-suite-5-eclipse-juno-processor-expert-and-unlimited-frdm-kl25z/) who had developed the ‘Redlib’.

The Redlib is a proprietary (non-GNU) and closed source, except the provided header files. It comes with some C90 and C99 implementations, for example single precision math functions, stdbool an itoa() implementation. The library does not support C++.

I recommend using a proprietary library like the RedLib, if:

- No or minimal support for standards is needed
- Wants to have the smallest footprint
- No requirements or need for portability
- No cross-vendor or cross-platform compatibility needed
- No support for wchar needed
- Does not use C++
- No need for source files or documentation
- Wants a free-of-charge library bundled with the tools
- Does not use any special gcc features like gcov/gprof or -fstack-protector
- Does not care if it is open source or not

## Host, Semihosting and Retargeting

One advantage of newlib and newlib-nano is that they have built in low-level I/O redirection built in with hooks for redirection and re-targeting. For example file or standard I/O operations can be redirected.

For example a call to printf() will call a \_write() function to write the data, with a low level function interface like below:

```
int _write(int fd, char *buffer, unsigned int count);
```

Note that there is a file handle ‘fd’, as like everything in Linux is a file, with stdin, stdout and stderr special ‘file’ handles. Such callbacks allow to direct the data streams for example to a UART or even to a physical file on the host.

Vendors including NXP provide different library variants with different ‘default’-hooks and handlers implemented. In the case of NXP, there are the following:

![](https://mcuoneclipse.com/wp-content/uploads/2023/01/library-variants.jpg?w=618)

- (**none**): the hooks are not not implemented at all. If they are used and not implemented, the linker will report an error. Use this for the smallest memory footprint and to ensure no stubs are used.
- (**nohost**): the hooks are implemented as ’empty’ functions, doing nothing. Like a NULL device. Use this variant if it is OK to have things like printf() in the application, but they will not do anything.
- (**semihost**): here standard I/O and file I/O operations are redirected to the host through the debugger ([semihosting](https://mcuoneclipse.com/2022/01/25/semihosting-with-nxp-mcuxpresso-sdk-and-frdm-ke02z/)). With no debugger attached, this will cause a hard fault on ARM Cortex-M, unless the application have as [special hardfault handler](https://github.com/ErichStyger/McuOnEclipseLibrary/blob/master/lib/src/McuHardFault.c) implemented. Communication with the host will rather slow. Choose this option only if semihosting is needed and used.

For the RedLib variant there more options: **nf** for (no-file) and **mb** for (memory buffer). Use the nf variant if no file operations are used (like fopen()). Use the mb variant to speed up the operation, at the expense of more memory used.

## Custom Library

The biggest problem with the Standard Library is the memory bloat with printf() and all the reentrancy issues. This is why many companies and universities maintain use their own subset of library functions. For example there are [better open source printf solutions](https://mcuoneclipse.com/2014/08/17/xformat-a-lightweight-printf-and-sprintf-alternative/). That can be easily combined with the needed middle-ware like RTOS or networking stacks. For example at the Lucerne University we maintain and use the ‘[McuLib](https://github.com/ErichStyger/McuOnEclipseLibrary)‘. The good thing about this is that it can be easily combined with different SDKs and used as an extension of other libraries as the newlib-nano.

### When using a custom library?

I recommend using a custom library, if

- Cross-platform and cross-vendor support is important
- Wants to combine and cherry-pick all with other library concepts
- Has the time and resources to maintain and configure it, or integrates open source solutions
- Wants to have a common set of utilities, middleware and software libraries across multiple teams
- Wants to have everything configurable

## Commercial Libraries

It is possible to use commercial available libraries too. For example SEGGER offers the [emLib library](https://www.segger.com/downloads/emlib/) and and commercial versions of the Standard Library as [emRun](https://www.segger.com/products/development-tools/runtime-library/) and [emRun++](https://www.segger.com/products/development-tools/emrunpp/). Such libraries aim at even lower footprint than newlib-nano or lower memory impact for C++ compared using newlib.

### When to use a commercial library?

I recommend using a commercial library, if

- Commercial and paid support is important
- Want to have custom functionality implemented
- Cross-Vendor and Cross-Platform support needed
- Has the money and budget
- Need special optimizations, e.g. for C++

## Keep it Pure and Clean: No Libraries

A good way to have the application small and efficient is clearly not adding bloat with printf() or C++ were it is not justified. Such an approach exists with the ‘EC++’ ([Embedded C++](https://en.wikipedia.org/wiki/Embedded_C%2B%2B)) where costly features are not allowed.

But maybe there are hidden malloc() or printf() used somewhere? How to make sure that such things are not used at all? The NXP MCUXpresso IDE for example has a dedicated library option to have you covered:

![](https://mcuoneclipse.com/wp-content/uploads/2023/01/no-libraries.jpg?w=917)

Linker Configuration to use no Standard Library and Runtime Library

If the applications uses anything from the standard library or runtime routines: I get a linker error and I know about it.

### When using No Libraries?

I recommend using ‘No Libraries’, if

- Want to keep everything it under full control
- Need to check against library usage (as a test)
- Want to use a commercial library instead with runtime functions

## Summary

Most applications use the Standard Library functions in one or the other way. Many developers probably are not aware of the implications, and probably just go with the default (which can be fine). But there are many different choices and options, so it can really pay off to make the library selection an informed and justified one. Personally, my preference is using the newlib-nano for most cases: it is open source, free-of-charge and gives reasonable performance. In most of my projects I combine it with using the McuLib which avoids many of newlib and newlib-nano shortcomings like reentrancy, buffer overflows or memory usage. I’m always amazed how few embedded developers care about reentrancy and footprint. Maybe this is just me?

An interesting inflection point might be C++: It could be used with tiny and small embedded devices. It is only that I rarely see it used that way, and if C++ is not wisely used, it many times is just an overkill. With the advent of more and more powerful devices with plenty of RAM and FLASH, this is much less of a concern, or no concern at all. With +128KByte RAM and +1MByte FLASH, and application can go full C++, and even does not need an optimized library. So it can easily go with newlib, why not?

But you might have different priorities and needs? For this I hope I gave you some insights and decision criteria?

Happy choosing 🙂

### Links

- C Standard Library: [https://en.wikipedia.org/wiki/C\_standard\_library](https://en.wikipedia.org/wiki/C_standard_library)
- Newlib: [https://en.wikipedia.org/wiki/Newlib](https://en.wikipedia.org/wiki/Newlib)
- [Why I don’t like printf()](https://mcuoneclipse.com/2013/04/19/why-i-dont-like-printf/)
- [Steps to use FreeRTOS with newlib reentrant Memory Allocation](https://mcuoneclipse.com/2020/11/15/steps-to-use-freertos-with-newlib-reentrant-memory-allocation/)
- [How to make sure no Dynamic Memory is used](https://mcuoneclipse.com/2022/11/06/how-to-make-sure-no-dynamic-memory-is-used/)
- [GCC ARM Embedded 4.7 Release](https://community.arm.com/arm-community-blogs/b/embedded-blog/posts/shrink-your-mcu-code-size-with-gcc-arm-embedded-4-7)
- McuLib: [https://github.com/ErichStyger/McuOnEclipseLibrary](https://github.com/ErichStyger/McuOnEclipseLibrary)