## SSH

### Non-standard Ports

To specify the port for an ssh connection:

```bash
ssh -p 1234 user@host.com
```

### Legacy Key Exchanges

The new openssh version (7.0+) deprecated certain older keys and is not using them by default.  If a server doesn't support modern key exchanges, you can force the Kali client to offer that key so it will negotiate the connection:

```bash
ssh -oKexAlgorithms=METHOD user@host.com
```

Just pick one of the methods the server says it is offering.

Ref:  https://www.openssh.com/legacy.html

### Specify a Private Key

In some cases if you can get a public/private keypair 

```bash
# Remember that 
ssh -i /path/f1fb2162a02f0f7c40c210e6167f05ca-16858 bob@10.11.1.136 

# Options
-i privatekeyfile	# Tells SSH to use a specific private key if it's not already in your ~/.ssh/ folder
```

Debian OpenSSL Predictable PRNG - The Debian keys that let you find a private key from a public key:  https://github.com/g0tmi1k/debian-ssh

```

