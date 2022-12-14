## My Side Project: Modal File Manager

One of the key programs for a developer is the file manager. I have many on my computer because I can’t seem to find the one that works best for me. Therefore, I decided to work on my own using  [Svelte](https://svelte.dev/docs)  and  [NW.js](https://nwjs.io/). I thought I would update my blog on it’s progress and features. There will be more coming as I develop the program. You can preview it on my [Modal File Manager repository on GitHub](https://github.com/raguay/ModalFileManager).

## History

I started my programming career with Emacs as my main text editor on a main frame computer 
in college. I really loved Emacs, but had a very hard time remembering all the key commands. 
I ended up using Vim once for a class I had (the teacher loved Vim) and hit the perverbial Vim 
wall -- How do I exit this thing! I finally had to kill the process. So, I went back to Emacs.

With Spacemacs and Doom-emacs, I learned that the Vim style keyboard was much more efficient. But, I kept going back to the Emacs key memory. I finally decided to really give Vim a run with neovim on my MacBook Air. I even installed Spacevim and really liked it. Now, I'm fully comfortable in the Vim style keyboard (but far from mastering it completely) and have adapted it to many other programs along the way. My current editor of choice is [OniVim2](https://github.com/onivim/oni2) which is the fastest programming editor I've ever used. It is great!

But, I've never been happy with the file managers I've used. [fman](https://fman.io/) is great, but not actively being maintained and expanded upon. I also don't really like python, it's API language. So, I decided to jump in and make one to suit me better. This is how Modal File Manager was started. I wanted something that was as configurable as TkDesk was, but with a modal keyboard model for hotkeys. To take it even further, new modes and keymaps can be added with extensions. Along with anything else you can do with a full Node.js backend.

Since there are so many dual pane file managers available, I knew this would never be a marketable product (and I did not want the hassle of endless customer complaint over a feature not working the way they want it). Therefore, I'm making this an open source project to hopefully get some help from other to really making this thing shine.

Therefore, I hope you enjoy this little program as much as I have. Feel free to sponor the project or just give me some tips along the way. Any help is appriciated.

I've created multiple open source resources (I have around 80 GitHub repositories), but this is the first full product. Therefore, please bear with me as I learn about maintaining and supporting an open source project.  

## Current Feature Set

- Dual pane file manager with vim style model hotkeys for navigation and action launching.
- Command bar for executing commands (`:` in any mode or `<ctrl>p` in normal mode). It also shows a description of what each command does.
- Fully extendable with extensions using the extensions API
- Fully extendable to alternate file systems
- Fully theme-able
- A file details side panel can be shown to overlay the panel not currently focused (`toggleExtraPanel` command or `s` in normal mode). It shows video previews and stats with the ffmpeg programs installed.
- Themes and extensions are explorable and downloadable from GitHub inside the program.
- Hot keys are programmable
- A number before a hot key runs it's associated command that many times (ex: `5j` will move the cursor 5 entries down the list).
- Watches for changes in the current directory and updates accordingly.
- Changing directories in the Directory Bar (normal mode `q`) shows a list of matching entries from history and then below the current directory.
- File editor is configurable by the `~/.myeditorchoice` file (see `Editing Files` below)
- Integrated with ScriptPad - another side project that isn’t ready for peer review.
- Quick Search - an input to type text so that any entry at that level is removed that doesn't have that text in it. Just refresh the pane to get back to normal. I think of it as a quick filter more than a quick search.


## Installation

I have a run script made with [Mask](https://github.com/jakedeichert/mask) and [Node.js](https://nodejs.org/en/). You have to put a copy of [NW.js](https://nwjs.io/) in the 'misc' directory as `nwjs.app`. Or, you can change the script. I'm assuming you aren't changing the scirpt in the following.

So, download the repository, create the `misc` directory, put nwjs.app in the `misc` directory. On the command line, run the following commands to accomplish these things:

```sh 
git clone https://github.com/raguay/ModalFileManager.git 
cd ModalFileManager
mkdir misc
npm install 
``` 

This will install all the libraries for building the project. Then run:

```sh 
mask build 
```

which will build the project. To run the project, run this command:

```sh 
mask launchapp
``` 

If everything is in place okay, it should then show you the file manager opened to your 
home directory. Dive in and have fun.

If you are interested in running in development mode, you have to have the NW.js SDK in the 
`misc` directory named `nwjs-sdk`. Then you can run:

```sh 
mask launchdev
``` 

which will launch the sdk version of NW.js with full development tools.

I'm working on making actual releases, but I'm having issues with getting it right on 
the macOS. You should be able to run it on Windows and Linux, but I haven't tested it 
on those platforms. Actually, since I'm using mostly command line commands to perform actions, 
running on Windows will most likely not work currently. But, I to plan on fleshing it out soon. 
I currently only have a Mac and do all my work on it. 

## Command Line Programs Used

There are a few non-standard command line programs I use with Modal File Manager. They are:

- [ffmpeg](https://ffmpeg.org/) for getting and using video information in the Extra Panel.
- [fd](https://github.com/sharkdp/fd) for quick file finding. It's a `find` replacement written in Rust.
- I also use the standard cp, mv, and rm commands on the command line. These still run faster than rewriting them in the scripting language. The major drawback is there isn't a backup method. Once deleted, always deleted. `Trashcan` support will come later.

All of the programs should be downloaded and in your shells path. Modal File Manager doesn't assume location for anything except or it's own configuration files. 

## Configuration Files

All extensions, themes, keyboard layouts, and anything else for configuring the Modal File Manager is found in it's main configuration directory. On Linux and macOS, it is located at: `~/.config/modalfilemanager`. It will be in the Windows User directory, but I haven't really looked at Windows functionality yet.

In this configuration directory, there are the `themes`, `extensions`, and `keyMaps` directories that contain their respective subfolders and files. Please refer the specific section for each directory for more details.

There is the `history.json` file and the `theme.json` file. The `history.json` file contains a list of directories that the Modal File Manager has visited. I use this to quickly pull up possible paths to go to in the file manager. 

Modal File Manager doesn't use the actual theme files downloaded from GitHub. Those are stored in the `themes` directory and are just referenced. The theme values that are actually used are in the `theme.json` file. When a user changes themes, that file is changed. Therefore, be careful if you manually change this file and want to keep it. It is best to create a theme in the theme directory and load it in the program.

## Source File Layout

The directory structure for the source files is:

### `src`
- components: Here are all the Svelte components for the UI
- modules: These are JavaScript Helper files with the data structures used.
- stores: This directory contains all the Svelte Store items
- FileManager.svelte: This is the main program
- main.js: This installs the main program into the HTML

All low level functions are in the `modules/macOS.js`, `modules/linux.js`, and `modules/windows.js` for the particular operating system. This is the initial breakdown and will be added upon in the future as needed.

## Editing files

Files will be edited (normal mode key `e`) using editor specified in the [xBar](https://xbarapp.com/) plugin [currentFiles.1h.rb](https://xbarapp.com/plugins/System/currentFiles.1h.rb).

To use with the **xBar** plugin, you will need to have [xBar](https://xbarapp.com/) installed and the [currentFiles.1h.rb](https://xbarapp.com/docs/plugins/System/currentFiles.1h.rb.html) plugin installed and configured. You can also use the [Alfred BitBar Workflow](https://github.com/raguay/MyAlfred/blob/master/Alfred%203/BitBarWorkflow.alfredworkflow) to control the plugin.

Alternatively, you can use the [TextBar](http://richsomerfield.com/apps/textbar/) program with the [Current Files and Editor](https://github.com/raguay/TextBarScripts/blob/master/Current%20Files%20and%20Editor.textbar) plugin installed. You can use the [Alfred](https://www.alfredapp.com/) with the [My Editor Workflow](https://github.com/raguay/MyAlfred/blob/master/Alfred%203/My%20Editor%20Workflow.alfredworkflow) to control the editor and edit files.

If the above editor setup isn't on the system, it will use the operating system to open the 
file in the default file editor.

## Things in the Works

- Theme Creator/Editor
- Extension Creator/Editor with templates.
- Add and test dynamically added states for keyboard
- Translating my fman extensions to work with Modal File Manager
	- Project Manager
		- Notes - Should be part of Project Manager?
	- Favorites
	- Dropbox File System
	- Launch Scripts
	- Regular Expression selection
	- Zip Selected Entries
- Internal drag and drop - I can't seem to get it to work right.
- Proper drag and drop with outside programs (NW.js doesn't support it directly, but I should be able to get it going with Node.js talking directly to the OS). 
- Reloading extensions without relaunching
- Add more file views for the Extra Panel
- Multiple windows
- Create an icon for the program
- Create a proper macOS application
- Get Windows working
- Get Linux tested and working
- Documentation!

## Default Key Bindings

### Normal Mode

| Key Press | Command Executed |
| --- | ------ |
| `r` | reloadPane |
| `p` | swapPanels |
| `d` | duplicateEntry |
| `e` | editEntry |
| `m` | moveEntries |
| `c` | copyEntries |
| `x` | deleteEntries |
| `g` | goTopFile |
| `G` | goBottomFile |
| `ArrowDown` | moveCursorDown |
| `ArrowUp` | moveCursorUp |
| `l` | goDownDir |
| `h` | goUpDir |
| `Enter` | actionEntry |
| `Tab` | cursorToNextPane |
| `k` | moveCursorUp |
| `j` | moveCursorDown |
| `i` | changeModeInsert |
| `v` | changeModeVisual |
| `/` | toggleQuickSearch |
| `s` | toggleExtraPanel |
| `<cmd>p` | toggleCommandPrompt |
| `:` | toggleCommandPrompt |
| `.` | reRunLastCommand |

### Visual Mode

| Key Press | Command Executed |
| --- | ------ |
| `Escape` | changeModeNormal |
| `k` | moveCursorUpWithSelect |
| `j` | moveCursorDownWithSelect |
| `ArrowDown` | moveCursorDown |
| `ArrowUp` | moveCursorUp |
| `:` | toggleCommandPrompt |

### Insert Mode

| Key Press | Command Executed |
| --- | ------ |
| `Escape` | changeModeNormal |
| `d` | newDirectory |
| `f` | newFile |
| `r` | renameEntry |
| `:` | toggleCommandPrompt |


## Command Prompt Commands

These commands can be ran from the command prompt. They all act upon the current cursor.

| Command Name                      | Command Description                                                        | Command Function Name    | 
|:--------------------------------- |:-------------------------------------------------------------------------- | ------------------------ | 
| `Move Cursor Up`                  | This will move the cursor up one line                                    | moveCursorUp             |
| `Move Cursor Up with Selection`   | This will move select the current entry and move the cursor up one line. | moveCursorUpWithSelect   |     |
| `Change Mode to Normal`           | Set the normal mode.                                                     | changeModeNormal         |     |
| `Change Mode to Insert`           | Set the insert mode.                                                     | changeModeInsert         |     |
| `Change Mode to Visual`           | Set the visual mode.                                                     | changeModeVisual         |     |
| `Cursor to Next Pane`             | This will move the cursore to the opposite pane.                         | cursorToNextPane         |     |
| `Action Entry`                    | This will open a file or go into a directory.                            | actionEntry              |     |
| `Go Up a Directory`               | Go to the parent directory.                                              | goUpDir                  |     |
| `God Down a Directory`            | If the current entry is a directory go to it.                            | goDownDir                |     |
| `Go to Bottom File`               | Move the cursor to the bottom most file.                                 | goBottomFile             |     |
| `Go to Top File`                  | Move the cursor to the top most file.                                    | goTopFile                |     |
| `Delete Entries`                  | Delete all selected entries or the one under the cursor                  | deleteEntries            |     |
| `Copy Entries`                    | Copy the selected entries or the one under the cursor to the other pane. | copyEntries              |     |
| `Move Entries`                    | Move the selected entries or the one under the cursor to the other pane. | moveEntries              |     |
| `Edit Entry`                      | Opens the file under the cursor in the editor specified.                 | editEntry                |     |
| `Duplicate Entry`                 | Make a copy of the current entry with "\_copy" added to it.              | duplicateEntry           |     |
| `New File`                        | Create a new file in the current pane.                                   | newFile                  |     |
| `New Directory`                   | Create a new directory in the current pane.                              | newDirectory             |     |
| `Rename Entry`                    | Rename the current entry.                                                | renameEntry              |     |
| `Swap Panels`                     | Swap the panel contents.                                                 | swapPanels               |     |
| `Toggle Quick Search`             | Show/Hide the Quick Search panel.                                        | toggleQuickSearch        |     |
| `Reload Pane`                     | Reload the Current Pane.                                                 | reloadPane               |     |
| `Toggle Extra Panel`          | Show/Hide the extra panel. | toggleExtraPanel |
| `Edit Directory` | Open the Edit Directory for the current panel.| editDirectory |
| `Toggle Command Prompt` | Show/Hide the command prompt. | toggleCommandPrompt |
| `Toggle GitHub Importer` | Show/Hide the GitHub importer panel for searching for themes and extensions on GitHub and installing them. | toggleGitHub |
| `Refresh Panes` | This will reload files in both the left and right pane. | refreshPanes |
| `Refresh Left Pane` | This will reload the files in the Left Pane. | refreshLeftPane  |
| `Refresh Right Pane` | This will reload the files in the Right Pane. | refreshRightPane |
| `Rerun Last Command` | This will rerun the last command along with it the number of times it was ran. | reRunLastCommand |

### Extension Commands

These commands require inputs and supply results. Therefore these commands can`t be used in 
hotkeys or command prompt. They are loaded and used in a different way in an extension and internally in the main program itself.

| Function Name | Description |
| --- | ---------- |
| `setCursor` | Set the cursor to the file name given in the current panel. |
| `getCursor` | Get the current cursor. |
| `cursorToPane` | Set the cursor to the pane given. Either "left" or "right" |
| `changeDir` | Change the directory of a pane and make it the current. |
| `addKeyboardShort` | A a shortcut for a keyboard map. |
| `getLeftFile` | Get the current file for the left pane. |
| `getRightFile` | Get the current file for the right pane. |
| `getTheme` | Get the current values for the theme. |
| `setTheme` | Set the values for the current theme. |
| `getOS` | Get the local OS name. |
| `addDirectoryListener` | Add a listener to directory changes. |
| `getLastError`   | returns the last error. |
| `getSelectedFiles` | Returns a list of Entries that have been selected. |
| `getCurrentFile`   | Get the current file. |
| `getCurrentPane` | Get the pane that is currently active. |
| `addSpinner` | Add a message box spinner value. |
| `updateSpinner` | Update a message box spinner value. | 
| `removeSpinner` | Remove a message box spinner value. | 
| `keyProcessor` | 'Send a keystroke to be processed.' | 
| `stringKeyProcessor` | 'Send a string of keystrokes to be processed.' | 

## Creating Themes

A theme is a GitHub repository or a repository on your system. It is setup as a normal npm 
project with a `package.json` file. The `package.json` file should be similar to this:

```json 
{
  "name": "dracula-ThemeModalFileManager",
  "version": "1.0.0",
  "description": "The Dracula theme for Modal File Manager.",
  "keywords": [
    "modalfilemanager"
  ],
  "author": "Richard Guay",
  "license": "MIT",
  "mfmtheme": {
    "name": "dracula-ThemeModalFileManager",
    "description": "Dracula theme for Modal File Manager.",
    "type": 0,
    "github": "",
    "main": "dracula.json"
  }
}
```

The subheading `mfmtheme` contains the information the program will use to load the 
theme. The `main` is set to the path of the JSON file containing the themes values. If 
it's in the top of the directory, then use just the file name as seen above.

The `type` is 0 for local only and 1 for a GitHub download. The `github` value is the 
URL to the repository on GitHub. The `description` is shown to the user that should 
accurately describe the theme.

The Theme JSON file is like this one:

```json 
{
  "font":"Fira Code, Menlo",
  "fontSize":"12pt",
  "cursorColor": "#363443",
  "selectedColor": "#FFCA80",
  "backgroundColor": "#22212C",
  "textColor": "#F8F8F2",
  "borderColor": "#1B1A23",
  "normalbackgroundColor": "#80FFEA",
  "insertbackgroundColor": "#8AFF80",
  "visualbackgroundColor": "#FF80BF",
  "Cyan": "#80FFEA",
  "Green": "#8AFF80",
  "Orange": "#FFCA80",
  "Pink": "#FF80BF",
  "Purple": "#9580FF",
  "Red": "#FF9580",
  "Yellow": "#FFFF80"
}

```

It should be a proper JSON structure with these definitions. Change the color values 
as you want.

## Adding Video Preview on Extra Panel

The normal copy of NW.js doesn't come with the codeces for displaying videos. In order 
for this feature, you will have to download the dynamically loaded library for your system 
from here:  https://github.com/iteufel/nwjs-ffmpeg-prebuilt/releases.

Then place this in your copy of NW.js as described in step 5 here:  http://docs.nwjs.io/en/latest/For%20Developers/Enable%20Proprietary%20Codecs/#enable-proprietary-codecs-in-nwjs

With this library in place, the Extra Panel will run mp4 files just fine. I haven't testing 
other versions since that what I mostly use.

## Creating Extensions

I haven’t finalized or worked up the documentation for the extensions API. I have been making extensions and testing some of the features, but it’s not documented at the moment. I’ll have another tutorial on it in the future. 

## Conclusion

So, you can go and give it a try. I’d love some feedback.
