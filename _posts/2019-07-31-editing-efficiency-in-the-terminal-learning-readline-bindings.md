---
layout: post
title: "Editing Efficiency in the Terminal: Learning Readline Bindings"
author: zeroalpha
date: 2019-07-31
categories: Guide
---

Everyone has their favorite editor, and some would fight to the death to defend their editor.
Editors are also a common topic of blog posts -- how to use a specific editor, how to configure it, what plugins to use, etc.
People mention Vim, Emacs, Atom, Sublime Text, VS Code...but nobody ever talks about the editor that you use the most on the command line -- the shell prompt.
In my opinion, the shell prompt is actually quite a powerful editor, and I hope this post will serve to convince you.

Shells like Bash and Sh use a library called 'Readline' to handle their input.
Zsh has its own implementation called 'ZLE', which works similarly to Readline.
Therefore, learning to use Readline properly and efficiently can, and probably will, speed up your workflow.

This is not a beginner-level guide, so you should be familiar with the basics of adding text on the command line, such as tab completion.
This post will introduce more advanced editing techniques to hopefully make you faster and more efficient.
I will mostly be referring to Bash, and I will highlight the differences in ZSH, because those are the shells that I work with.
However, they should work in all shells that use the Readline library.
Also, for those of you who know about Readline editing modes, I will be using Emacs mode, because most of the time it's the default.

The notation I will use is `Ctrl` for the control key, and `Meta` for the meta key.
The meta key is commonly the alt key, but I recommend googling how the meta key works for your particular OS and terminal.

## Movement
Here's how you move around with more efficiency:
* Line-by-line:
    * `Ctrl-a`: move to the beginning of the line
    * `Ctrl-e`: move to the end of the line
* Word-by-word:
    * `Meta-f`: move one word forward
    * `Meta-b`: move one word backward
    * Both of these take numeric arguments, see the gif demo below.
* Character-by-character:
    * `Ctrl-f`: move one character forward
    * `Ctrl-b`: move one character backward
* Searching:
    * `Ctrl-]`: search forwards for a character and jump to it (like `f` in Vim)
    * `Ctrl-Meta-]`: search backwards for a character and jump to it (like `F` in Vim)

