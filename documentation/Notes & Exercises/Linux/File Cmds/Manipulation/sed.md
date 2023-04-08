## sed - Stream Editor 

Think of this as a Find and Replace tool on steroids.  It's a very powerful stream editor, we just use it to manipulate text in files, std out, etc

Always remember, sed is not like grep or cut that disregards everything else.  Sed is a stream EDITOR, so any text you don't deal with will still be there.  When you do replacements, you are replacing whatever you selected with whatever you specify to replace with.  You may need to chain this with grep to accomplish complicated edits in order to filter out unwanted lines.

Options for this command can be found here:  https://www.gnu.org/software/sed/manual/html_node/The-_0022s_0022-Command.html

```bash
# Basic syntax
sed 's/matchstring/replacementstring/flags' filename
# Multiple match/replacements
sed 's/matchstring/replacementstring/flags;s/matchstring/replacementstring/flags' filename
# OR
cat filename | sed 's/matchstring/replacemeqntstring/flags'


# Options
''	# Single quotes are most often used to treat the contents literally.  Sometimes you may want to use a variable, in which case you can use double quotes, but be mindful of how the text will be interepreted by Bash.
s/	# This kicks off the command and declares the separator character (forwardslash in this case).  You can change the separator character but if you use it in the command, you have to escape it with a backslash
```

### Regular Expressions

BIG NOTE:  sed uses basic POSIX regular expressions by default, which means it doesn't support every RegEx command.  I haven't tried it, but allegedly you can use the -E option to support the 'Extended' regular expression commands.

Regular expressions can be used, but remember to escape special characters like parenthesis `()` or forward slashes `/` with a  backslash `\`.  If you want an entire chunk of text to be replaced, your RegEx must match that entire chunk.

You can use parenthesis to create groups with in the match, then refer back to those parenthesis in your replacement statement.  For example, consider this text:

file.txt contains 12345ABCDEF---

```bash
# This statement will select the consecutive digits at the start of the line as group 1, then the consecutive alphas aftter that as group 2, then it will replace that entire selection with group 2.
sed 's/\(^[0-9]*\)\([A-Z]*\)/\2/' file.txt
# Output: ABCDEF---
```

Notice that the dashes at the end are left unchanged because they were not a part of the match.

### Examples

Basic find and replace

```bash
# Find and replace 'hard' with 'harder'
echo "I need to try hard" | sed 's/hard/harder/'
```

Insert text into the beginning of a file

```bash
sed -i '1s/^/TEXT /' FILE
# options
-i	'1s/^/TEXT /'	# Edit file in place, 
					# REGEX: 1 = 1st line, s= regex, ^ = start of line, TEXT = text to insert
```

Search greppable nmap (`*.gnmap`) files for http ports:

```bash
for i in $(ls *.gnmap); do echo $i" http ports:"; sed 's/\([0-9]*\/open\/tcp\/\/http\)/\n\1/g' $i | grep http | sed 's/\(^[0-9]*\)[^\n]*/\1/g'; done

# Explanation of SED options
# First SED command:
s	# Command initator
/\([0-9]*\/open\/tcp\/\/http\)	# Regex matches occurrences of consecutive digits followed by the string '/open/tcp/http'
/\n\1	# Replaces each match with the group (which is the whole match here) and puts it on a newline, breaking up the file into newlines that start with the http ports 
/g		# Tells it to do this for every match, not just the first

# Then we grep this to remove any lines without http on them

# Second SED command:
s	# Command identifier
/\(^[0-9]*\)[^\n]*	# Regex matches on whole lines, creating a group out of the first set of consecutive numbers it finds, which will be the http port
/\1	# Replaces the whole line with just the group containing the http port
/g	# Does this for every match, not just the first
```

