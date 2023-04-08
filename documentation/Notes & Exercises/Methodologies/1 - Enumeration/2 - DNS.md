## DNS Enumeration

Figure out Domain

```bash
# There's not a guaranteed way, you can check the nmap output or try looking up the DNS server against itself:
host 10.10.10.13 10.10.10.13
```

Query Name servers

```bash
# Query Name Servers
dig ns inlanefreight.htb @10.129.14.128

# OR
nslookup -type=NS inlanefreight.htb ns.namerserver.com
# inlanefreight=domain to query
# ns.nameserver.com=NS to ask
```

Query DNS server version

```bash
# Query DNS Server version
dig CH TXT version.bind 10.129.120.85
```

Query A records

```bash
# 
dig inlanefreight.htb @10.129.14.128

# 
export TARGET="inlanefreight.htb"
nslookup $TARGET ns.namerserver.com

#
host -l domain.local 10.11.1.1
# host -l domain host_ip
```

Query records for subdomain

```bash
# 
dig a www.inlanefreight.htb @10.129.14.128

# 
export TARGET=www.inlanefreight.htb
nslookup -query=A $TARGET ns.namerserver.com
```

Query any available records

```bash
# This likely won't work because it's deprecated in RFC8482 

# 
dig any inlanefreight.htb @10.129.14.128

# 
export TARGET="inlanefreight.htb"
nslookup -query=ANY $TARGET ns.namerserver.com
```

Query PTR Records for reverse Lookup from IP

```bash
# 
dig -x 1.2.3.4 @10.129.14.128

# 
nslookup -query=PTR 1.2.3.4 ns.namerserver.com

#
for IP in $(seq 1 254); do host 10.10.10.$IP 10.10.10.13; done | grep pointer >> reversedns.txt
# host IP-to-reverse DNS-server
```

Query TXT Records

```bash
#
dig txt facebook.com @1.1.1.1

#
export TARGET="facebook.com"
nslookup -query=TXT $TARGET
```

Query MX Records

```bash
dig mx facebook.com @1.1.1.1

export TARGET="facebook.com"
nslookup -query=MX $TARGET
```

Zone Transfers

First look for name servers (see above sections) to find subdomains, then try to zone transfer them:

```bash
# (note you can specify subdomains to get more results)
dig axfr inlanefreight.htb @10.129.14.128

#
host -l thinc.local 10.11.1.220 > zonexfer
# host -l domain Auth_DNS > outfile

# OR
nslookup -type=any -query=AXFR domain.com ns.nameserver.com
```

### Dig Tricks

```bash
## Shorten responses
# Add in +noall +answer to the end, but you won't get anything if there's no answer
dig axfr inlanefreight.htb @10.129.14.128 +noall +answer 
```

### Subdomain Enumeration

#### Bruteforcing

DNS Enum

```bash
# Subdomain Bruteforcing
# Get list
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/DNS/subdomains-top1million-5000.txt
# Run script
for sub in $(cat subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done

dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f subdomains-top1million-110000.txt inlanefreight.htb
```

Gobuster

```bash
# Create file with URL patterns for subdomain (don't inlcude main domain in this)
# patterns.txt
lert-api-shv-{GOBUSTER}-sin6
atlas-pp-shv-{GOBUSTER}-sin6

# Setup variables (just a clean way to do it)
export TARGET="facebook.com"
export NS="d.ns.facebook.com"
export WORDLIST="numbers.txt"
gobuster dns -q -r "${NS}" -d "${TARGET}" -w "${WORDLIST}" -p ./patterns.txt -o "gobuster_${TARGET}.txt"
```

#### Seclists Subdomain lists

https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/



