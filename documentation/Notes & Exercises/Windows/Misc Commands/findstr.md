## Findstr - Filter results

Filter output of a command by a search string

```powershell
COMMAND | findstr /i "text"

# Options
/i 		# Case insensitive search
"text"	# String to search for
```

Exclude results by a search string

```bash
COMMAND | findstr /i "text" | findstr /i /v "not_this_text"

# Options
/v		# Excludes matching results
```

