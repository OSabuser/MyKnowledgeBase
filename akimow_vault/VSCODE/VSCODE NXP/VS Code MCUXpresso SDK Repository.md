#MCU #NXP #VSCODE

In a previous article I have installed the [MCUXpresso extension](https://mcuoneclipse.com/2023/08/09/vs-code-mcuxpresso-extension/) and used the [MCUXpresso Installer](https://mcuoneclipse.com/2023/08/11/vs-code-mcuxpresso-installer/) to install the necessary development tools.

In this article I’m going to import the SDK.

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/vs-code-import-repository.png?w=1024)

Import Repository in VS Code

Development with VS Code is built around using a version control system. If you have not used a version control system, then definitely you should look at it. If this is new to you, have a look at the documentation and tutorials on [https://git-scm.com/](https://git-scm.com/).

Note: At the time of writing this article, only NXP MCUXpresso SDKs with a version 2.13 are supported to import. If using an earlier SDK, have a look at [Building a Triumvirate: From Eclipse CDT to CMake, CMD and Visual Studio Code](https://mcuoneclipse.com/2023/04/19/building-a-triumvirate-from-eclipse-cdt-to-cmake-cmd-and-visual-studio-code/).

## Import Repository

In a first step, I’m going to import a (git) repository. The NXP MCUXpresso comes as a git repository, and this is what I’m going to import.

From the extension, I select the ‘Import Repository’ item:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/extension-import-repository.png?w=356)

Import Repository Action

## Repository Installation

There are three choices:

1. **Remote**: clone one of the remote repositories
2. **Local Archive**: use a zip file and create a repository from it
3. **Local**: specify the location of an already cloned repository

If you start with VS Code, you can chose from the first two options. Which one you want to select depends on how much disk space you have available.

## Import Remote Repository

If you don’t know yet, for which device you want to develop, and you have ~30 GB disk space available, then import the remove (full) SDK repository:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/import-repository.png?w=679)

Import Repository Dialog

In the dialog above I have requested to include the examples which require around 4 GB of disk space.

## Import Local (Zip) Archive

The other way is to build an SDK Zip file on [https://mcuxpresso.nxp.com](https://mcuxpresso.nxp.com/) for GCC ARM Embedded:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/build-sdk.png?w=1024)

Build SDK on mcuxpresso.nxp.com

This is my preferred way, because I can select exactly what I want and the zip/repository is only a few 100 MByte in size.

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/import-sdk-zip-repository.png?w=520)

Import local archive

## Import Local Repository

If you have cloned a repository already, the fastest way is to import it as a local repository: browse to the folder where the .git folder is located:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/import-local-repository.png?w=626)

## Repository List

The installed repository then shows up in the list:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/installed-repository.png?w=470)

The same view can be used to manage the repositories, to add and remove them.

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/installed-repository-1.png?w=466)

Managing installed repositories

With this, we have the repository with the SDK installed, and can use it to create a first example. This is [subject of a next post](https://mcuoneclipse.com/2023/08/14/vs-code-import-example-from-mcuxpresso-repository/).

Happy coding:-)

### Links

- [VS Code: Installation](https://mcuoneclipse.com/2023/08/05/vs-code-ide-installation/)
- [VS Code: Getting Started, literally](https://mcuoneclipse.com/2023/08/07/vs-code-getting-started-literally/)
- [VS Code: MCUXpresso Extension](https://mcuoneclipse.com/2023/08/09/vs-code-mcuxpresso-extension/)
- [VS Code: MCUXpresso Installer](https://mcuoneclipse.com/2023/08/11/vs-code-mcuxpresso-installer/)
- [VS Code: Import Example from MCUXpresso Repository](https://mcuoneclipse.com/2023/08/14/vs-code-import-example-from-mcuxpresso-repository/)
- NXP extension for Visual Studio Code: [https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-for-visual-studio-code:MCUXPRESSO-VSC](https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-for-visual-studio-code:MCUXPRESSO-VSC)