## icacls

Display or modify DACL on files or directories.  Outputs SIDs and permission masks.

| Mask | Permissions             |
| ---- | ----------------------- |
| F    | Full Access             |
| M    | Modify Access           |
| RX   | Read and execute access |
| R    | Read-only access        |
| W    | Write-only access       |

### Syntax

```powershell
icacls <filename> [/grant[:r] <sid>:<perm>[...]] [/deny <sid>:<perm>[...]] [/remove[:g|:d]] <sid>[...]] [/t] [/c] [/l] [/q] [/setintegritylevel <Level>:<policy>[...]]

icacls <directory> [/substitute <sidold> <sidnew> [...]] [/restore <aclfile> [/c] [/l] [/q]]


```

https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/icacls

### Examples

```powershell
# show basic security info of a file
icacls "c:\program files\program\program.exe"
```

