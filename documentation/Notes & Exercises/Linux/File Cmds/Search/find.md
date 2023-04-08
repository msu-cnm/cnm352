## find

```bash
find [starting-point] -name PATTERN
# Other options
-type f			# Only Files
-mtime -1		# Modified in last Day
# Note on above, the minus is important because it specifies the last 24hrs vs just using 1 specifies modified exactly 24hrs ago. +1 would mean it was changed at least two days ago because it rounds to the nearest 24hr.
\! 				# invert the next option (NOT)
-user root		# owned by user root
-exec ls -l		# Execute ls -l
```

### Examples

```bash
# Find every file writeable by current user
find / -writable -type d 2>/dev/null
```

```bash
# Find all files with the SUID bit set
find / -perm -u=s -type f 2>/dev/null
```

