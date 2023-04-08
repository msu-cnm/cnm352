## Xclip - Text to Clipboard Tool

This can be used to send output of a command to the clipboard.  Note that there are multiple clipboards and sometimes when you try to paste, it won't paste what you expect because you clipped it to another clipboard.  Most examples tell you to use this selection:

```bash
ls -la | xclip -selection clipboard
```

However, what I found was if you want to share the text with your VM Host or paste in bind shells, use the 'primary' clipboard.  

```bash
ls -la | xclip -selection primary
```

There are some caveats to this and sometimes you just need to try either or.  Sometimes it will paste in one area but not another.   🤷🏻‍♂️