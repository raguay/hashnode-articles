## Modal File Manager: Update and Extensions

In my last article on [Modal File Manager](https://blog.customct.com/my-side-project-modal-file-manager), I mentioned that it allows for extensions. Though this article isn’t about writing them, I thought I would introduce you to the ones I’ve written and talk about some additions to the Modal File Manager.

## Program Installation

I’ve been packaging a full program with the latest releases. You have to unzip the `mfm-mac.zip` file downloaded from the [releases page](https://github.com/raguay/ModalFileManager/releases) and move it to your `/Applications` directory. 

Since this program isn’t signed, macOS has you go through loops to launch it. You will need to launch the `System Preferences` program and go to the `System Preferences->Security & Privacy->General` dialog. Make sure the system has the `App Store and identified developers` radial selector selected. Then run the program.

It will first give a dialog to delete the program. 

![VerifyDeveloper2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624431365431/vf8K3fc-y.png)

You wil need to hit cancel, and go to the `System Preferences->Security & Privacy->General` dialog and press the button `open anyway`. 


![SecurityPrivacy1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624431324847/zWkWfM1jr.png)

The next time you launch it, it will ask again to run it or not. This time you tell it to open the program and it should launch. 

![VerifyDeveloper.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624431300480/msdiMLQqD.png)

In the `System Preferences->Security & Privacy->Privacy` dialog, you will need to go to the `Full Disk Access` on the left side and click the `+` button on the right. Select the `mfm` program and click `open`. It will ask to close and reopen the program. When you click on `reopen`, it will close it but it will not reopen. You will have to launch it yourself again. Then you should be able to use the program normally.

Once you have the Modal File Manager running, you will notice that it has the name `nwjs` in the processes panel. `nwjs` is the name of the program that I’m using to display the program. My work is all in HTML, CSS, and JavaScript using the Svelte framework. That is because my attempt to make it unique is constantly failing. But, I’m working to improve this situation.

## Key Map Additions

I’ve added some keyboard defaults to the latest version. If you have already downloaded and used Modal File Manager, then you will need to delete the folder `~/.config/modalfilemanager/KeyMaps` and then run the newer version of the program. 

## Themes

These themes are available to install for Modal File Manager. They are downloadable inside the program by running `Toggle GitHub Importer`. You can then select the theme you want to download and use. You can also switch themes in the new **Preferences** panel. All themes in GitHub have to have the labels `theme` and `modalfilemanager` in order for the program to find them on GitHub.

|   Name  |   Description   |
| --- | --------|
| Dracula Pro | This is the Dracula Pro colors that comes standard with the program. |
| Dracula Pro Buffy | This is the Buffy variation to Dracula Pro |
| Dracula Pro Van-Helsing | This is the Van Helsing variation to Dracula Pro |

## Extensions

These extensions are available to install for Modal File Manager. They are downloadable inside the program by running `Toggle GitHub Importer`. Then you can select the extension you want to download. All downloaded extensions are loaded each time in Modal File Manager. All extensions in GitHub have to have the labels `extension` and `modalfilemanager` in order for the program to find them on GitHub.

|   Name  |   Description   |
| --- | --------|
| [Alfred](https://github.com/raguay/Alfred-ModalFileManager) | This extension assigns the `a` key in `normal` mode to open the current cursor in the Alfred Prompt. |
| [CopyToClipboard](https://github.com/raguay/CopyToClipboard-ModalFileManagerExtension) | This adds commands for copying the name or path of the current cursor to the clipboard. |
| [Favorites](https://github.com/raguay/Favorites-ModalFileManagerExtension) | This extension allows for the creation/deleting of favorite directories to quickly jump to, a `fav` mode for saving and jumping to directories quickly (not saved between runs of the program), and backtracking to previously entered directories. |
| [iTerm](https://github.com/raguay/iTerm-ModalFileManagerExtension) | This extension assigns the `O` key in `normal` mode to open the current cursor directory in iTerm2. |
| [moveToDir](https://github.com/raguay/moveToDir-ModalFileManagerExtension) | This extension moves the currently selected files to the current directory that the cursor is on using the `M` key in `normal` mode. |
| [runScripts](https://github.com/raguay/runScripts-ModalFileManagerExtension) | This extension allows you to create user scripts that can be ran from Modal File Manager. These scripts will get the current properties of the Modal File Manager in the environment. Please see the extension’s page for move details. |

I’m still working on the Project Manager with Notes extension and other extensions. If there is an extension you would like to see, just let me know.

## Preferences Panel

Running the command `Show Preferences` will open the Preferences Panel:

![Pref-General1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624432411826/3JFYKvGsI.png)

Here, you can change many aspects of the Modal File Manager. As the program expands, more parts will naturally be added to the preferences panel.

There are three tabs: General, Themes, and Extensions. You can move between the tabs by clicking on them, or by pressing `g` for the General panel, `t` for the theme panel, and `e` for the extension panel. You can also scroll up and down the panel using `j` and `k`.

The General tab is shown when the Preferences Panel is opened. Here, you can add/delete environment variables that will be created for all subtasks ran, including the User Scripts extension. Therefore, if the Modal File Manager can’t find a program, check this panel’s `PATH` setting and edit it as needed.

![Pref-General2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624432282340/IKMOupCv8.png)

After the environment variable section is the check box for using the system trashcan or not. In order for this to be checked, you should have the [trashcan cli](https://github.com/andreafrancia/trash-cli) program installed.

Also, the `Exit Preferences` button at the bottom will leave the Preference Panel. Any changes made are recorded and saved as you make them. Therefore, exiting will not go back to the values before you changed them.

The Theme panel allows you to change the current colors of the various aspects of the program along with font values. 

![Pref-Theme1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624432304155/EDDiRUEEe.png)

By clicking on a color box, a color picker dialog will open allowing you to change the color. As you change the sliders in the color picker, that change will immediately be shown in the program.

![Pref-theme3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624432321836/AWu_m7veF.png)

Once you change the colors, you can to go the end of the various color selectors and save the current theme setting to a theme folder. You enter the name and press `Save New Theme` button. The theme will be saved and listed at the bottom.

![Pref-Theme2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624432311996/eeCUAfFO6.png)

The last thing in the Theme panel is a list of downloaded or created themes. You can set one of the themes to be current, update the theme with the new values assigned above, or you can delete the theme from your computer.

The Extensions panel shows a list of extensions and give you the option to `Edit` or `Delete` the extension. If you press edit, the current panel will be opened to the extension directory and the main file for the extension will be opened in the editor you have setup. If you press `Delete`, then the extension will be deleted from your computer.

![Pref-Extensions.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624432255726/MpHJqD74n.png)

At the bottom, you can put in a name for a new extension and press `Create New Extension` button. A directory with the extension name, a `package.json` file for the extension, and the main extension file will be created and the extension file will be opened in the editor you setup for the Modal File Manager. My next tutorial will explain how to write extensions.

## Conclusions

These are the current additions made to the Modal File Manager. If you haven’t downloaded the program yet, please do and enjoy this flexable file manager. I use it everyday. 

If you have any suggestions or constructive feedback, just create a note in the [discussions section](https://github.com/raguay/ModalFileManager/discussions) of the Modal File Manager GitHub repo.

Above all, have fun!