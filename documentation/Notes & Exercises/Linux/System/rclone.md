## rclone - Cloud Sync Tool

### Setup

Google Drive Setup - https://rclone.org/drive/

Encrypted config with same password as coyote user

Setup my own Application ID through DMX account

Used DMX Account for Google Drive

rclone name:  `DMX_Gdrive`

To set a root folder, open the desired folder up in Google Drive in a browser and look at the URL.  If the folder you want rclone to use has a URL which looks like `https://drive.google.com/drive/folders/1XyfxxxxxxxxxxxxxxxxxxxxxxxxxKHCh` in the browser, then you use `1XyfxxxxxxxxxxxxxxxxxxxxxxxxxKHCh` as the `root_folder_id` in the config.

### Use

Copy local directory to drive directory:

```bash
rclone copy /home/coyote DMX_Gdrive:kali_backup
```

Sync a directory

```bash
rclone sync /home/coyote DMX_Gdrive:backups/home/coyote/
```

Updated and modified version:

```bash
rclone -vv --tpslimit 10 -L --exclude-from /root/rclone_exclude.txt sync /home/coyote/ DMX_Gdrive:backups/home/coyote/

# Optiopns
-vv		# Very verbose output
--tpslimit 10	# no idea but supposed to make it go faster
--exclude-from /root/rclone_exclude.txt		# Folder exclusion list
```

