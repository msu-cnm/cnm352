```bash
# Error
bash: ./47165.sh: /bin/sh^M: bad interpreter: No such file or directory
```

Fix:

- Open the file in vi
- Issue the command `:set fileformat=unix`
- Save the file