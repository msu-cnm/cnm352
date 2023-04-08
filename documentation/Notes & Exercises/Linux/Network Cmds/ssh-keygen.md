### ssh-keygen

#### Generate an RSA keypair

```bash
# Runs through a series of prompts to create an RSA keypair
ssh-keygen
# If no path is specified, keys are stored in ~/.ssh/
```

#### Generate a fingerprint for a key file

```bash
ssh-keygen -l -E md5 -f keyfile
# Options
-E md5	# Specifies the fingerprint hash type, default is sha256
```

More info on fingerprinting:  https://weberblog.net/ssh-key-fingerprints/