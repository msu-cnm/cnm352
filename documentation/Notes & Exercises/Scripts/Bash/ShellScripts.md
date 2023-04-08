### Shell Scripts

#### Basics

A shell script is a plain-text file that contains a series of commands that are executed as if they had been typed at a terminal prompt and whose files usually end with .sh.

##### Shebang

Scripts begin with the 'shebang', a comment on the first line with an absolute path to the interpreter to be used to run the script.  This line is ignored by the interpreter.

```bash
# Regular old shebang
#!/bin/bash
# Or to display debug output:
#!/bin/bash -x
# Debug output will be shown inline with output, denoted by a + for code executed in the current shell and ++ for a subshell
```

Comments are preceded by a hashtag

```bash
# I am a comment
```

##### Execution

A user must have execute permissions on the script file in order to run it.  If the user is the owner of the file, generally the permission would be set to `chmod 700`.  You can also use `chmod +x script.sh`

When executing anything in bash, it must be referenced by path (relative or full) or be found in a directory included in the system path ($PATH).  Therefore, to run scripts in the current directory, precede it with `./` (as in `./script.sh`) so that Bash knows you intend to run the file in the current directory.  This design prevents users from accidentally running something in the current directory when they really intended to run a program that is in the system path elsewhere.

#### Variables

Assigned with VARNAME=VALUE, with no spaces around the equal sign.  Variable values can contain spaces, but it must be enclosed in single or double-quotes first.  Variable names are case sensitive.

##### Special Variables

| Variable  | Description                                                  |
| --------- | ------------------------------------------------------------ |
| $0        | Refers to the Bash script name                               |
| $1 - $9   | Refers to the 1st, 2nd, etc. argument passed to the Bash Script (arguments are delimited by spaces) |
| $#        | Number of arguments passed to the Bash script                |
| $@        | All arguments passed to the Bash Script                      |
| $?        | Displays the exit status of the most recently run process    |
| $$        | The process ID of the current Bash Script                    |
| $USER     | The username of the user running the script                  |
| $HOSTNAME | The hostname of the machine                                  |
| $RANDOM   | Generates a random number                                    |
| $LINENO   | The current line number of the script                        |

#### User Input

Display a prompt to the user and get input:

```bash
read -sp 'Prompt text: ' INPUT_VAR
# Options
-p	'Prompt Text: '	# Specify a prompt with given text
-s 					# Silent input (does not show input)
```

To add line breaks in the prompt:

```bash
read -rep $'Prompt text:\n' INPUT_VAR

#Words of the form $'string' are treated specially.  The word expands to  string, with backslash-escaped characters replaced as specified by  the  ANSI  C  standard.  Backslash escape sequences, if present, are decoded  as follows:

#- (...)
#- \n     new line
#- (...)

#The expanded result is single-quoted, as if the  dollar  sign  had  not  been present.
# A  double-quoted  string  preceded  by a dollar sign ($) will cause the  string to be translated according to the current locale.  If  the  cur-  rent  locale  is C or POSIX, the dollar sign is ignored.  If the string  is translated and replaced, the replacement is double-quoted.
```

#### If Statements

Uses a very specific syntax, including required spaces.  The testing conditions are enclosed square brackets, which are actually a reference to the `test` command (see 'Test Syntax' section for testing options).  When building the test condition, refer to the test command for the syntax within these square brackets.  Also be sure to include the spaces between the brackets and the conditions inside them.

```bash
# General format, NOTE the spaces in the brackets!
if [ <some test> ]
then
	<perform actions>
fi

# Technically correct, but harder to read
if test <some test>
then
	<perform an action>
fi

# Example
a=20
if [ $a -le 20 ]
then
	echo $a
fi
```

You can negate any test by placing an exclamation in front of it (with a space):

```bash
# If NOT the test
if [ ! <some test> ]
then
	<perform actions>
fi
```

One liners

```bash
if [test]; then <cmd>; fi
```

##### else statement

