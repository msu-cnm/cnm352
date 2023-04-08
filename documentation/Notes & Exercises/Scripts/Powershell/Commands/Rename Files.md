```powershell
ls | Rename-Item -NewName {$_.name -replace "OLD-FILE-NAME-PART","NEW-FILE-NAME-PART"}
# NOTE The Search string in the old-file-name is a REGEX, meaning you have to escape things like parenthesis
# The second part, new-file-name, is just a string and does not use escape characters

# Example
ls | Rename-Item -NewName {$_.name -replace "Treble \(Win7\)","Windows 7 Pro 64-Bit"}
```

