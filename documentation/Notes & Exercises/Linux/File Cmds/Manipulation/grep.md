## grep - Global Regular Expression 

```bash
# Options
-r			# recursive search
-i			# ignore text case
-E			# Extended regex
-v			# Invert match (NOT operator)
-o regex	# ONLY return the string that matches the regex, not the whole line.
-B N		# Show N lines of context BEFORE match
-A N		# Show N lines of context AFTER match
```

### Logic Operations Examples

#### OR

```bash
# Matches pattern1 OR pattern 2
grep 'pattern1\|pattern2' filename
# Another way of doing it
grep -E 'pattern1|pattern2' filename
# Same thing as grep -E
egrep 'pattern1|pattern2' filename
```

#### AND

```bash
# There is no true AND, but it can be simulated
grep -E 'pattern1.*pattern2' filename
# AND - OR combo
grep -E 'pattern1.*pattern2|pattern2.*pattern1' filename
# You can just use multiple greps piped together
grep -E 'pattern1' filename | grep -E 'pattern2'
```

#### NOT

```bash
# Matches anything but pattern1
grep -v 'pattern1' filename
```