Searching is not set up by default in ZSH.
Please see the [ZSH Specifics](#jump-to-a-character) section for information on how to set it up.

To clear the screen, use `Ctrl-l`.

![Movement demo](/img/readline-demos/movement.gif)

## History
One way to avoid re-typing commands is by using your history.

To search your history, press `Ctrl-r` and type a string.
Then, use `Ctrl-p` to go to older commands containing the search string, and `Ctrl-n` to go to newer ones.
You can use `Ctrl-s` to search forward in your history (but I don't use this as much since I usually search for commands that I typed previously).

You can also type a command like "ls", and then press `Meta-p` and `Meta-n` repeatedly to cycle through previous and next commands from your history that contain "ls".

Another useful feature is reusing arguments of previous commands.
You're probably familiar with expansions: `!!` for the previous command, `!$` for the last argument of the previous command, and `!:n-m` for arguments from the one at position n up to and including the one at position m.
Readline also offers key bindings for arguments, and these are often more convenient to use than expansions.

To insert the last argument of the previous command, type `Meta-.`.
Then you can press `Meta-.` repeatedly to cycle through all previous arguments.

![Inserting args demo](/img/readline-demos/inserting args.gif)

You can also press the two key combinations `Meta-NUM Ctrl-Meta-y` to insert the previous argument at position NUM (with NUM being a number). The argument at position 0 is the command name.
This particular key binding doesn't work in ZSH by default, please see the [ZSH Specifics](#insert-an-argument-from-the-previous-command) section for information on how to set it up.

![Numerically insert arg](/img/readline-demos/num arg insert.gif)

Finally, there's the undo command.
You can press `Ctrl-_` repeatedly to undo all the changes you made to a line.

![Undo demo](/img/readline-demos/undo.gif)

## Editing
Readline also offers a bunch of useful key bindings for more efficient editing.

To delete:
* Line-by-line:
    * `Ctrl-k`: kill (delete) from the cursor to the end of the line
    * `Ctrl-u`: kill (delete) from the cursor to the end of the line
    * ZSH also offers a way to delete the entire line with one key binding, please see the [ZSH Specifics](#kill-a-whole-line) section for information.
* Word-by-word:
    * `Meta-d`: delete from the cursor to the end of the word
    * `Ctrl-w`: delete from the cursor to the start of the previous space-delimited word
    * `Meta-Delete`: delete from from the cursor to the start of the previous word (`Delete` is the backspace key)
        * The difference between `Ctrl-w` and `Meta-Delete` is easier to explain with an example. Let's say you have the string `/some/path/here`, with your cursor after the word `here`. If you type `Ctrl-w`, you'll delete the whole path. `Meta-Delete` would delete individual components of the path, so typing `Meta-Delete` would delete the word 'here'.
* Character-by-character:
    * `Ctrl-d`: delete the character under the cursor

If you 'kill' (delete) some text, you can paste ('yank') it into some other line with `Ctrl-y`.
You can cycle through everything you previously deleted with `Meta-y`.

You can also switch ('transpose') the last two words in your command with `Meta-t`, which is useful if you, for example, type the paths in a `mv` command in the wrong order.

![Transposing demo](/img/readline-demos/transposing.gif)

If you have a lot of whitespace around your cursor, you can use `Meta-\` to delete it.
This doesn't work in ZSH by default, and I haven't found a way to set it up yet.

![Whitespace demo](/img/readline-demos/delete whitespace.gif)

Another one I use quite often is `Meta-#`, which comments the current line and starts a new one.
You can remember it easily due to the fact that Bash comments start with '#'.
There's a command for this in ZSH, but you need to set a key binding, so see the [ZSH Specifics](#comment-out-the-current-line) section.

![Commenting demo](/img/readline-demos/commenting.gif)

Finally, if you're editing a really long command, you might want a full-featured editor.
You can open the current line in your $EDITOR with `Ctrl-x Ctrl-e`, which often pops up Vim, Nano, or Emacs.
When you save the file and quit the editor, it'll run the command that you edited.

![Open in editor demo](/img/readline-demos/open in editor.gif)

## Macros
Yes, Bash has built-in macro functionality, where you can record a series of key strokes and then play them back whenever you want.
Type `Ctrl-x (` to start recording a macro, and `Ctrl-x )` to stop recording a macro.
Then, type `Ctrl-x e` to execute the macro that you just recorded.

This works in Bash and other shells that use Readline, but I haven't found a way to make it work in ZSH yet.

![Macro demo](/img/readline-demos/macros.gif)

# ZSH specifics
## Kill a whole line
ZSH gives you the key binding `Ctrl-x Ctrl-k` to kill the entire line.

## Jump to a character
You have to bind a key to the vi-find-next-char and vi-find-prev-char functions.

Put this in your `.zshrc`:

```zsh
bindkey '^]' vi-find-next-char
bindkey '^\e]' vi-find-prev-char
```

Now you can use the same Bash bindings to jump to a character, forwards and backwards.

## Insert an argument from the previous command
For this to work, you have to define your own ZSH 'widget', and then bind a key to it.

Put this in your `.zshrc`:

```zsh
# Define the function
insert-arg-of-prev-cmd() {
    # Get the argument
    : ${NUMERIC:-1}
    (( NUMERIC++ ))

    # Get the previous command
    words=($(fc -ln -1))

    # Extract the argument
    RBUFFER+="$words[$NUMERIC] "

    # Move the cursor to the end of the line
    zle end-of-line
}

# Create a ZLE widget
zle -N insert-arg-of-prev-cmd

# Bind 'Ctrl-Meta-y' to it
bindkey "\e^y" insert-arg-of-prev-cmd
```

Now you can use the same bindings as in Bash to insert a specific argument of a previous command.

## Comment out the current line
There's already a function for this in ZSH, but you need to bind a key to it.

Put this in your `.zshrc`:

```zsh
bindkey '\e#' pound-insert
```

Now you can use the Bash-style key binding.
