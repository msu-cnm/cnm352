## PyInstaller

Create Python executables that bundle the script with the python binaries.

Note:  You must compile from a machine that is the same platform as the target.

Install PyInstaller from PyPI:

```python
pip install pyinstaller
```

Go to your program’s directory and run:

```python
pyinstaller yourprogram.py -F
```

This will generate the bundle in a subdirectory called `dist`.

