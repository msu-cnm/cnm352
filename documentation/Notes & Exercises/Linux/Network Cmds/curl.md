## curl - Client URL Tool

This can be used to do a host of things, such as downloading files or transferring over other protocols like SCP, POP, SMTP, etc

```bash
# Download a file at URL and name it FILENAME
curl -o FILENAME URL

# Open a URL, ignoring whether it is secure or not
curl -k https://www.url.com/dir
```

### Specify a user agent, useful when the site is filtering for the agent 

```bash
# Pose as a Google Bot
curl -H "User-Agent: Googlebot/2.0" http://10.11.1.39/robots.txt
```

### Download a file

```bash
curl http://some.url --output some.file
```

### POST form data

Examine the form to find the fields being used (you can use Burp to help too by examining a POST request)

```bash
curl -X POST -F 'username=theguy' -F 'password=something' http://domain.tld/post-to-me.php
```

Another way (used this with XML POST)

```bash
curl --data @hello.txt http://10.10.10.9/xmlrpc.php
```

### Untrusted Certificates

To bypass an untrusted certification, add the `--insecure` or `-k` options

```bash
curl --insecure http://10.10.10.9/index.html
```

### Unsupported Protocol Error

If you get this error:

```bash
curl: (35) error:1425F102:SSL routines:ssl_choose_client_version:unsupported protocol
```

It is because your openssl minimum version is set to use more modern version.  To fix this, you have to modify `/usr/lib/ssl/openssl.conf` and change `MinProtocol` to something like `TSLv1.0` or whatever you need.

Alternately, you can bypass it with this flag too:

```bash
curl --tlsv1.0 http://....
```

