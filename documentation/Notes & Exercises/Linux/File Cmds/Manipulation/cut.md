## cut

Cut a section of text and output to stdout. Can only specify a single character for delimiter

```bash
# ex. Set delimiter to colon and cut the 1st field
cut -d ":" -f 1 /path/file

# Options
-d " "			# Sets the delimiting character.  Defaults to TAB
-f N			# Selects which field to cut. Starts with 1, not 0
```

