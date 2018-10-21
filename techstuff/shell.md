---
title: Git stuff
---

# Keyboard shortcuts
* `C-x C-e` — edit command in $EDITOR (vim for me)
* `C-a` — go to start of command
* `C-e` — go to end of command
* `Alt-f` — go forward by word
* `Alt-b` — go backward by word

# Directory stack
* `pushd $dir` — push cwd to stack, cd to dir
* `popd` — pop the directory from the stack and cd to it
* `dirs` — list directory stack

# `exec` builtin
usage:

* **with a command:** completely replaces shell by program. e.g. `exec vim` replaces the shell with vim, when you close vim you quit the terminal.
* **with a redirection:** makes the redirection apply to the current shell

examples:

```bash
exec 1> >(lolcat >&2)   # make shell redirect stdout to process lolcat, 
                        # which redirects to stderr
```

```
exec 3< input.txt        # create a new file descriptor, reading from input.txt
read LINE <&3            # read from the file descriptor into LINE
exec 3<&-                # close the file descriptor
```

```bash
exec 4> output.log       # create a new file descriptor for writing to output.log
echo “foo” >&4           # write “foo” to file descriptor 4 (output.log)
exec 4>&-                # close the file descriptor
echo “foo bar” > file    # write “foo bar” to file
exec 5<>file             # open file for rw on file descriptor 5
read -n 3 var <&5        # read 3 characters from file descriptor 5 into var
echo "Hello" >&5         # write "Hello" into the file
exec 5>&-                # close the file for writing
exec 5<&-                # close the file for reading
```

# Change keybindings
* `set -o vi` — vim keybindings for shell
* `set -o emacs` — emacs keybindings for shell

# Misc sets
* `set -e` — exit script on error
* `set -x` — print code as it executes

# Variable expansion
* `${var##*/}` — remove longest string of anything followed by / from var
* `${var#*/}` — same but shortest
* `${var%%*/}` — same as above but from end

# Reusing previously typed stuff
* `!!` — previous command
* `!$` — last argument of previous command
* `!^` — first argument of previous command
* `!*` — all arguments of previous command
* `!:a-b` — arguments number a to b of previous command
* `!$:h` — only the head of the previous argument (e.g. “~/Documents” in “~/Documents/file.txt”)
* `^x^y^` — replace x by y

# Redirection
* `&> file` — redirect all output to a file
* `(1)>&2` — redirect stdout to stderr, the 1 is implicit
* `2>&1` — redirect stderr to stdout
* `>file` — redirect stdout to file, overwriting
* `>>file` — redirect stdout to file, appending
* `<(command)` — pass output of command as file
* `>(command)` — redirect stdout to command
* `<<<` — here string/document, pass from stdin as input

order matters, creating a new redirection essentially duplicates the file descriptor (e.g. “command > file 2>&1” != “command 2>&1 > file"