Provides an alternate branch if the test fails.   Notice there is no `then` statement that follows `else`.

```bash
# NOTE the spaces in the brackets!
if [ <some test> ]
then
	<do this if the test is true>
else
	<do this if the test is not true>
fi
```

##### elif statement

Provides multiple tests and multiple branches.  Notice `elif` is followed by a `then` statement.

```bash
# NOTE the spaces in the brackets!
if [ <some test> ]
then
	<do this if the 1st test is true>
elif [ <another test> ]
then
	<do this if the elif test is true>
else
	<do this if none of the tests are true>
fi 
```

#### Case statement

```bash
# Evaluates the value of the Expression/Variable and performs commands based on outcome
case $VARIABLE in

  PATTERN_1)
    STATEMENTS
    ;;

  PATTERN_2)
    STATEMENTS
    ;;

  PATTERN_N)
    STATEMENTS
    ;;

  *)
    STATEMENTS
    ;;
esac
```

#### Loops

##### For Loops

Performs an action for each item in a list.

```bash
# General Format
for VAR in <list>
do
	<action to perform>
done

# Example (with seq command substitution)
for IP in $(seq 1 10)
do
	echo 10.11.1.$IP
done

# Example as one-liner
for IP in $(seq 1 10); do echo 10.11.1.$IP; done
```

##### While Loops

Executes code while an expression is true.

Note:  Look out for 'off-by-one' errors due to incrementing or not incrementing a counter in an expected way.  Always test your script to make sure your results include the intended range.

```bash
# General Format
while [ <some test> ]
do
	<action to perform>
done

# Example - Print IP's from 1 to 10
counter=1
while [ $counter -le 10 ]
do
	echo "10.11.1.$counter"
	((counter++))
done
```

Practical Examples:

```bash
# Loop through a file and perform a command on each line
while read -r LINE; do nc -nv $LINE 110; done < hosts
```

#### Test Syntax

As stated, the square brackets [ ] are actually an alias to the `test` command.  Below are common options that can be used within the test command's brackets:

| Operator                 | Description:  Expression True if...                   |
| ------------------------ | ----------------------------------------------------- |
| ! EXPRESSION             | The EXPRESSION is false. (Note the space after the !) |
| -n "$STRING"             | STRING length is greater than zero                    |
| -z "$STRING"             | STRING length of STRING is zero (empty)               |
| "$STRING1" != "$STRING2" | STRING1 is not equal to STRING2                       |
| "$STRING1" == "$STRING2" | STRING1 is equal to STRING2                           |
| INTEGER1 -eq INTEGER2    | INTEGER1 is equal to INTEGER2                         |
| INTEGER1 -ne INTEGER2    | INTEGER1 is not equal to INTEGER2                     |
| INTEGER1 -gt INTEGER2    | INTEGER1 is greater than INTEGER2                     |
| INTEGER1 -lt INTEGER2    | INTEGER1 is less than INTEGER2                        |
| INTEGER1 -ge INTEGER2    | INTEGER1 is greater than or equal to INTEGER 2        |
| INTEGER1 -le INTEGER2    | INTEGER1 is less than or equal to INTEGER 2           |
| -d FILE                  | FILE exists and is a directory                        |
| -e FILE                  | FILE exists                                           |
| -r FILE                  | FILE exists and has read permission                   |
| -s FILE                  | FILE exists and it is not empty                       |
| -w FILE                  | FILE exists and has write permission                  |
| -x FILE                  | FILE exists and has execute permission                |

##### Checking for Unset Variable

You can also check if a string is not set with `if ${var+:} false;`  or some variation of that.  Basically `${var+something}` is a brace expansion that will produce the string `var` plus whatever is after the `+` if `var` has been set.  If `var` doesn't exist, then it returns `NULL`, which is useful in checking to see if a variable exists or not before trying to use it.  

​	See this page:  https://stackoverflow.com/questions/19097745/what-does-plus-colon-mean-in-shell-script-expressions

#### Boolean Logical Operators

