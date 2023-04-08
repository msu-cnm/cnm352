# pspy - unprivileged Linux process snooping

You can run `pspy --help` to learn about the flags and their meaning. The summary is as follows:

- -p: enables printing commands to stdout (enabled by default)
- -f: enables printing file system events to stdout (disabled by default)
- -r: list of directories to watch with Inotify. pspy will watch all  subdirectories recursively (by default, watches /usr, /tmp, /etc, /home, /var, and /opt).
- -d: list of directories to watch with Inotify. pspy will watch these directories only, not the subdirectories (empty by default).
- -i: interval in milliseconds between procfs scans. pspy scans  regularly for new processes regardless of Inotify events, just in case  some events are not received.
- -c: print commands in different colors. File system events are not  colored anymore, commands have different colors based on process UID.
- --debug: prints verbose error messages which are otherwise hidden.

The default settings should be fine for most applications. Watching files inside `/usr` is most important since many tools will access libraries inside it.

Some more complex examples:

```
# print both commands and file system events and scan procfs every 1000 ms (=1sec)
./pspy64 -pf -i 1000 

# place watchers recursively in two directories and non-recursively into a third
./pspy64 -r /path/to/first/recursive/dir -r /path/to/second/recursive/dir -d /path/to/the/non-recursive/dir

# disable printing discovered commands but enable file system events
./pspy64 -p=false -f
```

Available at: https://github.com/DominicBreuker/pspy		