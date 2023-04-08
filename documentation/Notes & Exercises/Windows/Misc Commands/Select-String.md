## Select-String - Filtering with Context

To filter output for a string and supply lines of context for it:

```powershell
COMMAND | Select-String -Pattern TEXT -Context 6,6

# Options
-Pattern TEXT	# String to search for
-Context 6,6	# Lines of context before and after a match
```

