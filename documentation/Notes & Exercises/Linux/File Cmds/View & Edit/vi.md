## Visual editor

### Shortcuts

These commands are entered in Command Mode, which can be accessed by pressing ESC to exit any sub-mode you may be in.

| Command | Description                                           |
| ------- | ----------------------------------------------------- |
| u       | Undo last action                                      |
| CTRL+R  | Redo last action                                      |
| v       | select text                                           |
| y       | yank (copy) selected text                             |
| d       | cut selected text                                     |
| p       | paste text in the buffer                              |
| yy      | yank (copy) the line at the cursor position           |
| Nyy     | yank (copy) the next N lines from the cursor position |
| dd      | delete the line at the cursor position                |
| Ndd     | delete N lines from the cursor position               |

### Disable Visual Mode

Anytime I try to highlight text in VI, it goes into this annoying visual mode.  You can disable that by doing the following:

Temporary:  issue the command :set mouse-=a

Permanent:  insert the directive set mouse-=a into your `~/. vimrc` file

### Show Line Numbers

```bash
set number
```

I turned line numbers on permanently by modifying `/etc/vim/vimrc` and inserting this command in there.

### Disable Auto-Tabs

When pasting code from other sources, auto-tabbing can jack up your code.  To manually enable/disable it (it seems backwards, but I guess it's tied to pasting?):

```bash
# Enable auto-tabs
:set nopaste
# Disable auto-tabs
:set paste
```

I modified `/etc/vim/vimrc` and inserted this command to allow turning paste on or off with F2:

```bash
set pastetoggle=<F2>
```



