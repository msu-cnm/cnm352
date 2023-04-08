## CherryTree - Note-taking tool

### Importing nmap Results:

Download plugin  https://github.com/sergiodmn/cherrymap

Output nmap results into XML (use `-oX` for XML only or `-oA` for all output formats)

NOTE: The target CherryTree file must be a non-protected XML file, otherwise this utility won't work.

### Usage:

```bash
cherrymap.py [-h] [-ah] [-ap] [-a] [-m dest_file] file

mandatory arguments:
  file             #Nmap XML file to parse into cherry tree

optional arguments:
  -h, --help       #show this help message and exit
  -ah, --allhosts  #add all hosts even when no open ports are detected
  -ap, --allports  #add ports closed or filtered
  -a, --all        #same as '-ah -ap'
  -m, --merge      #Specify a CherryTree destination file in which to write the contents
```

### Examples

Import an nmap scan file

```bash
./cherrymap.py nmap.xml -m Cheerytree.ctd
```

Import all XML files in a directory:

```bash
# This will print the full path of each xml file, then feed it to cherrymap to import into the target CTD file

for i in $(ls -rt ~/oscplab/it/enumeration/nmap/*.xml); do ~/opt/CherryTree/cherrymap/cherrymap.py $i -m ~/oscplab/CherryTree/OSCPLab.ctd; done
```

