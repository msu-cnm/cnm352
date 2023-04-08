## TheHarvester

A simple-to-use yet powerful and effective tool for early-stage  penetration testing and red team engagements. We can use it to gather  information to help identify a company's attack surface. The tool  collects `emails`, `names`, `subdomains`, `IP addresses`, and `URLs` from various public data sources for passive information gathering.

1. Create a list file of sources to use
   ```bash
   # sources.txt
   baidu
   bufferoverun
   crtsh
   hackertarget
   otx
   projecdiscovery
   rapiddns
   sublist3r
   threatcrowd
   trello
   urlscan
   vhost
   virustotal
   zoomeye
   ```

2. Execute theHarvester using those sources
   ```bash
   export TARGET="facebook.com"
   cat sources.txt | while read source; do theHarvester -d "${TARGET}" -b $source -f "${source}_${TARGET}";done
   ```

3. Extract subdomains found
   ```bash
   cat *.json | jq -r '.hosts[]' 2>/dev/null | cut -d':' -f 1 | sort -u > "${TARGET}_theHarvester.txt"
   ```

4. Merge files
   ```bash
   cat facebook.com_*.txt | sort -u > facebook.com_subdomains_passive.txt
   cat facebook.com_subdomains_passive.txt | wc -l
   ```

   