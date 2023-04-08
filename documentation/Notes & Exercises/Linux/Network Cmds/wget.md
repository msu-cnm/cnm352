## wget

```bash
# Download a file at URL and name it FILENAME
wget -O OUTFILE URL

# Download a file from an HTTPS site and avoid certificate errors
wget -q --no-check-certificate URL -O OUTFILE
# Options
-O	# Specify filename to save as
-q	# quiet mode, turn off wget's output
--no-check-certificate	# "Insecure HTTPS" - Don't check the server cert against available certificate authorities and don't require the URL hostname to match the common name presented by the cert.  Only use this on trusted sites.
```

#### Recursively download an entire website

```bash
wget --recursive https://download.openwall.net/pub/projects/john/contrib/win32/pwdump/
```

#### Recursively download a directory of a website and child directories

```bash
wget --recursive --no-parent https://download.openwall.net/pub/projects/john/contrib/win32/pwdump/
```

#### Change User Agent

```bash
wget -U "User-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0; SLCC1; .NET CLR 2.0.50727; Media Center PC 5.0; .NET CLR 3.0.04506)" https://download.openwall.net/pub/projects/john/contrib/win32/pwdump/
```

