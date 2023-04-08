### Merging Exercise Files

Copying exercises together:

1. Make file list in Windows

   ```powershell
   dir /b > files.txt
   ```

2. Make sure filenames are sorted correctly and adjust

3. Save file list as copy.bat, then do the following:

   1. Find and replace `\r\n` for  `+  ` (surrounded by spaces)
   2. Insert `copy /b ` at the start
   3. Insert `main.md` at the end

4. Run copy.bat to combine the files

5. Verify the file size of the individual parts is equal to the combined

6. Use this Regex on the combined file to fix the headers that are appended to the last line of the previous file

   ```bash
   # NOTE: There is a space at the end of each of these
   Find:  (?<!^)(?<!#)\#\#\# 
   Replace: \r\n### 
   ```

7. Make sure all the asset files are pointed to the current folder

   ```bash
   
   ```

   

