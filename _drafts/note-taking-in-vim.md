---
layout: post
title: "How I take notes in Vim with Vimwiki"
author: zeroalpha
---
In this post, I'll describe how I take notes in lectures using Vim.
I am a university computer science student, so I often have to take notes on subjects with mathematical symbols and/or code.
I've found Vim with the Vimwiki plugin to be a perfect fit for me, as it allows me to quickly edit notes and then export them to HTML.

Disclaimer: this is just my setup, and I'm not posting this for someone else to blindly take what I'm using.
I am posting this so that others may perhaps be inspired by my setup, as I have been inspired by posts of others.

# Taking notes
I create a single page per subject while I'm working on my notes.
So for example, the page on Equational Programming that I'm currently working on would be called "Equational Programming.txt".
I don't keep my Vimwiki in version control, because I don't want to bother with commits for every small change.
Furthermore, I have persistent undo set up in Vim, so I can go back to a previous version of my notes at any point anyway.
Instead, I keep it in Dropbox, and I have Vimwiki configured with a registered wiki in Dropbox, with the extension '.txt'.
This means that my notes are visible on any device.

Every page of notes starts with these two lines:

```markdown
%% vim: nospell spelllang=en spellcapcheck=
%template math
```

The first line is called the 'modeline', and sets file-specific options in Vim.
This specific line means:
* `nospell`: disable spellcheck (for now)
* `spelllang=en`: set the language to English. I use both American and British spelling, so I want Vim to accept both. If you want only American English, you can set it to `en_US`.
* `spellcapcheck=`: disable checking for capital letters. My notes are quite free-form and I tend not to worry about capitalization.

The second line tells Vimwiki to use the 'math' template while generating HTML (more about this in a later section).

Then, I start writing.
To enter code, I use Vimwiki's preformatted text markdown, along with a class attribute.
The general layout looks like this:

```markdown
{{{class="code {language}"
// code here
}}}
```

For example, for Haskell, I'd write something like this:

```markdown
{{{class="code haskell"
head [1,2]
}}}
```

I have the HTML generation template add a JavaScript syntax highlighting library, which uses the class attribute to determine what to highlight and which language to use (more on this in a later section).

To enter special characters, I use Vim's digraphs feature (`:help digraphs`).
You can see which digraphs are available by typing `:digraphs!`, and you can enter them by pressing Control-K and typing the two-character abbreviation associated with the digraph.
For example, to enter the lambda symbol λ, I would type Control-K l* (in Vim syntax, `<C-k>l*`).
Similarly, to enter the beta symbol β, I would type Control-K b* (`<C-k>b*`).

Sometimes, the digraph I want is not available.
In that case, I look up the symbol's hex code in a unicode table, and add it to my custom symbols.
For example, if I want to add the 'does not semantically entail' symbol with the 'ne' abbreviation, I'd look up its hex code (0x22AD), and then add it to Vim with `:execute "digraphs ne " . 0x22AD`.
Now, I can type the symbol by pressing Control-K and typing ne (`<C-k>ne`).
I have a section for symbols that I regularly use in my vimrc, so that I don't have to add them every time I open Vim.

If I'm writing more complicated math, I use Latex inside Vimwiki.
Vimwiki has its own syntax for this: `$...$` for inline math, and `{{$ ... }}$` for block math.
You can find more in `:help vimwiki-syntax-math`.
My math template then adds a JavaScript library to render the Latex, but more on this in a later section.

To insert images, I either take a screenshot and use ss-interceptor (a program I wrote) to save them to a specific folder, or I drag an image to the folder.
The folder is usually next to the notes file, and is named using the subject name, prefixed with 'img-'.
To insert an image into the notes, I use `{{local:path/to/image|Description}}`.
The difference between `local:` and `file:` is that in the generated HTML, `file:` links become absolute, whereas `local:` links become relative.
Therefore, I use `local:`, as I always want relative links.

# Polishing
When I'm done with my notes, I go to polish them and prepare them for publication.
First, I `:set spell` and use `]s` and `[s` to move between spelling mistakes.
To correct a mistake, I either use `z=` and choose an option, or `1z=` to choose the closest spelling.

Next, I create a new folder ({subject name}-notes), with the following structure:
* subject-name-notes/
    * contains the generated HTML files, and the CSS file
* subject-name-notes/src/
    * for the markdown files
* subject-name-notes/img/
    * for any images

I move the markdown file to this subject-name-notes/src, and rename it to 'index.wiki'.
Then, I open it in Vim. and point the image links to the right image folder, using `%s#local:img-subject-name#local:../img#g`.
Next, I generate a table of contents with `:VimwikiTOC`, and start splitting index.wiki into multiple files.

I usually record two macros for splitting files:
1. `V/^== kd:newpggddwvt=hy:p0v$gu:s/ /-/geA.wikiIw :let @"=expand("%<")k:on`
    * this takes a main section out of the file and saves it as a new file, named after the section
    * breakdown:
        * `V/^== `: make a visual line selection until the next '== ' at the start of the line. `` is the enter key.
        * `kd`: go one line up and delete (so as not to delete the header of the next section)
        * `:new`: create a new file
        * `pggdd`: paste the deleted text, go to the start of the file, and delete the blank line at the top
        * `wvt=`: move one word forward, and make a visual selection until the next equal sign
        * `hy`: move one character backwards, then yank the selection
        * `:`: open the command line, then immediately edit the command
        * `p`: paste the yanked text (the name of the section)
        * `v$gu`: visually select until the end of line, then make all letters lowercase
        * `:s/ /-/ge`: substitute all spaces with a dash, ignoring any errors
        * `A.wiki`: append a '.wiki' extension
        * ``: stop editing the command
        * `Iw `: insert 'w ' at the start, preparing to write the buffer to the constructed filename
        * ``: confirm the write
        * `:let @"=expand("%<")`: save the filename without the extension to the `"` register, for use later in macro 2 below
        * `k`: move focus one split up (to the large file with the table of contents)
        * `:on`: close all other buffers
2. `0f#"_dt#PF[a/0`
    * this changes a link in the Vimwiki table of contents to point to the newly created file
    * breakdown:
        * `0`: go to the start of the line
        * `f#`: find the first '# on the line'
        * `"_dt#`: delete up to the next '#', without saving the deleted characters to a register (discarding them into the black hole register)
        * `P`: insert the contents of the `"` register (I usually run this macro after number 1 above, so this register contains the filename)
        * `F[`: find the first previous occurrence of '['
        * `a/`: append a '/' to indicate the root, and press escape (`` is the escape key)
        * `0`: go back to the start of the line

I usually run macros 1 and 2 in sequence, so it ends up looking like this:

After I run these a few times, I have a table-of-contents page that links to other pages.

Next, I generate HTML.

# Generating HTML
* make sure that every vimwiki page points to %template
* %% let g:vimwiki_wikilocal_vars[1]['path_html'] = expand('%:p:h').'/../'
* %% let g:vimwiki_wikilocal_vars[1]['template_path'] = '/Users/alex/Dropbox/vimwiki/templates/'
* fix images with `:Ag ../../img` and `:cdo s#../../img#img#g`
* HTML template example
* style.css
* test with `python -m SimpleHTTPServer`

# Publishing
* create a new repo
* upload to repo
* set up github pages
* add as submodule to original page
