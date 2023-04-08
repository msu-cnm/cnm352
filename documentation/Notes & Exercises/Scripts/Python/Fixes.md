#### Missing pip packages that were already installed

If running the command as sudo, you have to reinstall the pip package using sudo too

#### Wheel

Error:

```bash
  ERROR: Command errored out with exit status 1:
   command: /usr/bin/python -u -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-AFgpA5/pyip/setup.py'"'"'; __file__='"'"'/tmp/pip-install-AFgpA5/pyip/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' bdist_wheel -d /tmp/pip-wheel-58eOGG
       cwd: /tmp/pip-install-AFgpA5/pyip/
  Complete output (6 lines):
  usage: setup.py [global_opts] cmd1 [cmd1_opts] [cmd2 [cmd2_opts] ...]
     or: setup.py --help [cmd1 cmd2 ...]
     or: setup.py --help-commands
     or: setup.py cmd --help
  
  error: invalid command 'bdist_wheel'
  ----------------------------------------
  ERROR: Failed building wheel for pyip

```

Solution:  pip install wheel

#### Random

Error:

```bash
[+]Program started in Enumeration Mode
[+]Checking for possible enumeration techniques
Traceback (most recent call last):
  File "ikeforce.py", line 379, in <module>
    iCookie = ikeneg.secRandom(8).encode('hex')
  File "/home/coyote/opt/tools/ike/ikeforce/ikeclient.py", line 46, in secRandom
    randomBytes = OpenSSL.rand.bytes(bytes)
AttributeError: 'module' object has no attribute 'rand'
```

Solution:

```bash
pip install 'pyopenssl==17.2.0'
```

#### Python 2.7 Incompatibilities

You may run into scripts that don't work with Python 2.7.  You have two options:

1. Run the script using Python 2.7

   - This may require getting Python 2.7 packages, which requires Pip 2.7 to be explicitly installed

     ```bash
     #Install PIP for Python2.7 (Note, the second step will take a few mins to complete)
     wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
     sudo python2.7 get-pip.py
     
     #Install the requests and termcolor modules for Python2.7
     pip2.7 install MODULENAME
     ```

   - And then running the scripts with Python2.7

     ```bash
     python2.7 SCRIPT.py
     ```

2. Convert the script to Python3

   ```bash
   # Install the 2to3 Converter
   pip install 2to3
   
   # Convert the file
   2to3 -w SCRIPT.py
   # OR 
   2to3-2.7 -w SCRIPT.py
   ```

   Note: this is not perfect by any means and has a LOT of gotchas that may have you pulling hair out for a while, so it is really easier to just use 2.7 natively, tbh.
