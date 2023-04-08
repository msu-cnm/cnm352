## Compiling Code

### Compiling from Docker

I mad a script to do this easier, make sure to include it in quotes:

```bash
sudo go_gcc.sh "<gcc .....command here>"
```

You can use all these commands below at the end of this command:

```bash
sudo docker run --rm -v "$PWD":/usr/src/myapp -w /usr/src/myapp danielga/gcc-multilib COMPILE_CMDS_HERE

# Ex
sudo docker run --rm -v "$PWD":/usr/src/myapp -w /usr/src/myapp danielga/gcc-multilib gcc -o 18411 18411.c -m32
```

### Compiling for Windows on Windows

Ideally you want to compile code for a platform using that same platform.

1. Install mingw-w64
2. Run `mingw-w64.bat` to setup the path for the gcc executable
3. Run `gcc.exe` to confirm things are working properly.
4. Check any compile instructions in the code, then compile with `gcc code.c -o output.exe`.
5. You may get some warnings, but check for errors.

### Compiling for Windows on Linux

Use mingw-w64 to compile Windows code using Linux

```bash
sudo apt install mingw-w64
```

Compile some code in C for Windows

```bash
i686-w64-mingw32-gcc SourceFile.c -o OutFile.exe
```

Google any errors that show up and resolve them if needed.

Use Wine for Linux to run Windows programs on Linux

### Compiling For Linux on Linux

Note that when compiling for Linux, the kernel version of the machine compiling the code MUST match the kernel version of the intended target.

1. Obtain the source code

2. Compile code

   ```bash
   gcc SourceFile.c -o OutFile
   ```

### Cross-Architecture Compiling

Note: I am not comfortable updating my libraries before the OSCP, so for now I'll just have to compile on the Debian client they give me.

1. Make sure you have the libraries installed

   ```bash
   sudo apt-get install gcc-multilib 
   ```
   
2. Specify the architecture to compile for:

   ```bash
   # 64-bit
   gcc -m64 geek.c -o out
   # 32-bit
   gcc -m32 geek.c -o out
   ```

### Compiling for Older Kernels

- If you're compiling a binary for a really old kernel, you might see the following error when you try to run your executable:

  ```
  ./shared-binary: error while loading shared libraries: requires glibc 2.5 or later dynamic linker
  ```

  If you see that error, it means you need to include the following  command-line argument when you compile & link your executable:

  ```bash
  -Wl,--hash-style=both
  ```

- If you get this error:

  ```bash
  FATAL: kernel too old
  Segmentation fault
  ```

  Then I think the only option is to compile on an older kernel, this is an issue with the gcc libraries themselves

### Static Binaries

When compiling, by default it will create linked binaries, meaning it does not include the dependencies in the binary, so those libraries have to exist on the target.  To statically compile it with all the libraries:

```bash
gcc --static -Wl,--hash-style=both code.c -o binary.elf
```

### Shared Linked Libraries

This didn't help me, but if you compile something and try to run it on a target and it complains about missing libraries, you can include them together with the binary.

1. Compile the binary normally and run this command to show what libraries are needed:

   ```bash
   # It will literally say 'NEEDED'
   readelf -d binary.elf
   ```

2. Create a folder called lib in the same directory as the binary and copy those libraries to it.

3. Copy the binary and that folder of libraries to the target.

4. When running the binary on the target, set the library path:

   ```bash
   LD_LIBRARY_PATH=lib ./binary.elf
   ```

### Examining Binaries & GCC

To see the version of gcc you are using:

```bash
gcc -v
```

Detect the type of binary

```bash
file binary.elf
```

Show library dependencies

```bash
ldd binary.elf
```

### Troubleshooting

```bash
# Error
undefined reference to `_imp__WSAStartup@8'

# Solution
-lws2_32	# Add this option to your compile command to include the Winsock32 library
```

```bash
# Error
76.c:337:5: warning: implicit declaration of function ‘memcpy’ [-Wimplicit-function-declaration]
  337 |     memcpy(&lport[1], &lportl, 2);
      |     ^~~~~~
76.c:337:5: warning: incompatible implicit declaration of built-in function ‘memcpy’
76.c:39:1: note: include ‘<string.h>’ or provide a declaration of ‘memcpy’

# Solution
#include <string.h>		# Add this to your includes
```

```bash
# Error
usr/bin/ld: /tmp/ccNvU9Qk.o: in function `sslread':
29290.c:(.text+0x18b): undefined reference to `SSL_read'
/usr/bin/ld: /tmp/ccNvU9Qk.o: in function `main':
29290.c:(.text+0x7f7): undefined reference to `OPENSSL_init_ssl'
/usr/bin/ld: 29290.c:(.text+0x7fc): undefined reference to `TLS_client_method'
/usr/bin/ld: 29290.c:(.text+0x804): undefined reference to `SSL_CTX_new'
/usr/bin/ld: 29290.c:(.text+0x83f): undefined reference to `SSL_new'
/usr/bin/ld: 29290.c:(.text+0x87f): undefined reference to `SSL_set_fd'
/usr/bin/ld: 29290.c:(.text+0x8a9): undefined reference to `SSL_connect'
/usr/bin/ld: 29290.c:(.text+0xa4e): undefined reference to `SSL_write'

# Solution
-lssl	# Add this option to include the SSL library
```

