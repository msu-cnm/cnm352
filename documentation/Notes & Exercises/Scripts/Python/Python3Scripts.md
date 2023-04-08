## Python3 Scripts

This website is an excellent source of information for Python programming:  https://www.w3schools.com/python/default.asp

### Usage

Python is an interpreted programming language, which means code is run directly from files fed into the python interpreter to execute.  Python also has an interactive interpreter where you can enter code line by line directly.

The general syntax to run code in a file is:

```python
python myfile.py
# or
py myfile.py
```

To access the interpreter prompt, simply omit the filenames:

```python
python
# or
py
```

This will bring you to a `>>>` prompt to let you know it is ready to receive your code.  To exit the prompt, type:

```python
# Exit interactive interpreter
exit()
```

#### Indention Sensitivity

Python is INDENT SENSITIVE.  You will notice that there are no braces or ending statements to delineate blocks of code.  This is because grouping is handled by indentions.

There is no set amount of spaces or tabs, it just has to be indented at least one space, and it has to be consistent for that block of code.

```python
# This is okay
if 1 > 2
	<This is part of the if>
	<so is this>
<not part of if>

# This is not okay, inconsistent spaces within code block
if 1 > 2
 <This is part of the if>
  <so is this>
<but not this>
```

#### Arithmetic Operators

Python supports the standard operators +,-,*,/ 

Modulus is %

### Modules

Modules are packages of members, functions, classes, etc.  To load a module, import the name of the module.  To reference members of the module, type `MODULENAME.MEMBERNAME`.  

```python
# Imports the module named subprocess
import subprocess
# At this point, you'd need to use subprocess.run() to use the run function.
```

To avoid typing the `MODULENAME` for a particular member, import that member from the module explicitly.  This also saves overhead by not importing the entire module library when only a single function is needed.

```python
# Import the run function from the subprocess module
# FYI, this does not require importing the entire module
from subprocess import run
# Now you only need use run() because its been explicitly imported.
```

### Variables

Variables are declared with a simple statement.  Variables do not have fixed types, they match whatever type they've been assigned.  Types can also be changed by assigning a new value of another type to them.

Strings are enclosed in double-quotes.

```python
str_var = "String"
int_var = 100
int_var = "One hundred"
```

Values can be assigned to multiple variables at once

````python
x, y, z = "Eksu", "Why", "Zee"
````

Variable names follow these rules:

- Must start with a letter or underscore
- Cannot start with a number
- Can only contain alpha-numeric characters and underscores
- Are case sensitive

Output a variable value using the `print` command.   Strings can be concatenated with the `+` operator.

```python
str_var = "valuable"
print("str_var equals: " + str_var)
```

#### Lists

#### NEED TO FILL IN

##### Useful List Commands

```
# find unique values in a list
set(list)

# sort a list
sorted(list)

# do both
sorted(set(list))
```

#### Scope

Variables created outside a function are global by default and variables created in a function are local by default.  To declare a global variable inside a function, use the `global` keyword.

```python
def myfunc():
    global globvar = "You"

print("I can see " + globvar)
```

#### Variable Inspection

You can check the type of any variable with the `type` function:

```python
type(VARNAME)
```

In addition, the `inspect` module has a tool to show the members and values of variables:

```python
import inspect
# getmembers shows all members of the variable and the values
inspect.getmembers(VARNAME)
```

More in the inspect module here:  https://docs.python.org/3.8/library/inspect.html

### Arguments

Arguments can be passed to the python script as with any command.  Arguments are processed by python into a list variable called `sys.argv`, with the first item in the list being the script name itself.  The list can be accessed like a normal list variable:

```python
# running python test.py arg1 arg2 arg3
import getopt
len(sys.argv)	# returns the number of arguments + 1, which is just the length of the list
str(sys.argv)	# returns the arguments as a list
str(sys.argv[0])	# returns the script name as a string
str(sys.argv[1])	# returns the 1st argument as a string
```

### If Statements

Remember consistent indentions and ending condition statements with colons.

