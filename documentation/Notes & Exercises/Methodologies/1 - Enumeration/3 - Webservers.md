## 2 - Web Servers

### Sanity Checks

1. Pay careful attention to Nikto output!  Don't miss any default creds found!

### Web Server Infrastructure

#### HTTP Headers


```bash
curl -I "http://${TARGET}"
```

#### WhatWeb

ID web technologies, CMS, blogging platforms, stat/analytics packages, JS libs, web servers and embedded devices

```bash
whatweb -a3 https://www.facebook.com -v
```

#### Wappalyzer

Web browser extension to ID website

https://www.wappalyzer.com/

#### WafW00f

Web Application Firewall fingerprinting tool

```bash
sudo apt install wafw00f -y
wafw00f -v https://www.tesla.com
```

#### Aquatone

```bash
sudo apt install golang chromium-driver
go install github.com/michenriksen/aquatone@latest
export PATH="$PATH":"$HOME/go/bin"

cat facebook_aquatone.txt | aquatone -out ./aquatone -screenshot-timeout 1000
```

## Virtual Hosts

1. Poke the server to see how it responds to a right and wrong subdomain requests (get the lengths of each response)
   ```bash
   curl -s -I http://192.168.10.10 -H "Host: dev-admin.randomtarget.com"
   ```

2. Create or use a subdomain list (like SecLists/Discovery/DNS/namelist.txt)

   ```bash
   # .vhosts
   app
   blog
   dev-admin
   etc....
   ```

3. Fuzz the subdomain and look for differing lengths
   ```bash
   # Manual, terrible
   cat ./vhosts | while read vhost;do echo "\n********\nFUZZING: ${vhost}\n********";curl -s -I http://192.168.10.10 -H "HOST: ${vhost}.randomtarget.com" | grep "Content-Length: ";done
   
   # Using FFUF
   # Need to see what a 'bad result's size is and filter by that
   ffuf -w ./vhosts -u http://192.168.10.10 -H "HOST: FUZZ.randomtarget.com" -fs 612
   ```

### Virtual Hosts

1. Once you figure out the domain, you can scan for Virtual Hosts:
   ```bash
   gobuster vhost -w ~/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -u http://cybox.company -o vhost.out
   ```

### Files & Directories

1. Scan for folders (http only)

   ```bash
   # OR FFUF
   ffuf -recursion -recursion-depth 1 -u http://192.168.10.10/FUZZ -w ~/opt/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt
   
   # Doesn't scan recursively
   # Remember to adjust the http/https and file extensions
   gobuster dir -q -e -k -u http://10.10.10.10:80/ -x html,htm,txt -w /usr/share/dirb/wordlists/common.txt -o gobuster.txt
   
   # Other extensions to consider
   # Windows: 	jsp,asp,aspx
   # Linux:  	sh
   # Apache:  	php,sql
   # cgi-bin:	cgi
   
   # HTTP only
   wfuzz --field url -R4 --hc 403,404 -w /usr/share/dirb/wordlists/common.txt -u http://10.11.1.115/FUZZ -f wfuzz.txt
   ```

2. If folders were found, make a file containing those folders
   ```bash
   # Create folders.txt with folders found in scan above
   # Ex
   echo wp-admin > folders.txt
   echo wp-content >> folders.txt
   ```

3. Grab keywords from the website
   ```bash
   cewl -m5 --lowercase -w wordlist.txt http://192.168.10.10
   
   # -m5 grabs anything minimum of 5 characters or longer
   # -w makes the wordlist
   ```

4. Combine these files into FFUF to make a more targeted scan
   ```bash
   ffuf -w ./folders.txt:FOLDERS,./wordlist.txt:WORDLIST,~/opt/SecLists/Discovery/Web-Content/raft-small-extensions-lowercase.txt:EXTENSIONS -u http://192.168.10.10/FOLDERS/WORDLISTEXTENSIONS
   
   # Common extensions are found in ~/opt/SecLists/Discovery/Web-Content/raft-*-extensions*
   ```

   

5. If nothing good was found, try expanding the search with another wordlist.  You may not want to include files with this 

   ```bash
   # Remember to adjust the http/https and file extensions
   gobuster dir -q -e -k -u http://10.10.10.10:80/ -x html,htm,txt -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -o gobuster_big.txt
   
   gobuster dir -q -e -k -u http://10.10.10.88:80/webservices -x html,htm,txt -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -o gobuster_big.txt
   # Lists
   ## Expansive list but shorter
   /usr/share/dirb/wordlists/big.txt
   ## Biggest list for directories
   /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
   ## Good tradeoff of number of entries but still not too long (lowecase only)
   /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
   ```
#### If scans aren't working

1. Try changing user agent
   1. Try passive scanning using Burpsuite 
   
      - Dashboard Tab: New Live Task
   
      - Tools Scope: proxy/repeater/intruder
      - URL scope: everything
      - Deduplication: checked to ignore dupes
   
   2. Generic for avoiding blacklists:  "Custom Agent"
   
   3. Firefox Desktop:  "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:82.0) Gecko/20100101 Firefox/82.0"
   
   4. Googlebot: "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" 
      More at https://developers.whatismybrowser.com/useragents/explore/software_name/googlebot/
   
   5. Other agents: https://developers.whatismybrowser.com/useragents/explore/
2. Check if anti-CSRF is in play
3. If it responds to everything, filter by regex on page or by response size

### WordPress

If you find a `wp` folder, be sure to scan it with wpscan and try to bruteforce:

   ```bash
# Enumerate Info
wpscan --url http://10.10.10.10 --enumerate ap,at,cb,dbe --plugins-detection aggressive -o wpscan.txt cli-no-color 

# Enumerate plugins only
wpscan --url http://10.10.10.88/webservices/wp --enumerate p --plugins-detection aggressive
   ```

### Drupal

Try this out next time you find a Drupal site

https://github.com/droope/droopescan

### Webpage Previews

1. Run the go_cutycapt.sh script in the scans directory to create a preview of the discovered URLS.

   NOTE: This probably needs to be tuned, as it depends on the output file type (gobuster,wfuzz,etc)


### Manual Scans

#### WFUZZ 

Automatic recursion (up to 4 levels) of directories only

```bash
wfuzz --field url -R4 --hc 403,404 -w /usr/share/dirb/wordlists/common.txt -u http://10.11.1.115/usage/FUZZ -f wfuzz.txt
```

File search

```bash
wfuzz --field url -R4 --hc 403,404 -w /usr/share/dirb/wordlists/common.txt -u http://10.11.1.10/directory/FUZZ.ext -f wfuzz.txt

# Replace .ext with your desired extension
```

#### Gobuster

Files and directories, no recursion option

```bash
# Adjust directory and file extensions as needed
gobuster dir -e -k -u http://10.11.1.10/directory/ -x php,txt,jsp,asp,aspx,sql,html,htm -w /usr/share/dirb/wordlists/common.txt -o gobuster.txt

# scratch work
gobuster dir -e -k -u http://10.11.1.115/usage/ -x php,txt,jsp,html,htm -w /usr/share/dirb/wordlists/common.txt
```

