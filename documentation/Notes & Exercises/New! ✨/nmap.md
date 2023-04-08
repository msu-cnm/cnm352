New options to try

```bash
# Disables arp pings
--disable-arp-ping 
# Shows tcp dump for relevant responses
--packet-trace
# Displays the reason a port is in a particular state.
--reason
# Report status of nmap scan every 5 seconds, m for mins also works
--stats-every=5s
# Change verbosity to see open ports as they're discovered
-v
-vv

```

## Firewall/IPS/IDS Evasion

- ACK Scans are harder to filter (I think this is only good for determining port status)
  ```bash
  sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace
  
  10.129.2.28 	Scans the specified target.
  -p 21,22,25 	Scans only the specified ports.
  -sS 	Performs SYN scan on specified ports.
  -sA 	Performs ACK scan on specified ports.
  -Pn 	Disables ICMP Echo requests.
  -n 	Disables DNS resolution.
  --disable-arp-ping 	Disables ARP ping.
  --packet-trace 	Shows all packets sent and received.
  ```

- Decoys (found no use for this)
  ```bash
  sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
  
  10.129.2.28 	Scans the specified target.
  -p 80 	Scans only the specified ports.
  -sS 	Performs SYN scan on specified ports.
  -Pn 	Disables ICMP Echo requests.
  -n 	Disables DNS resolution.
  --disable-arp-ping 	Disables ARP ping.
  --packet-trace 	Shows all packets sent and received.
  -D RND:5 	Generates five random IP addresses that indicates the source IP the connection comes from.
  ```

- Specify Source IP if current network is blocked (only works if you have multiple IPs to use)
  ```bash
  sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0
  
  10.129.2.28 	Scans the specified target.
  -n 	Disables DNS resolution.
  -Pn 	Disables ICMP Echo requests.
  -p 445 	Scans only the specified ports.
  -O 	Performs operation system detection scan.
  -S 	Scans the target by using different source IP address.
  10.129.2.200 	Specifies the source IP address.
  -e tun0 	Sends all requests through the specified interface.
  ```

- DNS Proxying to scan from a source port with that has less restrictions
  ```bash
  sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
  
  10.129.2.28 	Scans the specified target.
  -p 50000 	Scans only the specified ports.
  -sS 	Performs SYN scan on specified ports.
  -Pn 	Disables ICMP Echo requests.
  -n 	Disables DNS resolution.
  --disable-arp-ping 	Disables ARP ping.
  --packet-trace 	Shows all packets sent and received.
  --source-port 53 	Performs the scans from specified source port.
  ```

  

