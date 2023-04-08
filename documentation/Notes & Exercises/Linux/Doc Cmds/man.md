## man - Manual pages

Man pages are broken up into the following sections and open in section 1 by default:

| Section | Contents                                       |
| ------- | ---------------------------------------------- |
| 1       | User Commands                                  |
| 2       | Programming interfaces for kernel system calls |
| 3       | Programming interfaces to the C library        |
| 4       | Special files such as device nodes and drivers |
| 5       | File formats                                   |
| 6       | Games and amusements such as screen-savers     |
| 7       | Miscellaneous                                  |
| 8       | System administration commands                 |

​	

```bash
# Search man pages
man -k REGEX

# ex. search for man pages with passwd only on the line
man -k ‘^passwd$’
# Produces only entire line matches of 'passwd'

# Go directly to a man page for a command
man X COMMAND
```

