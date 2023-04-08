## uniq

Prints unique entries in a file

```bash
uniq -c FILE

#Options
-c 	# prints number of occurrences of each unique line
-u	# only prints unique lines
```

To find the number of occurrences in a list and sort it, use this:

```bash
cat * | sort | uniq -c
```

