## MySQL

1. Footprint
   ```bash
   sudo nmap 10.129.231.249 -sV -sC -p3306 --script mysql*
   ```

2. Login
   ```bash
   mysql -u robin -probin -h 10.129.231.249
   # Note: That is -p and the password with no space between
   ```

### Commands

| **Command**                                          | **Description**                                              |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| `mysql -u <user> -p<password> -h <IP address>`       | Connect to the MySQL server. There should **not** be a space between the '-p' flag, and the password. |
| `show databases;`                                    | Show all databases.                                          |
| `use <database>;`                                    | Select one of the existing databases.                        |
| `show tables;`                                       | Show all available tables in the selected database.          |
| `show columns from <table>;`                         | Show all columns in the selected database.                   |
| `select * from <table>;`                             | Show everything in the desired table.                        |
| `select * from <table> where <column> = "<string>";` | Search for needed `string` in the desired table.             |

## MSSQL

Default super account is `sa`

1. Footprinting
   ```bash
   # Nmap
   # This errored out on me when I used it without a user/pass
   sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248
   
   
   
   # Metasploit
   sudo msfconsole
   use auxiliary/scanner/mssql/mssql_ping
   set RHOSTS 1.2.3.4
   run
   ```

2. Connecting
   ```bash
   # I found the .py version and a binary version that both worked the same way
   locate mssqlclient
   
   # Requires creds
   python3 /usr/share/doc/python3-impacket/examples/mssqlclient.py Administrator@10.129.201.248 -windows-auth
   # OR
   impacket-mssqlclient Administrator@10.129.201.248 -windows-auth
   ```

3. 