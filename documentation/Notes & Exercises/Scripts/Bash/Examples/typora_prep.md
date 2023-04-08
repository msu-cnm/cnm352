### Splitting a large text file into smaller ones, renaming them, and formatting them for Typora

#### Split Files Wherever a Line Starts with a Number

```bash
# Split a file where lines start with a number
sudo csplit --quiet --prefix=SPLIT-FILES -b="%d.md" -z SRC-FILE '/^[0-9]/' {*}
# Options
--prefix=SPLIT-FILES 	# name that split files will be given + serial number
-b="%d.md" 				# Adds an extension to every file of .md
-z SRC-FILE 			# Specify the source file to split
'/^[0-9]/' 				# Regex that matches any line that starts with a digit
{*}						# Tells csplit to create as many files as necessary
```

#### Rename files by the first line of the file

```bash
#!/bin/bash
# Grabs the first word of the file and renames the file to that.
# Usage: ./top_rename.sh <FILE>
# To use wildcards, you must escape the * with a backslash
# ex. ./top_rename KEYWORD\*
for FILE in $1; do 
        NEWNAME=$(head -n 1 $FILE | cut -d " " -f 1);
        mv $FILE $NEWNAME.md;
done
```

#### Insert text into start of the file

```bash
sudo sed -i '1s/^/### /' *.md
```

