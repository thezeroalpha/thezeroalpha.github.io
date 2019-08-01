---
layout: post
title: "Toggle Dark Mode With a Single Shortcut on macOS"
author: zeroalpha
---
Switching from light mode to dark mode is quite easy by itself.
But what if you want to also change your Terminal theme and your Vim theme, all with a single keyboard shortcut?
Then it gets a little more complicated, but it's still possible.
Here's how you do it.

In this guide I assume the reader is familiar with concepts such as aliases/symbolic links, Python packages and Pip, and basic macOS automation via Automator/AppleScript. If you're not, please google them, as I won't be explaining these concepts.
The final setup is presented in the last section, ["Wrapping Up"](#wrapping-up).

FYI, this is what I mean by toggling dark mode:

![Demo of dark mode toggling](/img/dark-mode-switching.gif)

## Prerequisites
I'm using [iTerm2](https://www.iterm2.com/) for my terminal, and [Pywal](https://github.com/dylanaraps/pywal) to set the theme.
I recommend you also use Pywal, because it makes it really easy to set your terminal theme based on your background image.

You'll also probably want Pywal to run at login and recall your most recent profile.
You can do this with an Automator application.
I have one called "wal-init.app" that looks like this:

![wal-init Automator application](/img/wal-init-automator.png)

I then set it to run at login via System Preferences → Users & Groups → Login Items.
If you do the same, make sure to change any paths to those that are relevant for your system.
Parts of the wal-init app will explained in later sections of this post.

In iTerm2, I have two profiles set up.
One for the light theme, called "Default Light", which has an off-white background, a minimum contrast of around 60%, and a light bold color.
The other is for the dark theme, called "Default Dark", and has a black background with "smart box cursor color" enabled.
The final script can also switch between these profiles, so if you want to do this, you should set up similar profiles.
It's a good idea to do this to avoid flashbanging yourself with every new window when dark mode is enabled, as it takes about a second for Pywal to react and change the theme.

![iTerm profiles for dark/light mode](/img/iterm-theme-profiles.png)

Also, choose two background images - one for the dark theme, and one for the light theme.
Then, save them in a convenient location.
For example, I have mine in ~/Pictures/Backgrounds/.
Also, create symbolic links (or aliases) to those pictures, and name them 'dark' and 'light' respectively.
Once you're done, you should have two files: ~/Pictures/Backgrounds/dark.jpg and ~/Pictures/Backgrounds/light.jpg.
These point to the dark theme and light theme backgrounds respectively.

If you're using Vim and you want to switch color schemes, also make sure you have those prepared and saved in ~/.vim/colors/.

## Toggling the system dark mode
Time to start creating the script.
The first part is an Automator quick action, which will tie the various components together to respond to a keyboard shortcut.
For this, open Automator, create a 'quick action', and search for "Run AppleScript" in the library pane on the left.
Drag that into the editing pane on the right as the first element in the workflow.

At the very top, make sure that you set the "workflow receives" fields to "no input" in "any application".
Then, inside "Run AppleScript", enter the following lines:

```applescript
on run {input, parameters}
    tell application "System Events"
        tell appearance preferences
            set dark mode to (not dark mode)
        end tell
    end tell
end run
```

This sets the "dark mode" state to the opposite of what it currently is, effectively toggling it.

Now if you save this as e.g. "Toggle Dark Mode" and click the run button, it should toggle the system-wide dark mode.

To bind it to a keyboard shortcut, open System Preferences, select the Keyboard section, and select the Shortcuts tab.
On the left side, select "services", then scroll all the way down to the "General" section.
Make sure your service ("Toggle Dark Mode" in this case) is enabled (i.e. the box next to the name is checked), and then click on the right where it says "none" and enter a keyboard shortcut.
I use control-alt-cmd-t.

![Image of keyboard preferences](/img/dark-mode-shortcut.png)

Test it out, you should now see the dock, menu bar, and other system elements switching between dark and white theme.

## Switching desktop backgrounds based on dark mode
The next step is to automatically switch to your selected "dark" background when dark mode is enabled, and to the "light" background when it's disabled.
First, we need to get the state of the dark mode and pass it to a shell script.
To do this, change the AppleScript from above to this:

```applescript
on run {input, parameters}
    tell application "System Events"
        tell appearace preferences
            set dark mode to (not dark mode)
            return (get dark mode as text)
        end tell
    end tell
end run
```

What changed is the 'return' line, which gets the current state of the dark mode setting (after toggling it) and returns it.
This will pass on the string "true" or "false" to the next element in the Automator workflow.

In the left library pane in Automator, search from "Run Shell Script", and drag it under the "Run AppleScript" element.
At the top of "Run Shell Script", make sure the shell is set to "/bin/bash", and "pass input" is set to "as arguments".

In the text section, add the following script:

```bash
if [ "$1" = "true" ]; then
    /usr/local/bin/wal -i ~/Pictures/Backgrounds/dark.jpg;
else
    /usr/local/bin/wal -l -i ~/Pictures/Backgrounds/light.jpg;
fi
```

This script first checks if the first argument (the value returned from the "Run AppleScript" element) is true.
If it is (i.e. dark mode is on), it calls Pywal with the '-i' flag, asking it to generate a dark theme based on the dark image.
Otherwise, it calls Pywal with the '-l' flag to generate a light theme, and the '-i' flag to generate the theme based on the light image.
You may need to change `/usr/local/bin/wal` depending on where you installed Pywal (find this out by typing `which wal` in the terminal).
You also need to change the paths to the light and dark images to point where you saved your images.

Save the service, and press the keyboard shortcut that you set to toggle dark mode.
You should now also see your background changing.

## Changing the iTerm theme
With iTerm2 version 3.3, a new [Python scripting API](https://www.iterm2.com/python-api/) was added, making life much easier.
This means that it's now possible to set a default profile via Python.

Open iTerm2, and in the menu bar, click Scripts, then Manage, and select New Python Script.
Name it "toggle_dark_mode.py".
In the editor window that opens up, enter the following Python code:

```python
#!/usr/bin/env python3.7
import iterm2
from sys import argv

async def main(connection):
    # Get only profiles whose name starts with "Default" (so "Default Light" and "Default Dark")
    profiles = [p for p in await iterm2.PartialProfile.async_query(connection) if p.name.startswith("Default ")]

    # If the theme was given via a command line argument, use it.
    if len(argv) > 1:
        is_dark_theme = str(argv[1])
    # Otherwise, ask AppleScript whether dark mode is on
    else:
        import subprocess
        result = subprocess.run(['osascript', '-e', 'tell application "System Events" to tell appearance preferences to return (get dark mode as text)'], stdout=subprocess.PIPE)
        is_dark_theme = str(result.stdout.rstrip().decode("utf-8"))

    # Cycle to either the light or dark profile, and set it as default, depending on the current theme
    for p in profiles:
        if "Light" in p.name and is_dark_theme == "false":
            await p.async_make_default()
        elif "Dark" in p.name and is_dark_theme == "true":
            await p.async_make_default()

iterm2.run_until_complete(main)
```

This script will accept an argument on the command line -- "true" or "false", depending on whether dark mode is on or off.
If no argument is supplied, it'll find out by itself via AppleScript (but we won't use that part much).

Now we need to add it to the Automator workflow.

Open up the same Automator workflow from the previous part ("Toggle Dark Mode"), and in the shell script section, change the text to this:

```bash
if [ "$1" = "true" ]; then
    /usr/local/bin/wal -i ~/Pictures/Backgrounds/dark.jpg;
    ~/Library/ApplicationSupport/iTerm2/iterm2env/versions/**/bin/python3 ~/Library/ApplicationSupport/iTerm2/Scripts/toggle_dark_mode.py true
else
    /usr/local/bin/wal -l -i ~/Pictures/Backgrounds/light.jpg;
    ~/Library/ApplicationSupport/iTerm2/iterm2env/versions/**/bin/python3 ~/Library/ApplicationSupport/iTerm2/Scripts/toggle_dark_mode.py false
fi
```

The lines that are added are those that call the toggle_dark_mode.py script, and provide it with an argument of either "true" or "false".
"true" if dark mode is on, "false" otherwise.

You can save the workflow and quit Automator and the script editor.

The final workflow should look like this:

![Image of Automator workflow](/img/toggle-dark-mode-workflow.png)

At this point, when you press the shortcut you set up, the iTerm theme should change along with the system theme and your desktop background.

## Changing the Vim theme
Finally, if you use Vim as your editor, you might want to change your theme and colors for dark mode.

To do so, add this to your ~/.vimrc:

```vim
" Ask via AppleScript if dark mode is on
if system("osascript -e 'tell application \"System Events\" to tell appearance preferences to return (get dark mode as text)'") == "true\n"
  " If it is, use colors that are better on a dark background
  set background=dark

  " And use a dark color scheme
  colorscheme {dark_scheme}
else
  " Otherwise, use colors that are better on a light background
  set background=light

  " And use a light color scheme
  colorscheme {light_scheme}
endif
```

You have to replace `{dark_scheme}` and `{light_scheme}` with the name of the color scheme that you want to use for dark/light mode.

And that's everything!
Now when you toggle dark mode using your shortcut, iTerm and Vim should follow suit.

## Wrapping up
Here's what all of the files look like in the final state.
Make sure to replace any file paths with those that are relevant to your system.

The Automator workflow:

![Final Automator workflow](/img/toggle-dark-mode-workflow.png)

~/Library/Application Support/iTerm2/Scripts/toggle_dark_mode.py:

```python
#!/usr/bin/env python3.7

import iterm2
from sys import argv

async def main(connection):
    profiles = [p for p in await iterm2.PartialProfile.async_query(connection) if p.name.startswith("Default ")]
    if len(argv) > 1:
        is_dark_theme = str(argv[1])
    else:
        import subprocess
        result = subprocess.run(['osascript', '-e', 'tell application "System Events" to tell appearance preferences to return (get dark mode as text)'], stdout=subprocess.PIPE)
        is_dark_theme = str(result.stdout.rstrip().decode("utf-8"))

    for p in profiles:
        if "Light" in p.name and is_dark_theme == "false":
            await p.async_make_default()
        elif "Dark" in p.name and is_dark_theme == "true":
            await p.async_make_default()

iterm2.run_until_complete(main)
```

The relevant ~/.vimrc section (you have to replace the theme name placeholders):

```vim

if system("osascript -e 'tell application \"System Events\" to tell appearance preferences to return (get dark mode as text)'") == "true\n"
  set background=dark
  colorscheme {dark_theme}
else
  set background=light
  colorscheme {light_theme}
endif
```
