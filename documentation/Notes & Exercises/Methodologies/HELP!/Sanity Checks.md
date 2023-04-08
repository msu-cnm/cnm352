## Sanity Checks

### Passwords

1. RETRY PASSWORDS
2. Is the username case sensitive?

### Websites

1. Do you need to re-scan web folders?
2. If you are staring at a page you think should be vulnerable to something, check the source code!  Check for what components its using.

### Reverse Shells

1. Try a lower port like 80/443 in case a firewall is blocking you
2. Try generating payloads for specific architecture (particularly for 64-bit)

### Binaries and Coding

1. Make sure you have a tty shell.  Test this with:

   ```bash
   tty
   # Should show you your pty line or tell you it's not tty
   ```

2. Make sure you properly compiled your binary for the target (32/63 bit)

3. If a shared library doesn't work, try a static one

### Others

1. Re-check for exploits, make sure you didn't miss one or make sure there aren't some that might work but don't match up with your version.

   - Use google if there's a ton of exploits so it will sort by popularity

     ```http
     site:exploit-db.com apache 2.4.7
     ```

2. Read through any exploits that are made for Metasploit and see if you can duplicate them.

3. Google for other exploits written for that particular CVE (maybe add in `github` to the terms)

4. Are you using the right version of python for a python exploit?  Try python2 or 3.

5. If you can't get the version of an application, don't pass on it, look for the simple exploits and try them out.

6. Use strings on suspicious image files!  (`strings -n 10`)

7. If AV is catching you, try using a batch file instead of an executable!