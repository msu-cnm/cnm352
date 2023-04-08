## SSH

1. Footprinting
   ```bash
   git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
   ./ssh-audit.py 10.129.14.132
   ```

2. Change Auth Method & Bruteforce
   ```bash
   # Make it use password auth and then try wordlist
   ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password
   ```
   
3. Key-based auth
   ```bash
   # Have to secure private key or SSH won't let you use it
   chmod 700 id_rsa
   
   # login to server using private key as auth mechanism
   ssh -i id_rsa ceil@10.129.92.184
   ```

   

## Rsync

Port 873

1. Footprinting
   ```bash
   sudo nmap -sV -p 873 127.0.0.1
   ```

2. Probe for shares
   ```bash
   nc -nv 127.0.0.1 873
   ```

3. Enumerate a share
   ```bash
   rsync -av --list-only rsync://127.0.0.1/dev
   ```

## R-Services

Uses `cat /etc/hosts.equiv`  system-wide and `.rhosts` in a user profile to specify who can connect and lets them connect without authenticating

| **Command** | **Service Daemon** | **Port** | **Transport Protocol** | **Description**                                              |
| ----------- | ------------------ | -------- | ---------------------- | ------------------------------------------------------------ |
| `rcp`       | `rshd`             | 514      | TCP                    | Copy a file or directory bidirectionally from the local system to  the remote system (or vice versa) or from one remote system to another.  It works like the `cp` command on Linux but provides `no warning to the user for overwriting existing files on a system`. |
| `rsh`       | `rshd`             | 514      | TCP                    | Opens a shell on a remote machine without a login procedure. Relies upon the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files for validation. |
| `rexec`     | `rexecd`           | 512      | TCP                    | Enables a user to run shell commands on a remote machine. Requires authentication through the use of a `username` and `password` through an unencrypted network socket. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files. |
| `rlogin`    | `rlogind`          | 513      | TCP                    | Enables a user to log in to a remote host over the network. It works similarly to `telnet` but can only connect to Unix-like hosts. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files. |

1. Footprinting
   ```bash
   sudo nmap -sV -p 512,513,514 10.0.17.2
   ```

2. Login with rlogin
   ```bash
   rlogin 10.0.17.2 -l htb-student
   ```

3. List authenticated users
   ```bash
   # From within the target
   rwho
   
   # Remotely
   rusers -al 10.0.17.5
   ```

   