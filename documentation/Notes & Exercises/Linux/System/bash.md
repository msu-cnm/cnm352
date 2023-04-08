## bash - Bourne-Again Shell

### Startup Options

Use this option when exploiting SUID to start bash as the filer owner rather than the user calling bash.

```bash
-p  # Turned on whenever the real and effective user ids do not match.  Disables processing of the $ENV file and importing of shell functions.  Turning this option off causes the effective uid and gid to be set to the real uid and gid.
```

### Special Characters

#### $ - Variable Marker

Variables preceded with the $ character will return their value.

```bash
# Example:
test=1234
echo $test
# Output:
1234
```

#### ` - Command Expansion

This is the backtick/grave character (key above the TAB key), not to be confused with a single quote.  When enclosing a command inside backticks, the output of that command is returned as the value.  This allows you to use command output as part of a larger command or output.

NOTE:  This is an old method of command expansion and has been replaced with `$(command)` as the best practice for command expansion.  These two methods differ somewhat in their behavior, but stick with `$(command)`.  

Also note, the command expansion happens in a sub-shell, so unlike sub-functions in bash scripts, any changes to variables will not be saved.

```bash
# Example1, assume pwd is /home/user
test=`pwd`
echo $test
# Output:
/home/usr

# Example2, combined with another command
ls -ls `pwd`
# Output
total 780
drwxr-xr-x 20 coyote coyote   4096 Jun  7 19:53 .
drwxr-xr-x  3 root   root     4096 May 12 14:50 ..
-rw-------  1 coyote coyote  26619 Jun  8 12:20 .bash_history
-rw-r--r--  1 coyote coyote    220 Nov 10  2019 .bash_logout
-rw-r--r--  1 coyote coyote   3511 Jun  5 00:17 .bashrc
-rw-r--r--  1 coyote coyote   3526 Nov 10  2019 .bashrc.original
-rw-r--r--  1 coyote coyote   1245 Jun  6 15:46 bind_shell.crt
-rw-------  1 coyote coyote   1708 Jun  6 15:46 bind_shell.key
-rw-r--r--  1 coyote coyote   2953 Jun  6 15:46 bind_shell.pem
```

#### \ - Escape Character

When needing to print a special character literally (such as wanting to show a dollar sign), it must be escaped by preceding it with the backslash so that bash knows not to interpret what comes next as a variable.

```bash
# Example, assume $cashmoney has not been defined
echo $cashmoney
# Output is empty because the variable isn't defined

# The escape character nullifies the $, so bash doesn't interepret 'cashmoney' as a variable
echo \$cashmoney
# Output
$cashmoney
```

### Quotation Rules

#### Double Quote

Double quotes ( " ) interpolate the data inside them.  Everything inside is taken literally except for items that begin with $, `, or \\.  This results in variables, commands, and escapes to be expanded.  Note that second character is a backtick, not a single quote.

```bash
# Example:
test=1234
echo "Test is $test"
# Output:
Test is 1234
```

#### Single Quote

Single quotes ( ' ) interpret everything contained within them literally.  Items are no interpolated (such as replacing a variable with its value) within single quotes.

```bash
# Example:
test=1234
echo 'Test is $test'
# Output:
Test is $test
```

### Environment Variables

Any unreserved name can be used as a variable.   When setting a value to a variable, ensure there are no spaces before or after the equal sign.  To retrieve the value of a variable, precede its name with a $.

```bash
VAR1=myvalue
echo $VAR1
# output:
myvalue
```

Variables set at the terminal exist only in that terminal session.   To make the variable available to child sessions of that terminal, use the export command.

```bash
# Export an existing variable
export VAR1
# You can also define variable for current shell and any child shells 
export VARIABLE=VALUE
# Show current shell instance PID
echo $$
# Show all environment variables
env
```

The output of a command can be assigned to a variable via command substitution.  Enclose the command in parenthesis preceded by a $.   This can also be accomplished using the backtick/grave character (the key above tab).  Note that command substitution is processed in a subshell, so any variables modified there will not be returned back to the parent shell.

```bash
here=$(pwd)
alsohere=`pwd`
echo $here
echo $alsohere
# output:
/home/user
/home/user
```



`< <( )` is a [Process Substitution](http://mywiki.wooledge.org/ProcessSubstitution) 

`<<<` is a *here-string*. 

### Aliases

Aliases give us a way to create a custom command that does another longer command.

**Warning:  You CAN create an alias that overrides a legitimate command, so be careful.**  

```bash
alias NAME='CMD'

# Example
alias lsa='ls -la'

# Show all aliases
alias

# Remove an alias
unalias ALIAS

```

### Piping and Redirection

POSIX specs state the following:

| Stream Name              | Description                                    |
| ------------------------ | ---------------------------------------------- |
| Standard Input (STDIN)   | Data fed into the program                      |
| Standard Output (STDOUT) | Output from the program (defaults to terminal) |
| Standard Error (STDERR)  | Error messages (defaults to terminal)          |

```bash
# Redirecting from a File
# Send the data from a file to a command as STDIN
COMMAND [OPTIONS] < FILE

# Redirect STDERR to /dev/null which blackholes it and prevents it from being displayed to the screen
COMMAND 2>/dev/null

# Output of command 1 is sent as input to CM2
CMD1 | CMD2
```

### Bash History

```bash
# Show history of commands entered
history
# History Expansion: Re-enter line number N from above history listing to run it
!N
```

#### Disable bash History

1. Disable history for a current shell.

```bash
set +o history
```

2. Clean command history.

```bash
history -c
```

3. Permanently disable bash history.

```bash
echo 'set +o history' >> ~/.bashrc
```

Next time when user login to the shell it will not store any commands to a history file **.bash_history**.

To apply this settings immediately for the current shell session execute the .bashrc file:

```bash
. ~/.bashrc
```

4. Disable a command history system wide:

```bash
echo 'set +o history' >> /etc/profile
```

#### History Environment variables

```bash
HISTSIZE=XXXX # Controls number of commands stored in memory for current session

HISTFILESIZE=XXXX # Configures how many commands are kept in the history file

# By default, history removes duplicate commands and commands beginning with spaces. This HISTCONTROL setting will cause it to only remove duplicates:
HISTCONTROL=ignoredups

# Use HISTIGNORE to filter out basic commands that are run frequently like ls, exit, history, bg, etc. Can take regular expressions too
HISTIGNORE="&:ls:[bf]g:exit:history"

# Control date and/or time stamps in the output of the history command:
HISTTIMEFORMAT='%F %T '
# Options
%F      # Year-Month-Day ISO 8601 format
%T      # 24-hr Time
# more formats are found in strftime man page

```

### Persistent Bash Customization

System-wide bash settings are stored in `/etc/bash.bashrc`

bashrc stands for Bash Run Commands.

Individual users home folders have a `.bashrc` file that overrides these settings

Alias commands can be inserted here to become persistent.

`HISTSIZE & HISTFILESIZE` can also be modified here.
