## Domain Passive Enumeration

You will need to the JSON processor and an API key for Shodan
```bash
sudo apt install jq
```

1. Build a list of subdomains for a domain (inlanefreight.com) into subdomainlist using Cert transparency logs on crt.sh
   ```bash
   # Install JSON processor (if needed)
   
   curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u > subdomainlist
   ```

2. Look up each one and find the IP to determine which are hosted in the organization and which are 3rd party
   ```bash
   for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done > domain2ip
   ```

3. Create a list of IP addresses for Shodan
   ```bash
   for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt; done
   ```

4. Look up on Shodan
   ```bash
   # Make sure Shodan is connected with your API key
   # shodan init <api key>
   for i in $(cat ip-addresses.txt);do shodan host $i;done
   ```

5. Look up DNS records for more hosts
   ```bash
   dig any inlanefreight.com
   ```

## WayBack Machine

Inspect Old URLS

1. Install Wayback Machine Search

   ```bash
   go install github.com/tomnomnom/waybackurls@latest
   ```

2. Search for URL snapshots by date
   ```bash
   waybackurls -dates https://facebook.com > waybackurls.txt
   cat waybackurls.txt
   ```

   
