---
title: "Sending Emails from Command Line using EmailIt"
datePublished: Thu Jul 27 2023 05:53:50 GMT+0000 (Coordinated Universal Time)
cuid: clkkqoiz400040ai715kq3oml
slug: sending-emails-from-command-line-using-emailit
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1690437077331/c88c3897-ba6e-4081-9afc-26c1aad7ca9c.gif
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1690437204069/6a4826a1-5050-478e-be9c-d6e922315946.gif
tags: go, javascript, svelte, wails, bubble-tea

---

I just recently released an update to the [EmailIt](https://GitHub.com/raguay/EmailIt) that includes many great new features:

* A TUI application for sending emails from the command line
    
* The body text of all emails are processed through the builtin [Handlebars](/handlebarsjs.com/) parser to expand macros.
    
* Removed Script Terminal for Scriptline.
    
* Added many new API endpoints to use to control EmailIt.
    
* Many great bug fixes and updates overall.
    

But, Let's just look at the command line EmailIt TUI.

## TUI

The word TUI comes from Terminal User Interface. It is basically creating a graphical user interface in a terminal. The EmailIt program is both a graphic interface and a command line TUI. First download the program from the GitHub repo: [https://github.com/raguay/EmailIt](https://github.com/raguay/EmailIt). Unzip it and move it to the `/Applications` folder.

In order to use the program from the command line, you need to make an alias to the programs executable file for the macOS:

```bash
alias em=/Applications/EmailIt.app/Contents/MacOS/EmailIt
```

With the above line in your `.bashrc` or `.zshrc` file (you will have to make a similar line for other shells), you can run the EmailIt program from any directory. The different command line options are shown with the basic `-h` or `--help` flag or the `help` or `h` command:

```bash
> em help
NAME:
   EmailIt - A program for sending emails, working with text, and scripts.

USAGE:
   EmailIt [global options] command [command options] [arguments...]

VERSION:
   v2.1.0

AUTHOR:
   Richard Guay <raguay@customct.com>

COMMANDS:
   mkemail, me     Create an email using a TUI
   notes, n        Open the notes.
   emailit, e      Open the EmailIt email sending application.
   scriptline, sl  Open the ScriptLine application.
   sendemail, se   Send the email directly. No GUI or TUI.
   help, h         Shows a list of commands or help for one command

GLOBAL OPTIONS:
   -a value       Address to send an email
   -s value       Subject for the email
   -b value       Body of the email
   --help, -h     show help
   --version, -v  print the version

COPYRIGHT:
   (c) 2022 Richard Guay
```

The `-a` flag will take the `value` and put it for the address to which to email. The `-s` flage will take the `value` and place it in the `Subject` field of the email. The `-b` flag will take the `value` and place it in the body of the email. The `mkemail` or `me` command will then open the TUI for creating the email. The `sendemail` or `se` command will take the command line information and send the email directly without using the TUI. The `notes` or `n` command will open EmailIt to the notes screen. The `scriptline` or 'sl' command will open the Scriptline screen. The `emailit` or `em` command will open the EmailIt screen.

The TUI looks like this:

![EmailIt TUI](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qkjzcropfreg9ie9eqsh.gif align="left")

If you have the Default account set up in EmailIt, it will then send your email to that account. The email resulting from the above video is:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690438503674/23765f62-8a63-44bd-a9b7-50e5828efca3.png align="center")

You're all set to do emails from the command line!