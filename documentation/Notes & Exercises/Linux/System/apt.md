## APT - Advanced Package Tool

APT databases are cached locally.  You should update them before doing operations with APT.

```bash
# Update local APT database
sudo apt update

# Update all installed packages and core system
sudo apt upgrade

# Update a single package
sudo apt-get --only-upgrade install PACKAGE
# OR
sudo apt-get install PACKAGE

# Shows info stored in internal cached package database
# Matches on the package’s description, not the name
apt-cache search PACKAGE

# Show installed packages
apt list --installed

# Clean the apt cache
apt clean

# Show package description
apt show PACKAGE

# Completely remove a package and all associated data
# Without --purge, some modified user config data is left behind
sudo apt remove --purge PACKAGE

# Program used by APT to install packages
# Manually install a debian package file (.deb)
# Does not resolve and install dependencies
sudo dpkg -i FILENAME.DEB
```

### Help With Install

This page has some good info on installing packages:  https://installlion.com/