Boolean Logical Operators perform TRUE/FALSE comparisons.  In Bash, these actually have different uses depending on the context.

##### Command Lists

One of these uses is command lists.  Piping is a type of command list where the output of one command is sent to another but command lists also allow execute one command based on whether the previous command failed  or succeeded.  This comparison is based on the return code of the program.  Whenever a program executes, it generally produces some type of output, but it also generates a return code, with a return code of zero (or Boolean TRUE) showing a successful execution and a non-zero return code showing a failure of some sort (Boolean FALSE).

| Operator                   | Description                                                  |
| -------------------------- | ------------------------------------------------------------ |
| <command1> && <command2>   | AND operator.  Only executes command2 if command1 succeeded  |
| <command1> \|\| <command2> | OR operator.  Only executes command2 if command1 failed      |
| <test1> && <test2>         | Returns TRUE if both tests are true and returns FALSE if either test is false. |
| <test1> \|\| <test2>       | Returns TRUE if either tests are true and returns FALSE if both tests are false. |

As shown above, these can also be used with the `test` command.

```bash
# AND Command Example
grep password /myfile && echo "Password found!"

# OR Command Example
grep password /myfile || echo "Password NOT found"

# AND/OR Command Example
grep password /myfile && echo "Password found!" || echo "Password NOT found"

# AND Test Example
if [ $A == '1' ] && [ $B == '2' ]
then
	echo "A = 1 AND B = 2"
fi

# OR Test Example
if [ $A == '1' ] || [ $A == '2' ]
then
	echo "A is 1 OR 2"
fi
```

#### Functions

Technically a bash script is actually one main function.  But functions are defined as sub-routines within a script that can be called multiple times.  There are two general formats which are both correct, just a matter of personal preference:

```bash
# Bash style format
function function_name
	{
commands...
}

# C-style format
# Note, the parenthesis are never used for more than marking the declaration.  Arguments are not defined in them as they would be in C.  The function just takes the arguments as $1, $2, etc like a normal script.
function_name () {
commands...
}

# Calling a function requires just the name, no other special characters:
function_name
```

##### Arguments

Functions can use arguments passed to the script  as well.  This does not require modification of the function declaration, simply use $1, $2, etc as you normally would with a script.

```bash
function function_name
	{
echo "Argument is $1"
}

function_name arg1
# Output
Argument is arg1
```

##### Return Codes

Functions in bash don't have return codes in the traditional sense, but they can return a value by specifying a exit status code with `return`.  This exit status can be retrieved immediately after the function exits with $?  

```bash
return_me() {
	echo "Returning a random value"
	return $RANDOM
}
 return_me
 echo "The function returned the value $?"
```

Using `return` without specifying a value will cause the function to return it's normal exit status code, which is 0 for a successful execution and various numbers for other exit states.

Another method is to define a global variable in the main routine that is modified from within the function but is accessible from the main routine.  This requires setting a variable in the proper scope.

##### Scope

This is the context in which the variable has meaning.  By default, a variable has a global scope, meaning it is accessible from anywhere in the script.  A local variable is one that can only be accessed from the function, block of code, or sub-shell it is defined in.

There are some important behaviors to observe:

- A variable defined in the main script is global to the entire script, including functions
- A variable defined in a function is local to that function and any sub-functions
- Functions can modify local or global variables from parent functions
- You may define a variable using the same name as a global variable by overlaying it as a local variable using the `local` keyword in front of the definition.  Note that local variables are still accessible by sub-functions of the function where they are defined.  All the `local` keyword does is begin the scope of the variable up to the current function level.

To see this behavior in action, see the example script below.

