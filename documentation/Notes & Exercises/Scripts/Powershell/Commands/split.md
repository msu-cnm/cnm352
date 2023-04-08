## split

Splits up a string based on a delimiter

```powershell
$child="CN=Nested_Group,OU=CorpGroups,DC=corp,DC=com"
$childName=($child.substring(3) -split ',')[0]
$childName
# Output
Nested_Group

# Options
.substring(3)	# Starts at index 3 of the string, omitting the first 3 characters
-split ','		# Splits the string into an array of substrings
[0]				# Selects the first element of substring array
```