```python
a = 33
b = 200
if b > a:
  print("b is greater than a")
elif b < a: # Elsif another condition
  print("a is greater than b")
else:
  print ("a is equal to b")
```

There is also a shorthand way of writing if statements:

```python
# Regular if
if a > b: print("a is greater than b")

# if-else - notice the flipping of the actions and conditions
print("A") if a > b else print("B") 

# ternary operator - notice the flipping of the actions and conditions
print("A") if a > b else print("=") if a == b else print("B") 
```

#### Supported Conditions

- Equals: `a == b`
- Not Equals: `a != b`
- Less than: `a < b`
- Less than or equal to: `a <= b`
- Greater than: `a > b`
- Greater than or equal to: `a >= b`

#### Booleans

Multiple comparisons can be combined using boolean statements as well:

```python
# AND
a = 200
b = 33
c = 500
if a > b and c > a:
  print("Both conditions are True")

# OR
a = 200
b = 33
c = 500
if a > b or a > c:
  print("At least one of the conditions is True")
```

### Loops

#### For Loops

```python
# Syntax
for i in LIST:
    <commands>
else:	# Specifies code to run once the loop is finished
    break # Can use a break statement to exit the loop early
    continue # Or use continue to stop the current iteration and skip to the next one 
```

#### Counters

There are different ways to iterate through a For loop.  You can use an array to iterate through the values of the array or you can use the range command to specify a range to iterate through.  

```python
# Using a range
for i in range(N[,M]):  # Specifying one number will cause it to begin with 0 and count up to (N-1).  Specifying two numbers will cause it to begin with N and end with (M-1).
	<commands>
```

#### While loops

While loops use the same conditions and Booleans as `if` statements.  The `else`, `break` and  `continue` statements from `for` loops are also supported here.

```python
i = 1
while i < 6:
  print(i)
  i += 1    
```



### Functions

```python
# Syntax for functions
def myfunc():
    <commands>

```

### Strings

Strings literals are represented by double or single-quotes, each perform the same function `'hello'` and `"hello"` are the same.

#### Printing Strings

Strings are actually arrays of bytes representing unicode characters.  There is no such thing as a character type in Python, so a single character is a string of length 1.

Individual letters of a string can be accessed using the index in square brackets.  The first position begins with 0:

```python
a = "I'm a string"
print(a[0])	# prints 'I'
```

Strings can also be sliced to select portions of the string using indexes, but remember the last index must be one more than the position you wish to capture.  In other words, the index is up to that position, but not including it:

```python
b = "I'm a string too"
print(b[6:12])	# prints 'string'

# Negative indexes start from the end of the string and work backwards
print(b[-10,-4])	# prints 'string'
```

Multi-line strings can be created using three double-quotes (or three single-quotes) to start & end the string:

```python
longstr = """This is a multiline
string.  It really could
have fit on one line tho"""
```

#### Escape Characters

| Code | Result |
| ----------------- | ----------- |
| \\' | Single Quote 	  |
| \\\ | Backslash 	  |
| \n 	| New Line 	      |
| \r 	| Carriage Return |
| \t 	| Tab 	          |
| \b 	| Backspace 	  |
| \f 	| Form Feed 	  |
| \ooo 	| Octal value 	  |
| \xhh 	| Hex value       |

#### Comparing Strings

`string == "test"`

#### String Functions & Methods

Functions

| Function           | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| len(STR)           | Returns the length of a string.  Remember, the last position number will be len() - 1 since positions begin with 0 |
| "CHECK" in STR     | Returns "TRUE" if the CHECK string is found in the STR variable |
| "CHECK" not in STR | Returns "TRUE" if the CHECK string is not found in the STR variable |

```python
#ex:
mystr="abcdefg"
if "abc" in mystr:
	print("TRUE")	
```

Methods

