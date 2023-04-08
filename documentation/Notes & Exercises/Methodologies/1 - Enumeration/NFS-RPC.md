## RPC-NFS Enumeration

### NFS Shares

1. The go_scan.sh script will run this automatically, but this command will enumerate the services being offered over RPC.

   ```bash
   nmap -sV -p 111 --script=rpcinfo 10.11.1.1-254
   
   # Or NMAP Scanning for just NFS
   sudo nmap 10.129.14.128 -p111,2049 -sV -sC
   ```

2. If nfs is listed, run additional scans against the NFS shares:

   ```bash
   sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049
   # OR
   showmount -e 10.11.1.72
   ```

3. Mount any discovered nfs shares

   ```bash
   sudo mount -o nolock -t nfs 10.129.14.128:/REMOTEDIR /mnt/nfs
   # OR
   sudo mount -o nolock,nfsvers=3 10.11.1.72:/REMOTEDIR /mnt/nfs
   ```
   
4. Look through filesystem and pay attention to UID/GIDs
   ```bash
   # Show names
   ls -l mnt/nfs/
   
   # Show IDs
   ls -n mnt/nfs/
   ```

5. Unmount when finished
   ```bash
   # Make sure you're not in this directory tree before unmounting or it'll say its busy
   sudo umount /mnt/nfs
   ```

   


### Mounting through Proxy

I could not get NFS to work with Proxychains and online guides don't use it either.  You need to directly forward the ports through the intermediate client.  You can do this with a single hop or multiple hops.

<u>Single Hop:</u>

This will cause you traffic to appear to the NFS server as the IP of your intermediate client:

```bash
# On Kali
# Check rpcinfo for the port you need to forward for `nfs` tcp port and `mountd` tcp port
# ssh user@intermediateIP -L kaliport:nfsIP:nfsPort....
ssh sean@10.11.1.251 -L 2049:10.1.1.27:2049 -f sleep 600m
ssh sean@10.11.1.251 -L 20048:10.1.1.27:20048 -f sleep 600m
# Mount the share
sudo mount -v -t nfs -o port=2049,mountport=20048,tcp 127.0.0.1:/srv/Share ./nfsmnt/
```

<u>Double-Hop:</u>

This will cause your traffic to appear to the NFS server as if it is local.  Note:  In the export file, the ACL can specify localhost or 127.0.0.1.  If you are matching an entry there, this make sure you use the one that is specified there or it won't match.

```bash
# On the Intermediate client
# ssh user@nfsIP -L intermediateport:nfs-localhost:nfsPort....
ssh megan@10.1.1.27 -L 2222:127.0.0.1:2049 -f sleep 600m
ssh megan@10.1.1.27 -L 3333:127.0.0.1:20048 -f sleep 600m
```

```bash
# On Kali
# ssh user@intermediateIP -L kaliport:intermediate-localhost:intermediatePort....
ssh sean@10.11.1.251 -L 2049:localhost:2222 -f sleep 600m
ssh sean@10.11.1.251 -L 20048:localhost:3333 -f sleep 600m
# Mount the share the same way
sudo mount -v -t nfs -o port=2049,mountport=20048,tcp 127.0.0.1:/srv/Share ./nfsmnt/
```

This guide was indispensable in solving this:  https://biowiki.org/wiki/index.php/Mounting_NFSThrough_SSHTunnel

### Windows XP RPC

Immediately vulnerable!

