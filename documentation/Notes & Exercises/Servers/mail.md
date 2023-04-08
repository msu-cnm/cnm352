## Mail Server Commands

### POP3

| Command       | Comment                                        |
| ------------- | ---------------------------------------------- |
| USER          | Your user name for this mail server            |
| PASS          | Your password.                                 |
| QUIT          | End your session.                              |
| STAT          | Number and total size of all messages          |
| LIST          | Message# and size of message                   |
| RETR message# | Retrieve selected message                      |
| DELE message# | Delete selected message                        |
| NOOP          | No-op.  Keeps you connection open.             |
| RSET          | Reset the mailbox.  Undelete deleted messages. |

### SMTP

| Command   | Comment                                                      |
| --------- | ------------------------------------------------------------ |
| ATRN      | Authenticated TURN                                           |
| AUTH      | Authentication                                               |
| BDAT      | Binary data                                                  |
| BURL      | Remote content                                               |
| DATA      | The actual email message to be sent This command is terminated with a line that contains only a |
| EHLO      | Extended HELO                                                |
| ETRN      | Extended turn                                                |
| EXPN      | Expand                                                       |
| HELO      | Identify yourself to the SMTP server.                        |
| HELP      | Show available commands                                      |
| MAIL      | Send mail from email account  **MAIL FROM: me@mydomain.com** |
| NOOP      | No-op.  Keeps you connection open.                           |
| ONEX      | One message transaction only                                 |
| QUIT      | End session                                                  |
| RCPT      | Send email to recipient **RCPT TO: you@yourdomain.com**      |
| RSET      | Reset                                                        |
| SAML      | Send and mail                                                |
| SEND      | Send                                                         |
| SOML      | Send or mail                                                 |
| STARTTLS  |                                                              |
| SUBMITTER | SMTP responsible submitter                                   |
| TURN      | Turn                                                         |
| VERB      | Verbose                                                      |
| VRFY      | Verify                                                       |

Src: https://www.suburbancomputer.com/tips_email.htm