### ARP Scans

After gaining access to a host, it's worth doing some ARP scans on the local subnet to see if any hosts were missed in the nmap scan.

```bash
# I need to fix this, you have to grep the output and cut it up to get just the host IP's
for i in $(seq 1 254); do arping 10.2.2.$i -c 5


```

