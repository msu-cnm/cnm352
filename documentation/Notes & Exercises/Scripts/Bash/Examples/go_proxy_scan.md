Same as go_scan.sh, but for proxy connections

```bash
# ~/opt/scripts/go_proxy_scan.sh
#!/bin/bash
################### NMAP SCANS #######################
# Using TCP Connect scan due to Proxychains
nmapports=$(proxychains nmap -sT -Pn -n -p - --min-rate=1000 -T4 $1 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
if [ ! -z $nmapports ] 
then
	mkdir -p ./enumeration/nmap
	# Using TCP Connect scan due to Proxychains
	proxychains nmap -O -sT -sC -sV -p$nmapports $1 -oA ./enumeration/nmap/$1
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
      wfuzz -R1 -w /usr/share/dirb/wordlists/common.txt -f ./enumeration/wfuzz/wfuzz_http_$1_$port.txt -p 127.0.0.1:1080:SOCKS4 http://$1:$port/FUZZ
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
      wfuzz -R1 -w /usr/share/dirb/wordlists/common.txt -f ./enumeration/wfuzz/wfuzz_https_$1_$port.txt -p 127.0.0.1:1080:SOCKS4 https://$1:$port/FUZZ
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
            proxychains lftp -c "
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
        proxychains smbmap -H $1 -r | grep -v "Working on it..." >> ./enumeration/smbmap/smbmap_$1.txt
    fi
    chown -R coyote: ./enumeration
fi
```

