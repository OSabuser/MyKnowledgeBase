#VSCODE 

What makes Microsoft Visual Studio Code really useful is the concept of **Extensions**: Probably for every problem or use case you might find an extension. There are [more than 40K extensions available for VS Code](https://blog.aquasec.com/can-you-trust-your-vscode-extensions). And VS Code asks to install extensions:

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/do-you-want-to-install-extension.png?w=463)

VS Code asking to install an extension

The issue with this is: more and more extensions get added, making VS Code slower and slower, caused by that ‘extension creep’. Even worse: extensions can cause conflicts, and clutter the development flow. Luckily, there is a cure for this in VS Code: **[Profiles](https://code.visualstudio.com/docs/editor/profiles)**.

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/currently-active-vs-code-profile.png?w=1024)

Active Profile in VS Code

‘Profiles’ have been introduced to VS Code in [February 2023 version 1.76](https://code.visualstudio.com/updates/v1_76), as [‘one of the All-Time Most Requested VS Code Features’](https://visualstudiomagazine.com/articles/2023/03/09/vs-code-profiles.aspx). With a profile I can quickly switch my current workflow for a project. The profile includes settings (like [colors](https://mcuoneclipse.com/2023/08/08/vs-code-color-themes/)), keybindings, and especially the set of [extensions](https://mcuoneclipse.com/2023/08/09/vs-code-mcuxpresso-extension/). The profile used is stored with the [workspace settings](https://code.visualstudio.com/docs/getstarted/settings).

The current active profile is visible in the left toolbar:

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/active-profile.jpg?w=1024)

Current Profile

Clicking on that icon I can manage my profiles (create, edit, delete, export, import).

I’m using profiles to switch between different development flows and lecture modules (which use different extensions):

![](https://mcuoneclipse.com/wp-content/uploads/2023/11/set-of-defined-profiles.jpg?w=1024)

Because the profile gets associated with a workspace, I organize my ESP32, NXP, … project in workspaces too, and have the profile assigned to it. So switching or loading a workspace will have the correct profile with the required extensions loaded to it. This is especially useful, as the vendor extensions are adding UI elements (buttons, toolbar items, sidebar items) to VS Code, and they will clutter up the UI, make it slow or even break things between extensions. Using profiles I can keep them separated, assigned to workspace and can easily switch a workspace with the profile.

Happy profiling 🙂

### Links

- Microsoft VS Code Extension store: [https://marketplace.visualstudio.com/vscode](https://marketplace.visualstudio.com/vscode)
- VS Code Profiles: [https://code.visualstudio.com/docs/editor/profiles](https://code.visualstudio.com/docs/editor/profiles)