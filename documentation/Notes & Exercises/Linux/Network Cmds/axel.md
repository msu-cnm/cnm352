## axel - Lightweight Download Accelerator

```bash
# Uses simultaneous download streams for faster download.
axel -a -n X -o FILENAME URL

# Options
-a			# more concise progress bar
-n			# number of parallel connections
-o FILE		# Output filename
-H"STRING"	# Modify the header
# The above is useful if the site is blocking Axel.  You can mimic another tool like wget with: -H"User-Agent: Wget/1.20.3 (linux-gnu)"
```

