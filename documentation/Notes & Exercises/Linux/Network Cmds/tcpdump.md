## TCPDump

### Command Options

```bash
sudo 	# required for capturing packets
-r		# read from a file
-n		# disable dns lookups
-X		# print packet data in both HEX & ASCII format
-A		# print packet data in ASCII
-s		# Specify the amount of data to capture. Defaults to 262144 bytes.
```

### Filters

```bash
src [host IP|NETWORK/MASK] 	# Filter by source host or network
dst [host IP|NETWORK/MASK] 	# Filter by destination host or network
port PORT					# Filter by port
```

#### Advanced Header Filtering

![image-20200608150626693](.tcpdump.assets/image-20200608150626693.png)

```bash
Flags occupy the 14th byte of the header (position 13):

CEUAPRSF
WCRCSSYI
REGKHTNN
00011000 = 24 (Value of ACK+PSH in decimal)
00000010 = 2 (Value of SYN in decimal)
00000100 = 4 (Value of RST in decimal)
00010000 = 16 (Value of ACK in decimal)
```

##### Flags

- ACK - used on all packets sent & received after the 3-way handshake.  4th bit of the 
- PSH - Enforces immediate delivery of packet & commonly used in interactive Application Layer protocols to avoid buffering (delaying).

Flags can be filtered by bits:

```bash
'tcp[13] = Y'				# Filter where tcp header 13 (the flags section, byte #14) equals decimal value Y.
# ex.
'tcp[13] = 24'		# filters for packets with ACK+PUSH set
```

or by name

```bash
# Match if ONLY a single flag is set and no others
'tcp[tcpflags] == FLAG'
# Match one flag or another
'tcp[tcpflags] & (FLAG1|FLAG2) != 0'
# Flag options are: tcp-fin, tcp-syn,  tcp-rst, tcp-push, tcp-act, tcp-urg
# Ex. Filter for ACK OR PUSH
'tcp[tcpflags] & (tcp-ack|tcp-push) != 0'
# Ex. Filter for ACK AND PUSH - note the double && and single &
'tcp[tcpflags] & tcp-ack != 0 && tcp-push != 0'
```

### Booleans

Booleans are specified between different filters as lowercase 'or', 'and', etc.

### Shortcuts

Grab all destination IP's in a pcap file, sort by IP,  show only unique IP's with the number of occurrences and display the top 10.

```bash
sudo tcpdump -r file.pcap -n | awk -F " " '{print $3}' | sort | uniq -c | head
```

