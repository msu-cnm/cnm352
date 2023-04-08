## Simple Port Scan Script

This is a good script to use if nmap is not an option.  It can be uploaded to a host and run from there, changing the target IP.

```bash
#!/bin/bash
host=10.5.5.11
for port in {1..65535}; do
	timeout .1 bash -c "echo >/dev/tcp/$host/$port" &&
		echo "port $port is open"
done
echo "Done"
```

