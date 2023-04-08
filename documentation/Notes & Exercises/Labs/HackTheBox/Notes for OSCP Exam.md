Notes for OSCP Exam
	Get a Burp Suite trial to speed up scans (https://portswigger.net/requestfreetrial/)

From https://medium.com/@parthdeshani/how-to-pass-oscp-like-boss-b269f2ea99d
	Tips For PWK Lab
		1. Set Goal to learn from PWK as much as possible and try to solve machine in different ways.
		2. Never ever set a goal to do all machines but do as good as you can do in lab
		3. Don’t just solve machines from lab but also read Offsec PDF and watch video also it also as important as solving machines. Spend minimum 2 hour for PDF and video everyday
		4. In PWK lab you can get easily exploit just searching on google remember “Google is your Best Friend”
		5. Feel stuck no worries try get help from offsec forms.
		
	My Pre-Exploitation Tools
		Nmap
		Sparta(my Fev)
		Nikto
		Dirbuster or Dirb
		BurpSuit
		Searchsploit
		WFuzz
		Smbmap and Smbclient
		Google ( Best Friend)
		
	My Pre-Exploitation Methodology
		Step 1: I always used nmap to scan my target system and also started scan with Sparta. Because sparta is scan all the port and even UDP port also and Sparta run many automatically tools that scan is very helpful like nikto and smbenum etc.
	
		Step 2: Always check all open ports and try to enumerate there service and versions and collect as much as information as you can using google
	
		Step 3: Then i tried Nikto and Dirbuster for finding something juicy and most of the time they didn’t disappointing me
	
		Step 4: After following all step still i didn’t get anything then I looked for SQL injection or try to find XML or any other vulnerability using burp this step also I follow while doing bug hunting.
	
	My Post-Exploitation Methodology
	Linux:
		Step 1: Run LinuEnum.sh or linuxprivcheck.py scripts.
		Step 2: my 1st preference is always looking for uncommon SUID permission and “sudo -l” command.
		Step 3: Enumeration uncommon files folders and /var directory
		Step 4: Look for programs, services and cronjobs.
	Windows:
		Step 1: Check net user and admin and user rights
		Step 2: Check if we have access of powershell if yes then run powerup.ps1,sherlock.ps1 and JAWS.ps1.
		Step 3: Try to get Meterpreter.
		Step 4: Load mimikatz,try bypass UAC, check SAM SYSTEM etc.
		Step 5: check for weird programs and registry.
	
	Exam Point Distribution
		1 machine which having 10 point (Easy).
		2 machines which having 20 point (Medium).
		2 machines which having 25 points one of them is Buffer overflow (Easy) another one is really hard
	
	Exam Strategy
		1) Start scans on all other machines with sparta & nmap
		2) Work on buffer overflow (25pt) machine while scans are running
		3) Tackle easy machine (10pt)
		4) Feel out the medium machines (20pt), or possibly check out the hard machine (25pt)
		5) Get as far as you can, then use the Metasploit option

Questions to ask about exam
	Clarification on 'no automated tools' rule
		Sparta, One-Two-Punch
		