
#VSCODE #MCU #NXP
We have a [project imported](https://mcuoneclipse.com/2023/08/14/vs-code-import-example-from-mcuxpresso-repository/), have built it, time to debug it on the hardware.

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/debugging-with-vs-code.jpg?w=1024)

Debugging with VS Code

The ‘Debug Probes’ sections shows me the currently connected debug probes:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/debug-probes.jpg?w=335)

Debug Probes listed

To debug a project, I click on the ‘debug’ Icon (triangle) of the the project:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/debug-icon.jpg?w=335)

Debug action for project

In the case of having multiple probes attached, it asks for the one to be used:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/selection-of-probe.jpg?w=740)

Choosing the debug probe

Should I decide later to use a different debug probe, I can reset the probe selection:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/reset-probe-selection.jpg?w=467)

Reset probe selection

Another way to debug to debug the project is using the context menu. From there I can as well erase the flash, program the device without debug and reset the probe selection if needed:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/debug-context-menu.jpg?w=408)

After a few seconds, I can debug my application on the board:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/debugging-with-vs-code.jpg?w=1024)

A small toolbar is used for continue, step over, step into, step out, restart and stop debugging:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/debug-tools.jpg?w=196)

Debug toolbar

Local variables and registers are shown on the left:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/variables.jpg?w=408)

Just below that, there is the view for the call-stack:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/callstack.jpg?w=539)

Breakpoints are set on a source line. Either directly clicking on the left of the line number, or using the context menu which offers additional breakpoint types:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/breakpoints-on-source-line.jpg?w=366)

Source with breakpoints

The breakpoints view is used to manage all the breakpoints:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/breakpoints.jpg?w=429)

Breakpoints

With this, I can flash my application and debug it.

So far I have used projects and devices supported with the NXP MCUXpresso SDK 2.13. My next article shows [how to deal with SDKs which only have on older SDK](https://mcuoneclipse.com/2023/08/20/vs-code-importing-pre-v2-13-0-mcuxpresso-sdk-projects/).

Happy debugging 🙂

### Links

- [VS Code: Import Example from MCUXpresso Repository](https://mcuoneclipse.com/2023/08/14/vs-code-import-example-from-mcuxpresso-repository/)
- [VS Code: MCUXpresso SDK Repository](https://mcuoneclipse.com/2023/08/13/vs-code-mcuxpresso-sdk-repository/)
- NXP extension for Visual Studio Code: [https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-for-visual-studio-code:MCUXPRESSO-VSC](https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-for-visual-studio-code:MCUXPRESSO-VSC)