## FTP Servers

- Check the files in ./enumeration/ftp from the go_scan.sh results.  

   ```bash
   # Display the results of each host
   for file in $(ls -rt); do echo "****************$file*******************"; cat $file; done | less
   ```

   - Check out the directory listings for interesting files
   - Look for `____coyote_write_test____` files, indicating write access to the FTP server

- Settings to turn on once logged in

  1. status
  2. debug

- Get all files at once
  ```bash
  wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
  ```

- 

