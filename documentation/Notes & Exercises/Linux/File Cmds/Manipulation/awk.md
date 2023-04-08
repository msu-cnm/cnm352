## awk - Text Manipulation

Like cut but is actually a programming language designed for text extraction. 

Allows more complex delimiters.

```bash
# ex. Set field separator to :: and print the 1st & 3rd fields to stdout
echo "test1::test2::test3" | awk -F "::" '{print $1, $3}'

-F "::"				# Define field separator, defaults to space (" ")
'{print $1, $3}'	# Defines output, print fields 1 & 3.  The first fields is 1, not 0.  Note, variables in double-quotes are interepreted literally.  To combine variables and strings, enclose strings in double quotes and keep variables outside of the quotes.  Ex. "Testing " $1 $2 $3 " end of my string."
```

