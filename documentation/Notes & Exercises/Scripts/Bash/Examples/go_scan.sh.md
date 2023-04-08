This script:

- Scan all ports (or modify go_scan.sh for top ports, etc)
- Runs a webserver scan using dirb for any webports 
- Does a recursive directory listing of any FTP servers with anonymous access
- Also tests FTP for write access using a file name ____COYOTE_WRITE_TEST____

Note:  I had a couple of https sites come up with  `443/open/tcp//https?///` and `443/open/tcp//http//CoreHTTP` but I found that these were all non-functional, so I didn't bother trying to include them in the HTTP scans and just stuck with ones that showed `ssl ` and `http` in the scan results.

```bash
# ~/opt/scripts/go_scan.sh
#!/bin/bash
################### NMAP SCANS #######################
nmapports=$(nmap -Pn -p - --min-rate=1000 -T4 $1 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
if [ ! -z $nmapports ] 
then
  mkdir -p ./enumeration/nmap
  nmap -O -sC -sV -p$nmapports $1 -oA ./enumeration/nmap/$1
################### WFUZZ SCANS #######################
######## HTTP
  httpports=$(sed 's/\([0-9]*\/open\/tcp\/\/http[^ ]*\/\/\)/\n\1/g' ./enumeration/nmap/$1.gnmap | grep '\/\/http[^ ]*\/\/' | sed 's/\([0-9]*\)\/open\/tcp\/\/http[^ ]*\/\/.*$/\1/g')
  if [ ! -z "$httpports" ] 
  then
    mkdir -p ./enumeration/wfuzz
    echo "Found http ports: $httpports"
    for port in $httpports 
    do
      # Kicking these off in parallel.  If this causes issues, remove the ampersand
      wfuzz -R1 -w /usr/share/dirb/wordlists/common.txt -f ./enumeration/wfuzz/wfuzz_http_$1_$port.txt http://$1:$port/FUZZ
    done
  fi 
######## HTTPS
  httpsports=$(sed 's/\([0-9]*\/open\/tcp\/\/[^\/]*ssl[^ ]*http[^ ]*\)/\n\1/g' ./enumeration/nmap/$1.gnmap | grep '\/\/ssl[^ ]*http' | sed 's/\([0-9]*\)\/open\/tcp\/\/[^\/]*ssl[^ ]*http.*$/\1/g')
  if [ ! -z "$httpsports" ] 
  then
    mkdir -p ./enumeration/wfuzz
    echo "Found https ports: $httpsports"
    for port in $httpsports 
    do
      # Kicking these off in parallel.  If this causes issues, remove the ampersand
      wfuzz -R1 -w /usr/share/dirb/wordlists/common.txt -f ./enumeration/wfuzz/wfuzz_https_$1_$port.txt https://$1:$port/FUZZ
    done
  fi 
  
################### FTP SCANS #######################
  ftpports=$(sed 's/\([0-9]*\/open\/tcp\/\/ftp\)/\n\1/g' ./enumeration/nmap/$1.gnmap | grep ftp | sed 's/\(^[0-9]*\)[^\n]*/\1/g')
  if [ ! -z "$ftpports" ]
  then
    mkdir -p ./enumeration/ftp
    for port in $ftpports
    do
      echo "~~~~~~~~~~~~~~~~~~ CONNECTING TO $1:$port ~~~~~~~~~~~~~~~" >> ./enumeration/ftp/ftp_$1_$port.txt
      echo "FTP Write Test File" >>  ____coyote_write_test____
      lftp -c "
      set net:timeout 5;
      set net:max-retries 1;
      set net:reconnect-interval-multiplier 1;
      set net:reconnect-interval-base 3;
      open $1:$port;
      put ____coyote_write_test____
      find -l;
      rm ____coyote_write_test____
      exit" >> ./enumeration/ftp/ftp_$1_$port.txt
      rm ____coyote_write_test____
    done
  fi
################### SMB SCANS #######################
  if [ ! -z "$(grep 445/open/tcp ./enumeration/nmap/$1.gnmap | awk -F ": " '{print $2}' | cut -d " " -f 1)" ]
  then
    mkdir -p ./enumeration/smbmap
    smbmap -H $1 -r | grep -v "Working on it..." >> ./enumeration/smbmap/smbmap_$1.txt
  fi
  chown -R coyote: ./enumeration
fi
```

