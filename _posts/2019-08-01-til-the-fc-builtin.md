---
layout: post
title: "TIL: the fc builtin"
author: zeroalpha
date: 2019-08-01
categories: TIL
---

The `fc` command is a shell builtin that lets you work with commands that you previously typed in, and stands for "fix command".
It has two forms: in the first, you can interactively edit a range of commands, while in the second, you can re-run a previous command after doing a replacement.

## First form: `fc` to interactively edit a range of previous commands
This lets you take a range of previous commands and open them in an editor.
Then, when you save the file and quit the editor, whatever text you edited is executed.

Usage: `fc first_command_number last_command_number`.
This will open all commands from first_command_number up to and including last_command_number in an editor.
If last_command_number isn't specified, the current command is used.
The command numbers can be relative, such as '-2' for 'two commands ago'.

Flags:
* `-e ename`: ename is the editor. If not given, $FCEDIT is used. If not set, $EDITOR is used.
* `-l`: list commands on standard output, don't open an editor
* `-n`: don't print command numbers
* `-r`: reverse the order of commands

<video width="100%" style="margin-bottom: 1em;" autoplay controls loop playsinline muted>
    <source type="video/mp4" src="/video/fc-demos/edit.mp4" />
    <p>Demo of the editing form</p>
    </video>


## Second form: `fc -s` to replace text in a previous command and re-run it
In this form, you can do a global substitution (text replacement) across a previous command and re-execute it.

Usage: `fc -s pattern=replacement command_number`.
The command number is the same as in the first form.
If it's not specified, the previous command is used.

In ZSH, this second form is a bit different.
To get the same behavior as in Bash, use `fc -e - pattern=replacement`.
You don't need the `-s` flag, and `-e -` tells ZSH to skip the editor.

<video width="100%" style="margin-bottom: 1em;" autoplay controls loop playsinline muted>
    <source type="video/mp4" src="/video/fc-demos/substitute.mp4" />
    <p>Demo of the substitute form</p>
</video>


