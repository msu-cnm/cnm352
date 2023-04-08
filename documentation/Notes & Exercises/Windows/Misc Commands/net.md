# NET * Commands

## User Commands

```powershell
# Create a user
net user /add <username> *  # Prompts for password
net user /add <username> <password>  # supply password at CLI (insecure)

# Remote a user
net user /del <username>

# Change password
net user <username> *  # prompts for password
net user <username> <password>  # supply password at CLI (insecure)
```



## Group Commands

```powershell
# Show local groups
net localgroup

# Show users in a group
net localgroup <groupname>

# Create a group
net localgroup <groupname> /add

# Delete a group
net localgroup <groupname> /delete

# Add a user to the group
net localgroup <groupname> /add <username>

# Remove a user from the group
net localgroup <groupname> /delete <username>
```

