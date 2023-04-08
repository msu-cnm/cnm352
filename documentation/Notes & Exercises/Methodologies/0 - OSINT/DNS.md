## Passive Enumeration

#### Passive Enumeration

[VirusTotal](https://www.virustotal.com/gui/)

1. Saves DNS resolutions it's received
2. Type the domain name into the search bar 
3. Click on the "Relations" tab.
4. Can also use Sublist3r but you need to sign up with Virustotal for an API key
   1. https://github.com/aboul3la/Sublist3r/pull/285


<u>Certificates</u>

1. Visit one of these two sites:
   - https://search.censys.io/certificates
   - https://crt.sh
2. Search for the domain

Or use one of these scripts:

```bash
export TARGET="facebook.com"
curl -s "https://crt.sh/?q=${TARGET}&output=json" | jq -r '.[] | "\(.name_value)\n\(.common_name)"' | sort -u > "${TARGET}_crt.sh.txt"

# curl -s 	Issue the request with minimal output.
# https://crt.sh/?q=<DOMAIN>&output=json 	Ask for the json output.
# jq -r '.[]' "\(.name_value)\n\(.common_name)"' 	Process the json output and print certificate's name value and common name one per line.
# sort -u 	Sort alphabetically the output provided and removes duplicates.

export TARGET="facebook.com"
export PORT="443"
openssl s_client -ign_eof 2>/dev/null <<<$'HEAD / HTTP/1.0\r\n\r' -connect "${TARGET}:${PORT}" | openssl x509 -noout -text -in - | grep 'DNS' | sed -e 's|DNS:|\n|g' -e 's|^\*.*||g' | tr -d ',' | sort -u


```

<u>TheHarvester</u>

Automated tool for gathering info

## Active Enumeration

Refer to DNS under `1 - Enumeration` section of notes