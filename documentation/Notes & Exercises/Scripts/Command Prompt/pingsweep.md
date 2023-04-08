## Ping Sweep Script

```powershell
for /L %i in (1,1,255) do @ping -n 1 -w 200 10.5.5.%i > nul && echo 10.5.5.%i is up.

# Options
&& echo 10.5.5.%i is up	# Only prints this if the ping succeeds

# Syntax   
FOR /L %%parameter IN (start,step,end) DO command 

# Key
   start       # The first number  
   step        # The amount by which to increment the sequence 
   end         # The last number 

   command     # The command to carry out, including any parameters.
               # This can be a single command, or if you enclose it in (brackets), several commands, one per line.

   %%parameter # A replaceable parameter.  Note: in a batch file use %%parameter (on the command line %parameter)
```

