## Wine - Windows Emulator

### Installing Wine

Use Wine to run Windows programs on Linux.  Installing wine can be a little tricky:

The first issues was that wine32 wasn't in the repository `E: Package  'wine32' has no installation candidate`.  

This was fixed with these  steps:

1. sudo dpkg --add-architecture i386
2. apt-get update
3. sudo apt-get install wine32

When I fixed that, I got an error about unmet dependencies  `libc6-dev : Breaks: libgcc-9-dev (< 9.3.0-5~) but 9.2.1-22 is to be installed,  E: Error, pkgProblemResolver::Resolve generated breaks, this may be  caused by held packages.`   

This issue was resolved with the following  step:

1. sudo apt-get install gcc-9-base libgcc-9-dev libc6-dev
2. sudo apt-get install wine32

 I did get two errors (shown below) at the end of the installation, but  wine seemed to be working and I was able to execute the .exe exploit. 

```bash
Errors:
E: Could not configure 'libc6:i386'.
E: Could not perform immediate configuration on 'libgcc-s1:i386'. Please see man 5 apt.conf under APT::Immediate-Configure for details. (2)` 	
```

### Using Wine

The command below will run the program.

```bash
wine program.exe
```

