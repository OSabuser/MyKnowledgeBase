#VSCODE #MCU
There are many cases where setting a breakpoint on a line of code does not help much. Cases like data or variables get modified from somewhere. That can be data in a linked list somewhere, and all what I have found out so far that it gets changed or corrupted. But I do not know what piece of code is responsible for it.

The solution for such problems are ‘data breakpoints’ or ‘[watchpoints](https://mcuoneclipse.com/2012/04/29/watchpoints-data-breakpoints-in-mcu10/)‘. Still, not many developers seem to be aware of watchpoints? They are incredibly helpful. And VS Code has at least some basic support for it.

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/break-on-data-change.jpg?w=426)

Data Breakpoints in VS Code

## Embedded Target

Point in case: I had to find a defect in a firmware for the NXP LPC55S16:

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/lpc55s16-evk.jpg?w=597)

NXP LPC55S16-EVK

It took me a while to find out how to use watchpoints in that project using [VS Code](https://mcuoneclipse.com/2023/10/02/ci-cd-for-embedded-with-vs-code-docker-and-github-actions/).

I have to use the context menu on a variable inside the VARIABLES pane: There I can choose between

- **Break on Value Read**: Break for value **read**
- **Break on Value Change**: Break for value **write**
- **Break on Value Access**: Break on variable **read or write**

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/break-on-data-change.jpg?w=426)

Context Menu on Variables to break on date read or write

It only is supported by GDB based debug probes, and is for example working with [cortex-debug by marus](https://github.com/Marus/cortex-debug).

## Breakpoint List

After setting a data breakpoint, it shows up in the Breakpoints list with a red hexagon (very hard to distinguish from the normal red circle for breakpoints):

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/data-breakpoint-in-the-breakpoints-view.jpg?w=461)

Data Breakpoints

The context menu on that view can be used to delete the breakpoint (or in this case, the watchpoint).

## Debug Console

If the watchpoint condition triggers, it halts the debugger and I can see the reason in the Debug Console:

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/image.png?w=1020)

Watchpoint is hit

I can use the GDB command-line in the Debug Console to inspect the watchpoints:

```
info watchpoint
```
![](https://mcuoneclipse.com/wp-content/uploads/2023/11/info-watchpoint.jpg?w=1024)

info watchpoint GDB command

## Summary

With this, I can find out who is accessing/reading/writing a memory location or variable. Just keep in mind that the hardware resources for watchpoints are limited, so best if you just use few or only one.

Happy watching 🙂

### Links

- [Watchpoints: Data Breakpoints](https://mcuoneclipse.com/2012/04/29/watchpoints-data-breakpoints-in-mcu10/)
- [Tutorial: Catching Rogue Memory Accesses with ARM Watchpoint Comparators and Instruction Trace](https://mcuoneclipse.com/2018/08/12/tutorial-catching-rogue-memory-accesses-with-arm-watchpoint-comparators-and-instruction-trace/)
- VS Code: Data Breakpoints: [https://code.visualstudio.com/Docs/editor/debugging#\_data-breakpoints](https://code.visualstudio.com/Docs/editor/debugging#_data-breakpoints)
- Cortex-Debug Extension: [https://github.com/Marus/cortex-debug](https://github.com/Marus/cortex-debug)
- GDB: working with watchpoints: [https://sourceware.org/gdb/download/onlinedocs/gdb/Set-Watchpoints.html](https://sourceware.org/gdb/download/onlinedocs/gdb/Set-Watchpoints.html)