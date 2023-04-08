## 1 - Scanning

### Direct Scans

#### Custom Scripts

Scan for open ports and enumerate ftp, smb, http/https

```bash
# Adjust the IP in this to fit your target net
for i in $(seq 1 254); do echo "Scanning 10.11.1."$i; sudo go_scan.sh 10.11.1.$i; done
```

#### AutoRecon

```bash
autorecon 192.168.1.1/24
```

### Proxy Scans

#### Scan Pingable Hosts

If using a proxy to pivot to an otherwise inaccessible network, scans may be slow, so to reduce the size of scanning:

1. Ping sweep the network from the compromised target that has access to the network first to create a host list 
   <u>On a Windows Target</u>

   ```powershell
   # Adjust the IP in this to fit your target net
   for /L %i in (1,1,255) do @ping -n 1 -w 200 10.1.1.%i > nul && echo 10.1.1.%i>>hostlist.txt
   ```

   <u>On a Linux Target</u>

   ```bash
   # Adjust the IP in this to fit your target net
   for i in {1..254} ;do (ping -c 1 10.1.1.$i | grep "bytes from" | cut -d " " -f 4 | sed s/:$// >> hostlist.txt &); done
   ```

2. On Kali, use that host list to do a full Portscan

   ```bash
   # Adjust the IP in this to fit your target net
   for i in $(cat hostlist.txt); do echo "Scanning "$i; sudo go_proxy_scan.sh $i & done
   ```

#### Scan All Hosts

Once you have these lists, you can proceed, but always remember, not all hosts respond to pings, so you may still want to do another full port scan of the whole range in the background while working on other things to see if any new hosts pop up:

```bash
# Adjust the IP in this to fit your target net
for i in $(seq 1 254); do echo "Scanning 10.11.1."$i; sudo go_proxy_scan.sh 10.1.1.$i & done
```

### Import Scans into Cherrytree

```bash
# Replace the ~/oscplab.... with your directory.
# Picks up each nmap.xml file and adds to CherryTree
# Save, make a backup, and exit Cherry tree before running
for i in $(ls -rt ~/oscplab/public/enumeration/nmap/*.xml); do ~/opt/CherryTree/cherrymap/cherrymap.py $i -m ~/oscplab/CherryTree/OSCPLab.ctd; done

for i in $(ls -rt ~/htblab/oscp_like/enumeration/nmap/10.10.10.222.xml); do ~/opt/CherryTree/cherrymap/cherrymap.py $i -m ~/htblab/CherryTree/htblabs.ctd; done

for i in $(ls -rt ~/htblab/oscp_like/enumeration/10.10.10.140/goscans/nmap_10.10.10.140.xml); do ~/opt/CherryTree/cherrymap/cherrymap.py $i -m ~/htblab/CherryTree/htblabs.ctd; done
```

### Questionable Services

If any services come up with a ?? beside them without some message about them being unrecognized, then try modifying the `totalwaitms` value in `vi /usr/share/nmap/nmap-service-probes`.  The default is 6000, but I had some services that weren't detected in that time, but I found that increasing it to 11000 or more gave it time to get the results.