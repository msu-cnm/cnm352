## xxd - Hex Viewer

View a file in hex

```bash
xxd FILE
```

Produce a plain hex dump of a file

```bash
xxd -p lib_mysqludf_sys.so | tr -d '\n' > lib_mysqludf_sys.so.hex

# Options
-p lib_mysqludf_sys.so	# Specifies a plain hex dump
tr -d '\n'	# Removes the newlines in the output to produce a single string
```

