<!--- META
title=Quickly add a Config File to your Bash Scripts
publish_date=20200618
description=Quickly add a config file to your bash scripts using source.
author=techbitsio
tags=bash,scripting
header_image=linux-commandline.jpg
comments=8
-->

**Source** lets you add external files/code into your scripts, but is also a great way of adding config files, providing you are aware of the risks.

The benefits are that source is extremely quick to run, with memorable syntax.

The risks are that all code in the file(s) you source will be run, even if you only anticipated the file holding variables.

In short, if there's any chance of anyone else adding/editing the config files (a curious user, perhaps), then source isn't the right option.

Let's say you have a config file (`config.ini`) that contains `your_var=your_val`.

In your script, you can use `source` or the shorthand `.` syntax

```bash
. config.ini
# or: source config.ini

echo $your_var 

>>> your_val
```

But if anyone can edit `config.ini`, and they add executable commands... Well just hope they are messing with you and add a relatively painless `reboot`, rather than `rm -rf /`. You've been warned!

*Header image by [Sai Kiran Anagani](https://unsplash.com/photos/Tjbk79TARiE)*