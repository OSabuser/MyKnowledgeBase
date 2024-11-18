
#VSCODE #MCU #NXP
In essence, VS Code is only a text editor, although with very nice features. An editor which can be enhanced with so called **extensions**. With the right set of extensions, VS Code can be specialized for web development, or even [LaTeX](https://mcuoneclipse.com/2013/10/07/compiling-documentation-and-presentations-latex/) documenting. Or as in the case of this article series, for embedded development.

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/installed-extensions.png?w=833)

Installed VS Code Extensions

‘Full’ IDEs as [Microsoft Visual Studio](https://visualstudio.microsoft.com/) (the non-Code one!) or the [Eclipse based MCUXpresso IDE](https://mcuoneclipse.com/2023/08/06/mcuxpresso-ide-11-8-0/) do come with all the necessary build tools and debugger.

VS Code is different, as it comes as editor only in the first place. Which is good because it gives you full freedom. If you are familiar with building a [DIY IDE](https://mcuoneclipse.com/2017/07/30/breathing-with-oxygen-diy-arm-cortex-m-cc-ide-and-toolchain-with-eclipse-oxygen/), then this is just perfect. You can follow [Visual Studio Code for C/C++ with ARM Cortex-M: Part 1 – Installation](https://mcuoneclipse.com/2021/05/01/visual-studio-code-for-c-c-with-arm-cortex-m-part-1/)) to install the extensions and tools.

## MCUXpresso for Visual Studio Code

After having [installed VS Code](https://mcuoneclipse.com/2023/08/05/vs-code-ide-installation/) and [configured an essential part](https://mcuoneclipse.com/2023/08/08/vs-code-color-themes/) :-), time to install some extensions.

I start with the installation of the [NXP extension for Visual Studio Code](https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-for-visual-studio-code:MCUXPRESSO-VSC). It comes with many of the essential extensions needed for embedded development. Plus it makes it easy to get and install the necessary tools like git, gcc, gdb and cmake. More about this in a next article.

[NXP has released a VS Code extension and installation package](https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-for-visual-studio-code:MCUXPRESSO-VSC), making it easier to use VS Code for embedded development.

> 💡 I’m not a lawyer, and this is **not** a legal advice. But you should carefully read and check the fine print and license agreement of VS Code and any extensions you are going to use and install. You usually find the link to the license and terms of usage on the extension page. If you prefer a more permissible license, check out [VSCodium](https://vscodium.com/) (MIT) or the [Eclipse Theia](https://theia-ide.org/) project.

## Installing MCUXpresso Extension

Start VS Code. Then click on the Extensions icon on the left side-bar:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/extensions-icon-in-sidebar.png?w=319)

Extensions Icon

Then search for the NXP extension and click on ‘Install’:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/mcuxpresso-for-vs-code-1.png?w=352)

This installs the extension plus if necessary all the VS Code extensions it depends on.

You can see and manage the installed extensions in the **Extensions** area:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/installed-extensions.png?w=833)

- **C/C++**: provides syntax coloring for C/C++ and editing (IntelliSense) support, plus basic debugging infrastructure.
- **CMake**: provides CMake file syntax coloring in the editor.
- **CMake Tools**: Extension supporting a CMake based flow to configure projects for CMake.
- **Embedded Tools**: Implements a register viewer for [CMSIS-SVD](https://mcuoneclipse.com/2017/08/07/adding-cmsis-svd-files-to-embsysregview-and-eclipse/) files.
- **Serial Monitor**: Extension implementing support for a serial terminal or serial monitor, both over a serial or TCP port

Note that the above is for the editor, and does not include the external tools itself (gcc, gdb, cmake, …).

What MCUXpresso Extension has added to the UI is a ‘X’ in the sidebar (which I will cover in a next article):

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/installed-extension.png?w=1024)

With this, we have the essential VS Code extensions installed.

In a next article I show how to install the needed build tools (gcc, gdb, cmake, …).

Happy extending 🙂

### Links

- NXP community forum: [https://community.nxp.com/t5/MCUXpresso-for-VSCode/bd-p/mcuxpresso-vscode](https://community.nxp.com/t5/MCUXpresso-for-VSCode/bd-p/mcuxpresso-vscode)
- NXP extension on VS Code marketplace: [https://marketplace.visualstudio.com/items?itemName=NXPSemiconductors.mcuxpresso](https://marketplace.visualstudio.com/items?itemName=NXPSemiconductors.mcuxpresso)
- NXP web page: [MCUXpresso for VS Code](http://www.nxp.com/vscode)