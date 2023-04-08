# Compare Checksum of a file

## Compare a file to a given hash value

```powershell
if ((Get-FileHash -Algorithm SHA256 .\myfile.zip).Hash.ToLower() -eq "GIVEN_HASH_GOES_HERE") {Write "The file is Good"} else {Write "The file is Bad"}

# If the given hash is all lowercase, you can use this as-is, otherwise change the ToLower() function to ToUpper()
```

## Compare two files' hash values

```bash
# Get the file hashes
$hashSrc = Get-Hash $file -Algorithm "SHA256"
$hashDest = Get-Hash $file2 -Algorithm "SHA256"

# Compare the hashes & note this in the log
If ($hashSrc.HashString -ne $hashDest.HashString)
{
  Add-Content -Path $cLogFile -Value " Source File Hash: $hashSrc does not 
  equal Existing Destination File Hash: $hashDest the files are NOT EQUAL."
}
```

