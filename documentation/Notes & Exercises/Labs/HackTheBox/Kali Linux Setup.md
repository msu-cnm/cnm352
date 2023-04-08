Using OffSec's 2020.1 official distro

Kali Securing Steps
	#Change default username/password (deleted kali, created coyote)
	#Install UFW, enable, and start
		sudo apt-get update
		sudo apt-get install ufw
		sudo ufw enable
	#Disable root user's shell
		sudo vi /etc/passwd
		#change shell of root to /usr/sbin/nologin
	#Disable SSH
		#already disabled
	#Disable root ssh login
		sudo vi /etc/ssh/sshd_config
		#Uncomment and set to no
		PermitRootLogin no
	#Restrict root on PAM
		sudo vi /etc/pam.d/login
		#AND
		sudo vi /etc/pam.d/sshd
		#Add lines
		auth    required       pam_listfile.so \
        onerr=succeed  item=user  sense=deny  file=/etc/ssh/deniedusers
		#Create denied users file
		sudo vi /etc/ssh/deniedusers
		#insert 'root' in file
		#set file permissions on it
		sudo chmod 600 /etc/ssh/deniedusers
	#denyhosts - didn't use
	#Bridge network connection instead of NAT

#Regenerate host SSH keys
	cd /etc/ssh/
	mkdir default_kali_keys
	mv ssh_host_* default_kali_keys/
	dpkg-reconfigure openssh-server

#Fix broken sbin paths on Kali 2020.1:
	sudo vi ~\.bashrc
	#add this line
	export PATH=/sbin:/usr/sbin:$PATH
	#Save & exit
	#Verify: 
	echo $PATH