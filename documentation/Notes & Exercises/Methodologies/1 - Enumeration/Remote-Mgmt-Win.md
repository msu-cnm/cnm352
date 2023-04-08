## RDP

1. Footprint
   ```bash
   # Note: IPS can see that nmap is making this request and block it
   nmap -sV -sC 10.129.201.248 -p3389 --script rdp*
   ```

2. RDP Security Check
   ```bash
   # Install
   sudo cpan
   # From subprompt 
   install Encoding::BER
   
   # Run check
   git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
   ./rdp-sec-check.pl 10.129.201.248
   ```

3. Connect to RDP
   ```bash
   # xfreerdp 
   # Make sure to use single quotes to avoid special characters messing up the command
   xfreerdp /u:cry0l1t3 /p:'P455w0rd!' /v:10.129.201.248
   ```

## WinRM

1. Footprint
   ```bash
   nmap -sV -sC 10.129.201.248 -p5985,5986 --disable-arp-ping -n
   ```

2. Connect
   ```bash
   evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!
   ```

## WMI

1. Footprint
   ```bash
   /usr/share/doc/python3-impacket/examples/wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname"
   ```

2. 