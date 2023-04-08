## Tmux Tutorial

-   Change to the directory you want to be the default for new tmux windows

-   Most commands start with the 'Prefix Key' which is CTRL+B by default

-   You may want to modify your prefix key if you plan to remote into other machines with tmux so you can run tmux commands on your local session without the remote session interpreting them.

-   To do logging requires this plugin
    https://github.com/tmux-plugins/tmux-logging

### Installing Tmux

```bash
# Debian Ubuntu
sudo apt-get install tmux

# Redhat/Fedora
sudo yum install tmux
```

### Configuration File

```bash
#~/.tmux.conf
# Remap prefix to CTRL+A and unmap CTRL+B
set -g prefix C-a
bind C-a send-prefix
unbind C-b

# Quality of Life
# sets the command line history
set -g history-limit 10000
# tmux windows won't dynamically rename if manually named
set -g allow-rename off

# Join Windows
bind-key j command-prompt -p "join pane from:" "join-pane -s ':%%'"
bind-key s command-prompt -p "send pane to:" "join-pane -t ':%%'"

# Search Mode VI (default is emac)
set-window-option -g mode-keys vi

# Enable Logging Module
run-shell /home/coyote/opt/tmux-logging/logging.tmux
```

### Logging Configuration Changes

```bash
#~/opt/tmux-logging/scripts/variables.sh

###Changed default directories
default_logging_path="$HOME/oscplab/console-logs"
default_screen_capture_path="$HOME/oscplab/console-logs"
default_save_complete_history_path="$HOME/oscplab/console-logs"
```

### Logging on Startup

```bash
#~/.profile

# Added this line to start logging for tmux automatically when creating a new pane
# This didn't work well with an autostart script because it would only log the last window that opened from the auto-start script
/home/coyote/opt/tmux-logging/scripts/toggle_logging.sh
```

### tmux Bash Commands

```Bash
# You can use these commands to prebuild a tmux session
# Examples
# Create new session and name it, then start vim
tmux new-session -d -n 'WindowName' 'vim' 
	# -d means don't attach
# Split a window
tmux split-window [-v|-h]
	# -v draws the split line horizontally and -h draws the split line vertically, which is opposite what you'd think
# Add a new window to the session and name it, then start python
tmux new-window -n 'WindowName' 'python'
# Attach to the created session
tmux -2 attach-session -d
	# -d causes any other clients attached to the session to be disconnected
	# -2 forces tmux to assume the terminal supports 256 colors
```

### Commands

#### Session Commands

```bash
# Start a new tmux session:
tmux -new <name>

# Create a new window
PREFIX C

# Switch between open windows (# at bottom)
PREFIX <window_number>

# List all tmux sessions
tmux ls

# Attach to a tmux session
tmux attach -t <session_name>

# Detach from a session
PREFIX d

# Rename current window
PREFIX ,

# Scroll through the history
PREFIX PGUP/PGDOWN
```

####  Edit Mode

Edit mode allows you to interact with the output in the terminal.

```bash
# Edit mode allows you to interact with the output in the terminal.

# Start Edit Mode
PREFIX [

# Searching
backwards: ?
forwards: /

# Highlight Mode (after entering edit mode)
SPACE

# Exit Edit Mode/Copy Highlighted Text to buffer
ENTER

# Paste Text in Buffer
PREFIX ]
```

#### Windows Splitting

```bash
# Split current pane vertically
PREFIX %

# Split current pane horizontally
PREFIX “

# Flip between vertical and horizontal split
PREFIX <space>

# Move between panes 
# (Press arrow keys in quick succession to keep moving)
PREFIX <arrow keys>

# Adjust current pane size
# (Press arrow keys in quick succession to keep resizing)
PREFIX (hold CTRL) <arrow keys> 

# Zoom into a pane
PREFIX Z

# Move panes around
left: PREFIX {
right: PREFIX }

# Identify Windows
PREFIX Q
```

#### Misc Commands

```bash
# Help Menu
PREFIX ?

# Show Clock
PREFIX t
```

#### Logging (from tmux-logging docs)

```bash
Logging

Toggle (start/stop) logging in the current pane.

    Key binding: PREFIX + SHIFT + P
    File name format: tmux-#{session_name}-#{window_index}-#{pane_index}-%Y%m%dT%H%M%S.log
    File path: $HOME (user home dir)
        Example file: ~/tmux-my-session-0-1-20140527T165614.log

2. "Screen Capture"

Save visible text, in the current pane. Equivalent of a "textual screenshot".

    Key binding: prefix + alt + p
    File name format: tmux-screen-capture-#{session_name}-#{window_index}-#{pane_index}-%Y%m%dT%H%M%S.log
    File path: $HOME (user home dir)
        Example file: tmux-screen-capture-my-session-0-1-20140527T165614.log

3. Save complete history

Save complete pane history to a file. Convenient if you retroactively remember you need to log/save all the work.

    Key binding: prefix + alt + shift + p
    File name format: tmux-history-#{session_name}-#{window_index}-#{pane_index}-%Y%m%dT%H%M%S.log
    File path: $HOME (user home dir)
        Example file: tmux-history-my-session-0-1-20140527T165614.log

NOTE: this functionality depends on the value of history-limit - the number of lines Tmux keeps in the scrollback buffer. Only what Tmux kept will also be saved, to a file.

Use set -g history-limit 50000 in .tmux.conf, with modern computers it is ok to set this option to a high number.
4. Clear pane history

Key binding: prefix + alt + c

This is just a convenience key binding.
```



###  General CLI Tricks

```bash
Search Command History
(Searches through previous commands)
CTRL+R

Argument history
(Cycles through previous arguments)
ALT+.

Move Around the CLI Line
to start: CTRL+A
to end: CTRL+E
to breaks in line: CTRL+ <arrow>
```

Note: If rebinding tmux to a key sequence that already has a use, just repeat the last letter to initiate the default usage. Ex. CTRL+A normally takes you to the start of the line. If binding tmux to CTRL+A, then to go to the start of the line, enter CTRL+A A
