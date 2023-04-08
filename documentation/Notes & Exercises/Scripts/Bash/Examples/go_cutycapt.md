The initial go_scan script will automatically run a background scan using wfuzz for any webservers found and create a ./enumeration/wfuzz folder with the results.  Once those scans are complete, run this script to generate a snapshot preview and link for all status code 200 index pages in a single web.html file. 

Note: This file must be run from within the wfuzz results directory

```bash
#!/bin/bash
# Usage - Run from a directory with dirb scan results, no arguments needed
mkdir cutycapt
echo "<HTML><BODY><BR>" > ./cutycapt/web.html
for file in $(ls -tr ./wfuzz*txt)
do
  host=$(echo $file | sed 's/\.\/wfuzz_\([^_]*\)_\([0-9]*\).txt/\http:\/\/\1:\2\//g')
  for uri in $(grep C=200 $file | cut -d "\"" -f 2 | sort | uniq -i | sed 's/ - /\//g'); 
  do
    url=$host$uri
    filename=$(echo $url | cut -d ":" -f 2,3 | sed 's/\//_/g').png
    cutycapt --url=$url --out=./cutycapt/$filename
    echo "<h1><a href="$url">$url</a></h1><BR><IMG SRC=$filename width=600><BR>" >> ./cutycapt/web.html
  done
done
  echo "</BODY></HTML>" >> ./cutycapt/web.html
```

