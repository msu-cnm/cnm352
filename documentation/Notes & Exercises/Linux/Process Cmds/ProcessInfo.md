## Processes

- Multitasking handled by processes
- A job is the workflow
- Piped commands are a chain of processes but considered one job
- Job control is the ability to suspend execution of jobs & resume later

### Backgrounding Processes

```bash
# Adding an ampersand (&) to the end of a command sends it to the background immediately and frees up the shell.
# ex.
ping -t 1.1.1.1 &

# Suspend a job while it is active 
CTRL+Z

# To cause a suspended job to run in the background
bg 
```

### Job Controls

```bash
# Terminate an active job
CTRL+C

# Lists all jobs that are running in the current terminal session
jobs

# Return a job from the background or suspended to the foreground
fg 

# Options
# Select the job to run using one of the following:
%NUMBER			# Refers to job number listed by jobs command
%STRING			# Refers to the beginning of the suspended commands name
%+ OR %%		# Refers to the current job
%-				# Refers to the previous job
# If only one process is backgrounded, no specification is needed
```

### Process Controls

```bash
# Process Status (ps) shows processes system wide
# This is a good command to run as a pentester after gaining access to a system
ps -ef
# Options (POSIX Standard Syntax)
-e			# selects all processes
-f			# displays full format listing (UID, PID, PPID, etc)
-C			# Select by command name
--sort -%cpu	# Sort processes by CPU
# These options are commonly used by mistake instead of the BSD options below, without the minus. '-aux' will look for user x, and if not found, the command will assume you mean 'aux'
-a			# selects all processes except both session leaders
-u user		# select process owned by userlist

# Options (BSD Syntax - not recommended)
# BSD Syntax doesn't use a minus in front of the options
a			# lists all processes with a terminal
u			# Display user-oriented format
x			# used with 'a' to list all processes including those without a terminal


ps -aux --sort -%cpu
```

