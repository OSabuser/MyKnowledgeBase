#VSCODE #NXP #MCU #CMAKE
In a [previous article](https://mcuoneclipse.com/2023/08/14/vs-code-import-example-from-mcuxpresso-repository/) I have imported an example project. Now I want to compile and build it.

Traditionally, the build action inside VS Code is somewhat hidden. There is a keyboard shortcut, but recent additions to VS Code making the build action more accessible.

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/different-build-actions-in-vs-code.jpg?w=1024)

Multiple ways to start a build in Visual Studio Code

To build a project, you have multiple options.

## CTRL+SHIFT+B

One standard way in VS Code is to use the `CTRL+SHIFT+B`:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/build-with-ctrl-shift-b.jpg?w=1024)

CTRL+SHIFT+B

This gives the actions to build or clean the project.

The output is shown for example in the Terminal:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/build-output.jpg?w=834)

Another place is the Output area for CMake/Build:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/cmake-and-build-output.jpg?w=950)

## Toolbar Icon

Yet another way is to use the build icon in the bottom toolbar:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/bottom-toolbar-build-icon.jpg?w=514)

Build action in bottom toolbar

The same icon is present to build the selected project:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/build-selected-project.jpg?w=376)

Build action on project list

## Context menu

The context menu on the project offers the same action, among others:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/context-menu-on-project.jpg?w=367)

Build action on project list context menu

## Build errors

Should the build fail with an error, then this is indicated in different places in VS Code. Check the ‘Problems’ in the bottom area, where you can double-click on the message to jump to the source location:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/failed-build.jpg?w=1021)

Failed build with an error

## Command Line

As the project is CMake based, I can easily build project from the command line. In the output view I can see what command is used:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/build-commandline.jpg?w=680)

I can open the project folder with the integrated terminal:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/open-in-integrated-terminal.jpg?w=430)

and using the commands to build it:

```
cmake --build armgcc/debug --config debug --target all
```
![](https://mcuoneclipse.com/wp-content/uploads/2023/08/terminal-with-command.jpg?w=867)

or to clean the project:

```
cmake --build armgcc/debug --config debug --target clean
```

## Summary

With this, I can compile and build projects. If you are looking for a more advanced way, check out the [usage of build tasks](https://mcuoneclipse.com/2021/05/06/visual-studio-code-for-c-c-with-arm-cortex-m-part-3/). Next step is to [debug the project](https://mcuoneclipse.com/2023/08/16/vs-code-debugging/).

Happy making 🙂

### Links

- [VS Code: Installation](https://mcuoneclipse.com/2023/08/05/vs-code-ide-installation/)
- [VS Code: Getting Started, literally](https://mcuoneclipse.com/2023/08/07/vs-code-getting-started-literally/)
- [VS Code: MCUXpresso Extension](https://mcuoneclipse.com/2023/08/09/vs-code-mcuxpresso-extension/)
- [VS Code: MCUXpresso Installer](https://mcuoneclipse.com/2023/08/11/vs-code-mcuxpresso-installer/)
- [VS Code: Import Example from MCUXpresso Repository](https://mcuoneclipse.com/2023/08/14/vs-code-import-example-from-mcuxpresso-repository/)
- NXP extension for Visual Studio Code: [https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-for-visual-studio-code:MCUXPRESSO-VSC](https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-for-visual-studio-code:MCUXPRESSO-VSC)
- [Visual Studio Code for C/C++ with ARM Cortex-M: Part 3 – Build](https://mcuoneclipse.com/2021/05/06/visual-studio-code-for-c-c-with-arm-cortex-m-part-3/)