---
layout: post
title: "Learning Readline"
author: zeroalpha
---

Bash:
* Movement
    * Ctrl-a/e
    * Ctrl-f/b
    * Alt-f/b
    * Ctrl-], Ctrl-Alt-] -- search forwards/backwards for a character and jump to it (like f/F in Vim)
        * in zsh, bind vi-find-next-char and vi-find-prev-char
* Ctrl-l
* History:
    * Ctrl-r
    * Ctrl-p/n -- previous/next command
    * Alt-p/n -- search history for string
    * Expansions:
        * !! -- previous command
        * !$ -- last argument of previous command
        * !:n-m -- arguments n to m of previous command, starting at 1 (so !:1-3 would insert the first, second, and third arguments of the previous command)
            * !:-n -- arguments up to the nth argument of the previous command
            * !:n- -- arguments from nth argument of the previous command until the end
    * Alt-. -- insert last argument of previous command
    * Alt-NUM Ctrl-alt-Y -- insert NUM argument of previous command
        * zsh: Alt-. with a negative argument. yank-nth-arg?
    * Ctrl-_ -- undo
* Editing:
    * Ctrl-d -- delete character under cursor
    * Alt-d -- delete from cursor to end of word
    * Ctrl-w -- delete from cursor to start of previous big word (big words are delimited by spaces)
    * Alt-Delete -- delete from cursor to start of previous word
        * example: in /some/path/here, ctrl-w would delete the whole path, and alt-delete would delete individual components of the path
        * in zsh, works the same as ctrl-w
    * Ctrl-k -- kill (delete) from cursor to end of line
    * Ctrl-u -- kill (delete) from cursor to start of line
    * Alt-t -- transpose (switch) the last two words
    * Alt-\ -- delete excess whitespace around cursor
        * bash only
    * Alt-# -- comment the line and start a new one
        * in zsh, can enable with binding.
    * Ctrl-y -- paste ("yank") the most recently killed (deleted) text
    * Alt-y -- cycle through the previously killed (deleted) text
    * Ctrl-x Ctrl-e -- open the current line in $EDITOR
* Macros (bash only):
    * Ctrl-x ( -- start recording a macro
    * Ctrl-x ) -- stop recording a macro
    * Ctrl-x e -- execute the recorded macro


zsh:
    * Ctrl-x Ctrl-k -- kill whole line