```bash
#!/bin/bash
# Example to illustrate the behavior of local & global variables in nested functions
mainvar_static="I'm the main"
mainvar_overlay="I'm gonna get hidden"

function sub2
{
        echo "****Entering sub2 now..."
 		# Naturally mainvar_static is accessible & remains the same as it wasn't modified
 		echo "mainvar_static is still $mainvar_static"			#I'm the main
		# sub2 is able to access sub1's local mainvar_overlay
		echo "mainvar_overlay is $mainvar_overlay"				#I'm sub1's local mainvar_overlay
		# sub2 is also able to access sub1's sub1var
		echo "sub1var is $sub1var"								#I'm sub1's global sub1var!
        echo "****Modifying mainvar_overlay and sub1var"
        mainvar_overlay="mainvar_overlay by sub2!"
        sub1var="sub2 wuz here"
        echo "now mainvar_overlay is $mainvar_overlay"			#mainvar_overlay by sub2!
        echo "Now sub1var is $sub1var"							#sub2 wuz here
        echo "****exiting sub2 now..."
}

function sub1
{
        echo "****Entering sub1 now...."
        echo "mainvar_static is $mainvar_static" 				#"I'm the main"
        echo "mainvar_overlay was $mainvar_overlay" 			#"I'm gonna get hidden"
        echo "***defining local mainvar_overlay and sub1"
        # All the local command does is set the beginning of the scope of the variable to this level
        # This local variable is still accessible by sub-functions
        local mainvar_overlay="I'm sub1's local mainvar_overlay"
        # No local command is needed here since sub1var was not previously defined.  It behaves just like mainvar_overlay, however, and is accessible by sub-functions
        # Though the local keyword is not necessary, it is good practice to keep from accidentally overwriting a global variable by a local function unless you intend to.
        sub1var="I'm sub1's 'global' sub1var!"
        echo "now mainvar_overlay is $mainvar_overlay"			#I'm sub1's local mainvar_overlay
        echo "and sub1var is $sub1var"							#I'm sub1's 'global' sub1var!
        sub2
		# Changes made in sub2 stick in sub1 because they modified variables in sub1's scope
		echo "now mainvar_overlay is $mainvar_overlay"			#mainvar_overlay by sub2!
        echo "and sub1var is $sub1var"							#sub2 wuz here
        echo "****exiting sub1 now..."
}

sub1
echo "Now mainvar_static is $mainvar_static"					#I'm the main
echo "Now mainvar_overlay is $mainvar_overlay"					#I'm gonna get hidden
```

#### Useful Tricks

##### Brace Expansions

Also known as  Sequence Expression.  Used to quickly generate a list of items in a range.

NOTE:  If you use a variable in a brace expansion, it will evaluate the brace expansion before the variable, meaning `{1..$n}` will be seen as just a string because `$n` is a variable and not an integer when the brace expansion occurs.  

To use variables in a range, use the `seq` command as shown in the for loop examples.

```bash
# Example with digits
for i in {1..10}; do echo $i; done
# Output
1
2
<snip>
10

# Example with characters
for i in {a..z}; do echo $i; done
# Ouput
a
b
<snip>
y
z
```

##### Arithmetic Expansion

Arithmetic expansion and evaluation can be performed on variables by placing them inside double-parenthesis and putting spaces between operators and operands.  

Note: this is is a command and therefore does not return a string, so if desiring to assign a value or show the value using the arithmetic function, it must be preceded by a $.

```bash
# Ex.
(( a = 23 ))	# Sets a to 23
(( a++ ))		# Increments by 1 after expanding (post-increment)
(( a-- ))		# Decrements by 1 after expanding (post-decrement)
(( ++a ))		# Increments by 1 before expanding (pre-increment)
(( --a ))		# Decrements by 1 before expanding (pre-decrement)

# Assignment using the arithmetic function
result=$(( a++ ))
# Printing values of the arithmetic function
echo $(( a++ ))
```

##### Iterating through output

```bash
# Iterate through a file's contents
for i in $(cat filename); do echo $i; done

# Iterate through a directory's contents
for i in $(ls); do echo $i; done
```

### Compiling Bash Scripts

See this page -> https://www.simplified.guide/bash/compile-script

or this -> https://www.linux-magazine.com/Online/Features/SHC-Shell-Compiler