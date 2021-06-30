<!--- META
title=Autologin and continue running a script in Linux
publish_date=20210319
description=How to get Linux to autologin and resume running a script.
author=techbitsio
tags=linux, scripting
header_image=linux-commandline.jpg
comments=3
-->

I'm going to prefix this whole post with: Computers/servers automatically logging in is **bad security practice**. Certain situations may require this (such as software requiring a GUI/interactive shell on startup), in which case other precautions could be taken to mitigate risk.

The biggest precaution is to limit the logged in user to access the bare minimum required to function. You certainly wouldn't want an admin or root account logging in automatically... would you?

## Log root in automatically

What did I just say?! Ok, fine. I guess it helps to know how to do things badly, plus a bit of context on what I was trying to do that would require this.

First though, this is all tested on **Ubuntu 20.04** (Update: and Ubuntu 21.04). Most things should be compatible with other versions and distros, but your mileage may vary. Secondly, our hypothetical deployment script is called: **deploy.sh**. Catchy, huh? 

I thought it would be fun (honestly) to create a script to automatically deploy servers/websites, plus after rebuilding this site I wanted to nail down the specific configuration so I could:
- easily rebuild it in future
- use the same configuration on a testing/staging site

Before installing any software, I want to perform a `dist-upgrade`, and while you don't always need to reboot, given the subsequent software installs and config changes, it seems like best practice to do so.

To get root to automatically log in following the reboot, we'll create a service:

```bash
mkdir /etc/systemd/system/getty@tty1.service.d/

cat <<EOT >> /etc/systemd/system/getty@tty1.service.d/override.conf
[Service]
ExecStart=
ExecStart=-/sbin/agetty --noissue --autologin root %I $TERM
Type=idle
EOT
```

There are numerous ways to run something at startup, but the best I came up with for running something interactively at login was this:

```bash
echo '[ -z "$SSH_TTY" ] && ./deploy.sh' >> /root/.profile
```
We're echoing `[ -z "$SSH_TTY" ] && ./deploy.sh` to our root `.profile`, which means this line will be executed every time root logs in. The `[ -z "$SSH_TTY" ]` part checks that the $SSH_TTY system variable isn't set (if it's set, it means we've SSH'd in. We don't want to run the script in this case, as every new SSH session would trigger the script again). If it's not set, it runs the script (`./deploy.sh`).

## Resuming the script

