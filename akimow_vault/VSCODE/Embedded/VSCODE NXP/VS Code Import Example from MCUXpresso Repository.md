#NXP #MCU #VSCODE 
After having [imported the repository with the SDK](https://mcuoneclipse.com/2023/08/13/vs-code-mcuxpresso-sdk-repository/), it is now time to create a first project.

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/blnky.png?w=1024)

Blinky Example in VS Code with the SDK

We have previously imported the SDK repository with the examples. Here we are going to create a new project from the examples in the repository.

From the extension, click on ‘**Import Example from Repository**‘:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/import-example-from-repository.png?w=366)

Import Example from Repository

Next, you can configure what you want to import:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/import-example-settings.png?w=774)

Import Example settings

Press ‘Create’ and it creates a project. The project then shows up under ‘Projects’ with extra information:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/projects.png?w=351)

Project details

All the source files are visible with the Explorer view, which is usually used in VS Code to inspect the files:

![](https://mcuoneclipse.com/wp-content/uploads/2023/08/project-explorer.png?w=358)

Explorer in VS Code

There are following folders inside the project:

- **\_\_repo\_\_** conains files from the repository from where we copied the example, notably the device specific files, startup code and device drivers.
- **.vscode** is a standard VS Code folder which contains configuration .json files
- **armgcc** contains the CMake build files

Beside the source files in the top folder, there is a .mex file for the graphical configuration utility to set pin muxing and the like. The `.xml` file is used as a description of the project created.

There is currently no way to create a ‘new/empty’ project. An example is a good starting point, and I will cover to create a project from scratch in a later article.

With this, we have a project we can build and debug, which is a topic for yet another article.

Happy importing 🙂

### Links

- [VS Code: MCUXpresso SDK Repository](https://mcuoneclipse.com/2023/08/13/vs-code-mcuxpresso-sdk-repository/)
- [VS Code: MCUXpresso Installer](https://mcuoneclipse.com/2023/08/11/vs-code-mcuxpresso-installer/)
- NXP extension for Visual Studio Code: [https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-for-visual-studio-code:MCUXPRESSO-VSC](https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-for-visual-studio-code:MCUXPRESSO-VSC)