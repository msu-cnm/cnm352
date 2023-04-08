## Finger

Enumerates users on the system, but the one time I did this, it was SLOOOOOW

http://pentestmonkey.net/tools/user-enumeration/finger-user-enum  

```bash
perl finger-user-enum.pl -U /usr/share/seclists/Usernames/Name
s/names.txt -t 10.10.10.76

perl finger-user-enum.pl -U /usr/share/seclists/Usernames/Names/names.txt -t 10.10.10.76
```

The output will be very messy, but maximize the terminal to get rid of the word wrap and look for lines that show the user logged in:

```bash
# ex:
root@10.10.10.76: root     Super-User            pts/3        <Apr 24, 2018> sunday              ..

sammy@10.10.10.76: sammy                 console      <Jul 31 17:59>..

sunny@10.10.10.76: sunny                 pts/3        <Apr 24, 2018> 10.10.14.4          ..
```

