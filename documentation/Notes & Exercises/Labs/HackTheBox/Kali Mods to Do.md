Kali Mods
[X] Open Burpsuite and click through warnings
[X] Add Burpsuite to favorites
[X] Allow 2 displays
[X] install dirsearch
		git clone https://github.com/maurosoria/dirsearch.git
[X] install php-reverse-shell
		git clone https://github.com/pentestmonkey/php-reverse-shell
	Remove ./local/bin and move to more usable location
[X] Create script for nmap scan
		vi goscan.sh
		ports=$(nmap -p- --min-rate=1000 -T4 $1 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
		nmap -sC -sV -p$ports $1 -oA ~/nmap-scans/$1
		chmod 700 goscan.sh
[X] Add tcpdump tab
[X] Prepopulate FF with 127.0.0.1:8080 proxy server but disable
[X] Keep settings with proxy tab open
[X] Download rockyou.txt
		wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
[X] Adjust timezone
[X] Install pip
		sudo apt install python3-pip
[X] Install Wordpress Scanner
		git clone https://github.com/m4ll0k/WPSeku.git wpseku
		cd wpseku
		sudo pip3 install -r requirements.txt
	Install Windows Reverse Shell
		git clone https://github.com/Dhayalanb/windows-php-reverse-shell.git
[X] Install One-Two Punch
		https://github.com/superkojiman/onetwopunch/blob/master/onetwopunch.sh
[X] Install Unicornscan
		apt install unicornscan
[X] Install Dradis
		sudo apt get dradis
		#Download templates from https://dradisframework.com/academy/industry/compliance/oscp/index.html
		#Follow instructions to import as needed (I did Full Project Export, HTML Template, and Note Templates)
			Path to templates: /usr/lib/dradis/templates/notes/
		Username/Pass: coyote/coyote
		You need to keep spaces above the section headers for them to be read by the app properly
[X] Import OSCP Dradis Sample Project as template
[X] Install tmux & created ~./tmux.conf file (see tmux documentation for contents)
[X] Install tmux logging (see tmux doc for details)
[X] Fix timezone from CLI (it didn't match the GUI)
[X] Created alias ..='cd ..' in .bashrc
[X] Increase command history to 10000 in .bashrc
[X] Added datestamp to command history in .bashrc
[X] Installed Dradis Bulk-Upload https://dradisframework.com/blog/2019/07/new-dradis-script-bulk-upload/