## Wireshark

### Basics

- Uses libpcap on Linux, winpcap on Windows to capture packets
- Filters
  - Capture filters selectively capture traffic and any traffic filtered out is forever lost.
  - Display filters selectively display already captured traffic
  - Better to spread the net wide (broad capture filters) and pick out the fish you want (display filters).

### Filters

Capture and Display filters sometimes use the same syntax.

There are predefined capture filters under Capture -> Capture filters

There are predefined display filters under Analyze -> Display filters

```bash
# by network
net 10.11.1.0/24

# by TCP port
tcp.port == 21
```

### TCP Streams

More often than not, we are more interested in streams of data rather than individual packets.  TCP streams allow one to reassemble a specific session and display it in various formats.  This function is found by right-clicking a packet, then Follow -> TCP Stream.