| Method                        | Description                                                  |
| ----------------------------- | ------------------------------------------------------------ |
| STR.strip()                   | Returns the string with any whitespaces from the beginning & end removed |
| STR.lower()                   | Returns the string in all lower case                         |
| STR.upper()                   | Returns the string in all upper case                         |
| STR.replace("FIND","REPLACE") | Returns the string with all instances of the FIND argument replaced with the REPLACE argument |
| STR.split("DELIMITER")        | Returns the string split up based on the supplied DELIMITER as a List |
| STR.rstrip('CHARS')           | Strips off the rightmost end of the string any instances of the specified set of characters.  Defaults to a space. |
| STR.strip('CHARS')            | Strips off any instances of the leading and trailing specified set of charaters.  Defaults to a space. |

All string methods:  https://www.w3schools.com/python/python_ref_string.asp

### Bytes

Bytes are another way text is represented, but are a bit more complicated.  

To create a bytes object

```python
# Option 1
data=b"I'm a byte"
# Option 2
data=bytes("I'm a byte",'utf8')
```

To convert a string to a byte

```python
strvar = "Hello"
# Convert to bytes
strvar = strvar.encode()
```

To convert a byte into a string:

```python
bytevar=b"Goodbye"
# Convert to string
bytevar=bytevar.decode()
```

### Bash Commands

Linux commands can be run by utilizing the `subprocess` module.

A few items that are sprinkled throughout these examples are described below:

- `stdout=subprocess.PIPE` prevents the output from being displayed on the terminal and stores it to the object's stdout member instead.
- `universal_newlines=True` converts the output to a string instead of a byte array.
- `shell=True` causes the string to be interpreted as a raw shell command and is dangerous if allowing for user input as the user could inject anything.
- `check=True` raises an exception if the command returns a non-zero error code.

#### Run & Get Return Code

```python
import subprocess
# Run commands & arguments in the list, redirect stdout
retcode=subprocess.call(["command1","arg1","arg2"],stdout=subprocess.PIPE)
print ("*** Return code was:  ", retcode)
```

Note that anything separated by a space is a separate argument.  So `-t OPTION` is two arguments.

#### Run & Get Output

```python
import subprocess
output=subprocess.check_output(["command","arg1"],universal_newlines=True)
# Does not display stdout, but returns stdout in string format
print ("*** Running command ***\n", output)
```

#### Run & Store Results

```python
import subprocess
process=subprocess.run(["command","arg1"],stdout=subprocess.PIPE,universal_newlines=True)
# Returns object 'subprocess.CompletedProcess'
output=process.stdout
print ("*** Running command ***\n", output)

```

This can also be run using a raw string rather than parameters:

```python
process=subprocess.run('COMMAND AND OPTIONS', shell=True, check=True, stdout=subprocess.PIPE, universal_newlines=True)
# Returns object 'subprocess.CompletedProcess'
output = process.stdout
print ("*** Running command ***\n", output)
```

An older method to do the same:

```python
import subprocess
p=subprocess.Popen(["command","arg1"], stdout=subprocess.PIPE)
output, err=p.communicate()
print ("*** Running command ***\n", output)
```

#### Pipe Commands

```python
from subprocess import Popen,PIPE

# this is equivalent to ls -lha | grep "foo bar"
p1 = Popen(["ls","-lha"], stdout=PIPE)
p2 = Popen(["grep", "foo bar"], stdin=p1.stdout, stdout=PIPE)
p1.stdout.close()

output = p2.communicate()[0]
```

Documentation for subprocess can be found here:  https://docs.python.org/3.8/library/subprocess.html

This page has some good examples, some of which were used above:  https://queirozf.com/entries/python-3-subprocess-examples

### File IO

#### Opening & Closing Files

Use the `open` command to open a file for input/output, specifying the file and the 'mode' you want to open the file in.

```python
# Open the file
fp = open('path/to/file.txt', 'r')

# Close the file
fp.close()
```

The modes are shown below:

| Mode | Description                                                  |
| ---- | ------------------------------------------------------------ |
| r    | Read - Default value. Opens a file for reading, error if the file does not exist |
| a    | Append - Opens a file for appending, creates the file if it does not exist |
| w    | Write - Opens a file for writing, creates the file if it does not exist |
| x    | Create - Creates the specified file, returns an error if the file exists |

