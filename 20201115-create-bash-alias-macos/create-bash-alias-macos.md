<!--- META
title=Create macOS Terminal aliases
publish_date=20201115
description=Use aliases in the shell to make the Terminal more useful for you!
author=techbitsio
tags=macos
header_image=macbook-air-header.jpg
-->

Aliases in the Terminal are shortcuts to run commands which you might find yourself using a lot, or they could be longer commands that you use infrequently but are more likely to get wrong. Aliases can even be used to override default commands.

Each alias must be one string of characters (could be one letter, or multiple hyphenated words), and should be something that you'll remember.

For example, the `ls` command is often used with the `l` (long-list, i.e. detailed file information) and `a` (show hidden files) options, so creating an alias for `ls -la` might be worth considering if you use this a lot.

Note: The instructions below will work equally well on Linux/Unix systems that use zsh/bash shells, but for bash, the file to edit will be `~./bashrc`.

## macOS Catalina and later (zsh shell)
From Catalina onwards, Apple has changed the default shell to zsh.

Start by editing `.zshrc` in your user home directory (it doesn't matter if this file doesn't already exist):

`nano ~/.zshrc`

Add a newline using the format below, replacing the alias name with your shortcut, and the command in single quotes with what you want to shorten.

`alias la='ls -la'`

Ctrl-X to quit (say Y when prompted to save).

Now run `source ~/.zshrc` to reload the zsh profile (or close and open a new Terminal window).

That's it! Running `la` will now give a detailed list, including hidden files.

## macOS Mojave and earlier (bash shell)
Before zsh was the default macOS shell, it was bash. The process for adding an alias to bash is the same, but we're editing .bash_profile instead of .zsh:

`nano ~/.bash_profile`

Add the alias as above, and after saving the file run `source ~/.bash_profile`.

## Overriding default commands
An an alternative to the `ls -la` example, if you *always* want to see all files, adding the alias `ls='ls -a'`, would mean you then only need to run `ls` to see all files. As running `ls -la` is equivalent to running `ls -l -a`, we'd still be able to add further arguments to the aliased-`ls` – `ls -l` would then the same as `ls -la`.

## Shell alias examples
Here are a few more examples of some aliases:

```bash
alias la='ls -la'

# Override the default behaviour of ls
alias ls='ls -a'

# Quickly run a script
alias myscript='~/Documents/scripts/bash/myscript/script.sh'

# Start a local web server for testing
# web PORT can then be used to specify port
alias web='python3 -m http.server'
```

*Header image by [Junior Teixeira](https://www.pexels.com/photo/semi-opened-laptop-computer-turned-on-on-table-2047905/)*