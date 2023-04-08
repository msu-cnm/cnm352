## SNMP Enumeration

UDP Port 161

1. Footprinting
   ```bash
   # With SNMPWalk
   snmpwalk -v2c -c public 10.129.14.128
   
   # With OnesixtyOne
   sudo apt install onesixtyone
   onesixtyone -c /opt/useful/SecLists/Discovery/SNMP/snmp.txt 10.129.14.128
   
   # With Braa
   sudo apt install braa
   braa <community string>@<IP>:.1.3.6.*   # Syntax
   braa public@10.129.14.128:.1.3.6.*
   
   
   ```

2. Figure