Running a script at startup is one thing, but resuming where it left off is another thing altogether. I ended up using [this](https://stackoverflow.com/a/50522191) stackoverflow post as inspiration.

```bash
# Get path to script location
SCRIPT=$( readlink -m $( type -p ${0} ))
run_dir=`dirname "${SCRIPT}"`

# Function to interate through stages and skip those already run
check() {
	step=$1
	step_file=$run_dir/$step.done
	if [ ! -f "$step_file" ]; then
		return 0
	else
		echo "Ignoring $step"
		return 1
	fi
}

# Function to run once a step has successfully run
complete() {
	touch "$run_dir/$1.done" && echo finished step $1
}

function_a() {
	echo "Do something here"
}

function_b() {
	echo "Do something else here"
}

function_reboot() {
	reboot
}

check step_a && function_a && complete step_a
check doreboot && function_reboot && done doreboot
sleep 5
check step_b && function_b && complete step_b
```

After each step, a file with `.done` suffix is created, and before each step, we check to see if that file exists. That means, each step only runs once.

The sleep command guards against the next step running before the reboot has initiated.

## Run as a different user
While this script runs as root when I manually trigger it for the first time, it would ideally continue running as a normal user following the reboot(s).

One of the first things the script does is create a regular user (with sudo privileges), so instead of using root, we can set the user to auto-login (same commands as root, above), and the command to be run as `sudo /root/deploy.sh`.

If we also edit `/etc/sudoers` and add the line `username ALL=NOPASSWD: /root/deploy.sh`, we'll be able to run the script with sudo, without entering the password (for this command only).

As the script was previously running as root, we end up with the same result, except a slight change in how we run a few commands as our non-root, regular user:

`npm install` is something that shouldn't be run as sudo. To run as the user from a root script, you could use: `su username -c "npm install something"`, but as the script is run as sudo, anything inside effectively has sudo, we'd need to drop it with `sudo -u username npm install something`.

This is a lot more secure, and despite temporarily having a logged in user, it is an account that doesn't have elevated permissions.

## Cleaning up
Once everything has run, we can remove the sudo permission, the autologin and the command from .profile:

```bash
# This will stop the script running on login
# root/.profile or /home/username/.profile
# depending on which account you're using.

sed -i '/deploy/d' /root/.profile

# Remove the auto-login
sudo rm /etc/systemd/system/getty@tty1.service.d/override.conf
sudo rmdir /etc/systemd/system/getty@tty1.service.d/

# Remove the ability to run the script as sudo without a password
sudo sed -i '/NOPASSWD/d' /etc/sudoers
```

Once all of this is done, we can perform a final reboot (or just log the user out) and call it a day.

## The result
Here is the final exampl-ified version, combining everything so far:

```bash
# Get path to script location
SCRIPT=$( readlink -m $( type -p ${0} ))
run_dir=`dirname "${SCRIPT}"`  

### CHECK & COMPLETE FUNCTIONS ###

# Function to interate through stages and skip those already run
check() {
	step=$1
	step_file=$run_dir/$step.done
	if [ ! -f "$step_file" ]; then
		return 0
	else
		echo "Ignoring $step"
		return 1
	fi
}

# Function to run once a step has successfully run
complete() {
	touch "$run_dir/$1.done" && echo finished step $1
}

### FUNCTION TO CREATE A CONFIG FILE ### 
## $username VARIABLE WON'T PERSIST ##

config_check() {
	config_file=$run_dir/deploy.config
	if [ ! -f "$config_file" ]; then
		# Create file if doesn't exist
		touch $run_dir/deploy.config
	else
		# Load config file if already exists
		. config_file
	fi
}

### THE FUNCTIONS WE WANT TO RUN ###

add_user() {
	echo "Enter username:"
	read username
	adduser --gecos "" $username
	usermod -aG sudo $username
	rsync --archive --chown=$username:$username ~/.ssh /home/$username
	echo "username=$username" >> config_file
}

enable_autologin_run() {
	mkdir /etc/systemd/system/getty@tty1.service.d/
	cat <<EOT >> /etc/systemd/system/getty@tty1.service.d/override.conf
	[Service]
	ExecStart=
	ExecStart=-/sbin/agetty --noissue --autologin $username %I $TERM
	Type=idle
	EOT
	
	echo '[ -z "$SSH_TTY" ] && sudo /root/deploy.sh' >> /home/$username/.profile
	
	echo "$username ALL=NOPASSWD: /root/deploy.sh" >> /etc/sudoers
}

reboot_time() {
	reboot
}

the_good_stuff() {
	# Install software, dependencies, packages etc.
	# Global packages can be run without
	# sudo -u $username  but some might need this.
	pip3 install a_py_pacakge
	sudo -u $username npm install something_great
}

disable_autologin_run() {
	sed -i '/deploy/d' /home/$username/.profile
	
	rm /etc/systemd/system/getty@tty1.service.d/override.conf
	rmdir /etc/systemd/system/getty@tty1.service.d/
	
	sed -i '/NOPASSWD/d' /etc/sudoers
}

### NOW CALL THE FUNCTIONS ###

config_check # We want this to run every time

check add_user && add_user && complete add_user
check enable && enable_autologin_run && complete enable
check reboot_1 && reboot_time && complete reboot_1
check main && the_good_stuff && complete main
check disable && disable_autologin_run && complete disable
check reboot_2 && reboot_time && complete reboot_2
```

There are, of course, many ways to achieve the various elements above, so I'd be interested to know if there's a simpler way of achieving anything here. If there's anything that's just plain wrong, you can also submit an edit.

*Header image by [Sai Kiran Anagani](https://unsplash.com/photos/Tjbk79TARiE)*
