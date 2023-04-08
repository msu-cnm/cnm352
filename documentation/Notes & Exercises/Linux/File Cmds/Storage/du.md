## du - Disk Usage

Shows usage of folders and files

#### Disk usage per folder

```bash
# Shows usage for all folders in home dir in human readable format (GB, MB, etc)
du -h ~/

# Options
-h 	# Human readable
~/	# Directory to list
```

#### Disk usage cumulative total for folder

```bash
# Shows full output of all usage and total usage at the bottom in human readable format
du -ch ~/

-c 	# cumulative total
```

#### Disk usage of folder by file size

Not sure how this is different from the cumulative total, maybe the other is allocation space, this just file sizes.

```bash
# Shows file size of a directory
du -sh
```

#### Sort by file or folder size

```bash
du ~/ | sort -n -r | less
```

#### Show top x largest files or folders

```bash
du -a / | sort -n -r | head -n 10
```

