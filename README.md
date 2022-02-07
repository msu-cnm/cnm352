# CNM352 Lab Files

This repository contains files for CNM 352 Labs.  These files will be automatically downloaded to the Kali Linux VM in NetLab into the folder `~/cnm352-labs` and can be viewed with Typora on Kali.  This will allow for easier copy/pasting of commands.

## WARNING FOR SAVING FILES

A template lab report is provided in this folder that can be used during your lab but BE SURE to copy it somewhere safe (like using https://cl1p.net/ to save it on your local machine) before ending the reservation as this file will not be saved.

Also note that any changes made in the `~/cnm352-labs` subfolders (including the `documentation` or `scripts` folders) on Kali will be discarded automatically on reboot.  The files inside the main `~/cnm352-labs` folder will be safe until you end the reservation.

## Downloading Files

### Manual

You can easily view the files online or manually download the files in the repository, but it will be tedious.  Be sure to get the `.asset` folders that share a name with the documentation text files (which are markdown files ending with `.md`)  which contain the images for that documentation.  That folder needs to be in the same folder as the documentation text file.

### Automatic

You can use git to clone the entire repository (basically all the files on Github in that folder) and then update it as the files change.  

To clone a repository (repo) on Windows:

1. Download git (get the portable version): https://git-scm.com/download/win
2. Install git to some location on your PC.
3. Run this command to create the initial clone of the repo and make your own local repository, replacing it with your install location and the folder you want the repo to be downloaded to: 
   `GIT-INSTALL-FOLDER\bin\git -C "LOCAL-FOLDER" clone https://github.com/msu-cnm/cnm352-labs.git`
4. To resync the directory, run this command:
   `GIT-INSTALL-FOLDER\bin\git -C "LOCAL-FOLDER" pull`
   - If you have made any changes to the files, this will fail and you'll need to discard your changes with this command:
     `GIT-INSTALL-FOLDER\bin\git -C "LOCAL-FOLDER" reset --hard main`

## Reporting Errors

If you find any typos or errors, report them using the `Issues` tab on GitHub (near the top).  
