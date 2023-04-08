# Startup Scripts

There are many different ways to get things to run at startup but only some may be right for the situation.

## Verifying Startup Scripts

You can see what startup scripts are actually loading on your system with `strace`

```bash
# Install strace
sudo apt install strace

# Check files that were opened by the shell
echo exit | strace bash -li |& grep '^open'
#-li means login shell interactive; use only -i for an interactive non-login shell.

```

## Startup in GUI

You can take the cheat way out and just use the desktop manager to setup a startup script.  This differs depending on the desktop manager.

### Xprofile

Most of these desktop managers apparently use ~/.xprofile for startup scripts in the GUI.  Although this didn't work for me in Kali Linux 2021.

### Desktop Managers

To run a program in graphical environment before a user logged in a  graphical environment depend on your display manager. A display manager  is in charge to provide you a login interface and setup your graphical  environment once logged in.   the most important are the following: 

- [GDM](https://wiki.gnome.org/Projects/GDM) is the GNOME display manager.
- [LightDM](https://www.freedesktop.org/wiki/Software/LightDM/) is a cross-desktop display manager, can use various front-ends written in any toolkit.
- [LXDM](https://sourceforge.net/projects/lxdm/) is the LXDE display manager but independent of the LXDE desktop environment.
- [SDDM](https://github.com/sddm/sddm) is a modern display manager for X11 and Wayland aiming to be fast, simple and beautiful.

We will review how to setup the execution of command when the display manager popup before any user logged in and how to execute something  when someone is finally logged in.

If you don't know which one you're running, you can refer to this question :
 [Is there a simple linux command that will tell me what my display manager is?](https://unix.stackexchange.com/q/20370/53092)

**IMPORTANT**
 Before I start, you are going to edit file that except if mention execute command as `root`. Do not remove existing stuff in those files except if you know what  you're doing and be careful in what you put in those file. This could  remove your ability to log in.

### GDM

Be careful with GDM, it will run all script as `root`, a different error code than 0 could limit your log in capability and GDM will wait for  your script to finish making it irresponsive as long as your command  run.   For complete explanation [read the documentation][5].  

#### Before Login

If you need to run commands before a user logged-in you can edit the file:   `/etc/gdm3/Init/Default`.   This file is a shell script that will be executed before the display manager is displayed to the user.  

#### After Login

If you need to execute things once a user has logged in but before its  session has been initialize edit the file:   `/etc/gdm3/PostLogin/Default` If you want to execute command after the session of  session  initialization (env, graphical environment, login...) edit the file:   `/etc/gdm3/PreSession/Default`

------

### LightDM

I will talk about lightdm.conf and not about /etc/lightdm.conf.d/*.conf.  You can do what you want what is important is to know the options you  can use.  Be careful with lightDM, you could already have several other script  starting you should read precisely your config file before editing it.  also the order in which you put those script might influence the way the session load.  

LightDM works a bit differently from the others you will put options  in the main configuration files to indicate script that will be execute.

Edit the main lightDM conf file `/etc/lightdm/lightdm.conf`.

You should add first line with `[Seat:*]`, [as indicated here](https://wiki.ubuntu.com/LightDM):

> Later versions of lightdm (15.10 onwards) have replaced the obsolete [SeatDefaults] with [Seat:*]

#### Before Login

   Add a line `greeter-setup-script=/my/path/to/script`   This script will be executed when lightDM shows the login interface.   

#### After Login

Add a line `session-setup-script=/script/to/start/script`   This will run the script as `root` after a user successfully logged in.  

------

### LXDM

#### Before Login

If you want to execute command before anyone logged in, you can edit the shell script:   `/etc/lxdm/LoginReady`

#### After Login

If you want to execute command after someone logged in but as root, you can edit the shell script:   `/etc/lxdm/PreLogin`   And if you want to run command as the logged in user, you can edit the script:   `/etc/lxdm/PostLogin`

------

### SDDM

#### Before Login

Modify the script located at `/usr/share/sddm/scripts/Xsetup`. This script is executed before the login screen appears and is mostly used to adjust monitor displays in X11. *Not sure what the equivalent would be for wayland*

#### After Login

`sddm` will now source the script located at `/usr/share/sddm/scripts/Xsession`, which in turn will source the user's [dotfiles](https://wiki.archlinux.org/title/Dotfiles) depending on their default shell.

For bash shell, it will source `~/.bash_profile` (among others), and for zsh, it will source `${ZDOTDIR:-$HOME}/.zprofile` (among others). You can take this opportunity to modify those files to also run any other command you need after logging in.

## Run after network interface is up

```bash
sudo touch /etc/network/if-up.d/script.sh
sudo chmod 755 /etc/network/if-up.d/script.sh
sudo chown root:root /etc/network/if-up.d/script.sh
```

Other places to put things that I didn't get to work:

## Zsh

Zsh always executes *zshenv*. Then, depending on the case:

- run as a login shell, it executes ~/.zprofile
- run as an interactive, it executes ~/.zshrc
- run as a login shell, it executes ~/.zlogin

At the end of a login session, it executes *zlogout*, but in reverse order, the user-specific file first, then the system-wide one, constituting a chiasmus with the *zlogin* files.

More reading:  https://zsh.sourceforge.io/Guide/zshguide02.html

## Bash

###  [/etc/profile](https://developer.toradex.com/knowledge-base/how-to-autorun-application-at-the-start-up-in-linux#etcprofile)

Each time a login shell is spawned, the script **/etc/profile** plus all scripts in `/etc/profile.d` are executed.

This is done for logins over a **serial** console, over an **ssh** connection and also for logins in the **display manager** to a graphical desktop.

**/etc/profile** is sourced upon login: it sets up the  environment upon login and application-specific settings by sourcing any readable file in `/etc/profile.d/`.

Using **/etc/profile** is well suited to set the environment or to do some small tasks.

Note that these scripts must return control in order to continue with the login.

Remove the file in `/etc/profile.d` or the additions to **/etc/profile** in order to undo the automatic execution.

To load a Shell Script at each login, you just need to add a script file in `/etc/profile.d/`.

Remembering that the Shell Script must have the ***.sh** extension.

Below is an example of a script file for deleting backup entries:

 remove_backup.sh

```
#!/bin/sh
rm /home/root/*~
```

### Bash Startup Files

This section describes how Bash executes its startup files. If any of the files exist but cannot be read, Bash reports an error. Tildes are expanded in filenames as described above under Tilde Expansion (see [Tilde Expansion](https://www.gnu.org/software/bash/manual/html_node/Tilde-Expansion.html)).

Interactive shells are described in [Interactive Shells](https://www.gnu.org/software/bash/manual/html_node/Interactive-Shells.html).

#### Invoked as an interactive login shell, or with --login

When Bash is invoked as an interactive login shell, or as a non-interactive shell with the --login option, it first reads and executes commands from the file /etc/profile, if that file exists. After reading that file, it looks for ~/.bash_profile, ~/.bash_login, and ~/.profile, in that order, and reads and executes commands from the first one that exists and is readable. The --noprofile option may be used when the shell is started to inhibit this behavior.

When an interactive login shell exits, or a non-interactive login shell executes the `exit` builtin command, Bash reads and executes commands from the file ~/.bash_logout, if it exists.

#### Invoked as an interactive non-login shell

When an interactive shell that is not a login shell is started, Bash reads and executes commands from ~/.bashrc, if that file exists. This may be inhibited by using the --norc option. The --rcfile file option will force Bash to read and execute commands from file instead of ~/.bashrc.

So, typically, your ~/.bash_profile contains the line

```
if [ -f ~/.bashrc ]; then . ~/.bashrc; fi
```

after (or before) any login-specific initializations.



#### Invoked non-interactively

When Bash is started non-interactively, to run a shell script, for example, it looks for the variable `BASH_ENV` in the environment, expands its value if it appears there, and uses the expanded value as the name of a file to read and execute.  Bash behaves as if the following command were executed:

```
if [ -n "$BASH_ENV" ]; then . "$BASH_ENV"; fi
```

but the value of the `PATH` variable is not used to search for the filename.

As noted above, if a non-interactive shell is invoked with the --login option, Bash attempts to read and execute commands from the login shell startup files. 