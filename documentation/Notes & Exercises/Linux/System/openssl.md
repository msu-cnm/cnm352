## openssl - SSL Certificate Creation and Management

### Create a self-signed certificate

```bash
openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 362 -out bind_shell.crt
# Options
req			# initiate a new certificate signing request
-newkey	rsa:2048	# # generate a new private key and use RSA encryption with a 2048 bit key length
-nodes		# Store the private key without a passphrase
-keyout FILENAME	# Save the key to a file
-x509		# Output a self-signed cert instead of a certificate signing request (CSR)
-days N		# Set validity period to N days
-out FILENAME	# Save the cert to a file
```

### Merge Certificates into PEM

You can combine the private key and certificate in a single PEM or Privacy Enhanced Mail file, which is a Base64 encoded DER certificate.

```bash
cat bind_shell.key bind_shell.crt > bind_shell.pem
```



