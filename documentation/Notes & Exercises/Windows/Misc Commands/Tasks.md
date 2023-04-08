## Task Management

Show tasks

```powershell
# All tasks
tasklist

# Services
tasklist /SVC
```

Kill a process

```powershell
# By PID
taskkill /f /pid PID_NUMBER

# By name
taskkill /f /im "PROCESS_NAME"
```

