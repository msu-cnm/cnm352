### Recon Examples

#### Extract domains from file & lookup ip

Digs through a file, finds all subdomains of a given hostname, then performs a DNS lookup on them.

```bash
grep -o '[^/]*\.HOSTNAME\.com' index.html | sort -u > list.txt

for url in $(cat list.txt); do host $url; done | grep "has address" | cut -d " " -f 4 | sort -u
```

#### Download exploits by keyword

Downloads all exploits matching given keywords to individual files named with the exploit ID.

```bash
# One-liner
for e in $(searchsploit KEYWORD1 KEYWORD2.. -w -t | grep http | cut -f 2 -d "|"); do exp_name=$(echo $e | cut -d "/" -f 5) && url=$(echo $e | sed 's/exploits/raw/') && wget -q --no-check-certificate $url -O $exp_name; done
```

```bash
# Script file
#!/bin/bash
# Bash script to search for a given exploit and download all matches.
for e in $(searchsploit afd windows -w -t | grep http | cut -f 2 -d "|")
do
exp_name=$(echo $e | cut -d "/" -f 5)
url=$(echo $e | sed 's/exploits/raw/')
wget -q --no-check-certificate $url -O $exp_name
done
```

#### Scan for webservers & grab snapshot of main page

```bash
# Perform nmap scan & output to grepable file
sudo nmap -A -p80 --open 10.11.1.0/24 -oG nmap-scan_10.11.1.1-254
# Feeds in an nmap grepable format file & outputs image of front page of each server
for ip in $(cat nmap-scan_10.11.1.1-254 | grep 80 | grep -v "Nmap" | awk '{print $2}'); do cutycapt --url=$ip --out=$ip.png;done

for url in $(grep index * | grep 200 | cut -d " " -f 2); do cutycapt --url=$url --out=$url.png; done;
```

#### Build a webpage to display several images

```bash
# Build a single html doc to show all the images found
# Note - that is ls -ONE, not ls -L in lowercase
echo "<HTML><BODY><BR>" > web.html
ls -1 *.png | awk -F : '{ print $1":\n<BR><IMG SRC=\""$1""$2"\" width=600><BR>"}' >> w
eb.html
echo "</BODY></HTML>" >> web.html
```