The modes above can be combined with these two modes to specify Text or Binary data, although it assumes Text by default, so it's not really necessary to specify that.  For example 'Read Mode for Text', `rt` is the same as `r` but 'Read Mode for Binary' is only `rb`.

| Mode | Description                                                  |
| ---- | ------------------------------------------------------------ |
| t    | Text - Default value. Text mode                              |
| b    | Binary - Binary mode (e.g. images)                           |

It's important to remember to close the file you have opened, as the program will not always do this automatically when the process terminates.  The file can be closed automatically by wrap the code in a `with` statement, which will automatically close the file in the clean-up code once the code-block has completed.

```python
with open('path/to/file.txt','r') as fp:
    # do stuff with fp
```

This older method accomplishes the same thing:

```python
try:
    fp = open('path/to/file.txt')

    # do stuff with fp
finally:
    fp.close()
```

#### Reading Files

There are three common explicit methods to read data.  As with any method, these are referenced by VARNAME.METHOD().

| Method          | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| VAR.read(N)     | Reads N characters from the file, incrementing on each run, and returns a string.  Omitting N will cause it to read the entire file. |
| VAR.readline()  | Reads only one individual line at a time, incrementing on each run, and returns it as a string. |
| VAR.readlines() | Returns the lines of the file as a list of strings.          |

```python
# Example:
with open('path/to/file.txt','r') as fp:
  wholebook=fp.read()
  oneline=fp.readline()
  collectlines=fp.readlines()
print(wholebook)  # Prints the entire document
print(oneline)
```

#### Iterating over lines in a file

```python
with open('path/to/file.txt','r') as fp:
  for line in fp:  
    print(oneline)
```



#### Sending Command Output to a File

```python
import subprocess
from subprocess import Popen

path_to_output_file = '/tmp/myoutput.txt'
myoutput = open(path_to_output_file,'w+')

p = Popen(["ls","-lha"], stdout=myoutput, stderr=subprocess.PIPE, universal_newlines=True)

output, errors = p.communicate()

output
# there's nothing here because we didn't set stdout=subprocess.PIPE

errors
# '' empty string

# stdout has been written to this file
with open(path_to_output_file,"r") as f:
    print(f.read())
```

### User Input

```python
# Accept input from a user and store it in a variable
var = input("This is a prompt: ")
```

### Regular Expressions

Python supports regular expressions with use of the `re` module.

```python
import re
```

Note, when building regular expressions, it's best to use 'raw string' format, otherwise it will be a nightmare messing with special characters. For instance, without raw strings, to match a literal backslash, one might have to write `'\\\\'` as the pattern string, because the regular expression must be `\\`, and each backslash must be expressed as `\\` inside a regular Python string literal.

To further illustrate the difference, `(r"\n")` is a two string character that would match when the literal characters `\n` occur, whereas `("\n")`  is a one string character that matches on newlines.

#### Searching Strings

To search a string for a substring using regular expressions, the `search`, `findall`, or `finditer` methods can be used.  

With `search`, when a match is found, it stops reading the line and returns a `_sre.SRE_Match` object.  To view the match, use the `group` method.

```python
# Example
import re
text = "The #1 film, Jurassic Park, was released in 1993"
# Regex to search for a series of numbers
result = re.search(r"[0-9]+", text)
print(result.group(0))	# output is '1'

#re.match Options:
r 		# specifies raw format (see below)
"[0-9]+"	# regular expression, matches on any size digit
text	# The string to search
```

On the other hand, `findall` will return a list of all non-overlapping matches to the expression.  

```python
# Example
import re
text = "The #1 film, Jurassic Park, was released in 1993"
# Regex to search for a series of numbers
result = re.findall(r"[0-9]+", text)
print(result)	# output is ['1','1998']
```

The `finditer` method creates a 'callable_iterator' class that contains the string as one of its members.  The actual matching string can be accessed through the `VAR.group(0)` method.

```python
# Example
import re
text = "The #1 film, Jurassic Park, was released in 1993"
# Regex to search for a series of numbers
result = re.finditer(r"[0-9]+", text)
for i in result:
  print(i.group(0))	# output is '1' and '1998' on two lines
```

