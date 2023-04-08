## SMB Share Methodology

1. Check Samba Version - Enumeration of SMB is the same for Windows and Linux except you also need to find the version of SMB for Linux, which doesn't always report properly.   

   ```bash
   # Run this network grep
   sudo ngrep -i -d tun0 's.?a.?m.?b.?a.*[[:digit:]]' port 139
   # Connect with smbclient in another terminal
   smbclient -L 10.11.1.8
   ```

2. Next, check the ./enumeration/smbmap folder for the results from the initial scan

   ```bash
   # Display the results of each host
   for file in $(ls -rt); do echo "****************$file*******************"; cat $file; done | less
   ```

   Or to check one manually:

   ```bash
   # Anonymous
   smbclient -N -L 10.11.1.8
   
   smbclient -L 10.11.1.8 [-U username]
   # Older SMB:
   smbclient -L 10.11.1.8 [-U username] --option='client min protocol=NT1'
   ```

3. For any open shares, download any interesting files.  You may be able to mount it to get more access than just browsing through SMBclient:
   <u>SMB Connection</u>

   ```bash
   # Creates a connection
   smbclient //SERVER/SHARENAME
   
   # For shares with spaces in the name, escape space with backslash
   smbclient //SERVER/SHARE\ NAME
   ```
   
   <u>Mounting the Share</u>
   
   ```bash
   sudo mount //10.11.1.136/ShareName ./localfolder -o rw,vers=1.0,dir_mode=0777,file_mode=0666,nounix,guest
   
   # Options
   vers=1.0 	# Replace with the ver of SMB server is running
   guest		# Specifies anonymous login, otherwise user user=USERNAME,pass=PASSWORD
   ```
   
4. If you are getting `protocol negotiation failed: NT_STATUS_CONNECTION_DISCONNECTED`, then:

   ```bash
   # For a one-time connection.
   smbclient //SERVER/SHARENAME --option='client min protocol=NT1'
   
   # If you need to do more with this connection, read the smbclient doc about modifying your smb.conf file to allow these connections globally.
   ```

   <u>smbclient commands:</u>

   ```bash
   # Listing directories recursively
   recurse
   ls
   
   # Check a file's attributes
   allinfo FILE
   
   # Download files
   get REMOTEFILE
   
   # Upload files
   put LOCALFILE
   
   # Download a folder and its files
   mget REMOTEDIR
   
   # Upload a folder and its files
   mput LOCALDIR
   ```

5. Scan for Vulnerabilities (Nmap might help, better off just using searchsploit though.)

   ```bash
   nmap --script smb-vuln* -p 139,445 SERVER
   ```

### Pipe Audit

Some exploits require you to supply a named pipe.  You can enumerate these with Metasploit:

```bash
sudo msfconsole
use auxiliary/scanner/smb/pipe_auditor
set RHOSTS 10.250.226.25
run
```

### RPCClient

1. Login
   ```bash
   rpcclient -U "" 10.129.14.128
   ```

2. Enumerate system:
   ```bash
   srvinfo
   
   enumdomains
   
   querydominfo
   
   netshareenumall
   
   netsharegetinfo <share>
   ```

3. Enumerate Users:
   ```bash
   enumdomusers
   
   queryuser <user_rid>
   
   querygroup <group_rid>
   ```

4. Bruteforce User RIDs
   ``````bash
   for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
   ``````

### Automated Tools

#### Impacket Samrdump.py

Make sure impacket tools are installed

```bash
samrdump.py 10.129.14.128
```

#### SMBMap

```bash
smbmap -H 10.129.14.128
```

#### CrackMapExec

```bash
crackmapexec smb 10.129.14.128 --shares -u '' -p ''
```

#### Enum4Linux-ng

```bash
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt

./enum4linux-ng.py 10.129.14.128 -A
```





### Logon Command

https://medium.com/@nmappn/exploiting-smb-samba-without-metasploit-series-1-b34291bbfd63