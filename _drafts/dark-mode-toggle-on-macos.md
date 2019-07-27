---
layout: post
title: "Toggle Dark Mode With a Single Shortcut on macOS"
author: zeroalpha
---
Switching from light mode to dark mode is quite easy by itself.
But what if you want to also change your Terminal theme and your Vim theme, all with a single keyboard shortcut?
Then it gets a little more complicated, but it's still possible.
Here's how you do it.

PS: in this guide I assume the reader is familiar with concepts such as aliases/symbolic links, Python packages and Pip, and basic macOS automation via Automator/AppleScript. If you're not, please google them, as I won't be explaining these concepts.

# Prerequisites
I'm using iTerm2 for my terminal, and Pywal to set the theme.
I recommend you also use Pywal, because it makes it really easy to set your terminal theme based on your background image.

To install Pywal, run `pip3 install pywal`.

In iTerm2, I have two profiles set up.
One for the light theme, called "Default Light", which has an off-white background, a minimum contrast of around 60%, and a light bold color.
The other is for the dark theme, called "Default Dark", and has a black background with "smart box cursor color" enabled.
The final script can also switch between these profiles, so if you want to do this, you should set up similar profiles.

Also, choose two background images - one for the dark theme, and one for the light theme.
Then, save them in a convenient location.
For example, I have mine in ~/Pictures/Backgrounds/.
Also, create symbolic links (or aliases) to those pictures, and name them 'dark' and 'light' respectively.
Once you're done, you should have two files: ~/Pictures/Backgrounds/dark.jpg and ~/Pictures/Backgrounds/light.jpg.
These point to the dark theme and light theme backgrounds respectively.

If you're using Vim and you want to switch color schemes, also make sure you have those prepared and saved in ~/.vim/colors/.

# Toggling the system dark mode
Time to start creating the script.
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

Test it out, you should now see the dock, menubar, and other system elements switching between dark and white theme.

# Switching backgrounds based on dark mode
The next step is to automatically switch to your selected "dark" background when the dark theme is enabled, and to the "light" background when it's disabled.
First, we need to get the state of the dark mode and pass it to a shell script.
To do this, change the applescript from above to this:

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
If it is (i.e. dark mode is on), it calls pywal with the '-i' flag, asking it to generate a dark theme based on the dark image.
Otherwise, it calls pywal with the '-l' flag to generate a light theme, and the '-i' flag to generate the theme based on the light image.
You may need to change '/usr/local/bin/wal' depending on where you installed pywal (find this out by typing `which wal` in the terminal).
You also need to change the paths to the light and dark images to point where you saved your images.

Save the service, and press the keyboard shortcut that you set to toggle dark mode.
You should now also see your background changing.

# Changing the Vim theme

# Changing the iTerm theme
