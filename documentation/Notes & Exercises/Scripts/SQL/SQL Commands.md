## SQL Commands

### MS SQL

Basically the same as MySQL

### MySQL

Using mysql to connect:

```mysql
# Check the user's privileges
SHOW Grants;

# Check common variables
select @@hostname, @@tmpdir, @@version, @@version_compile_machine, @@plugin_dir;

# Check all variables
show variables;

# show the databases
show databases;

# Use a database
use DBNAME;

# Show tables of a database
show tables;

# Show columns of a table
show columns from TABLENAME;

# Show all data in a table
select * from TABLENAME;

# Clear the screen
system clear;
```

#### Good Info

The plugins directory could be exploited if it's non-standard and in a vulnerable spot.

### Oracle

Using sqlplus to connect:

```sql
-- Oracle Server Name
SELECT sys_context('USERENV','SERVER_HOST') server_host FROM dual;

-- Oracle Host Info
SELECT sys_context ('USERENV','db_name') db_name, sys_context ('USERENV','server_host') server_host, sys_context ('USERENV','current_schema') current_schema, sys_context ('USERENV','session_user') current_user, sys_context ('USERENV','host') client_host, sys_context ('USERENV','ip_address') client_ip_address FROM dual;

-- Check Users Privileges
select * from user_role_privs;

-- Show current database schema
SELECT user AS schema_name FROM dual;

-- Switch to different schema
ALTER session SET current_schema = <schema_name>;

----- Databases
-- Show all databases
SELECT username AS schema_name FROM dba_users ORDER BY username;
-- Show databases visible to current user
SELECT username AS schema_name FROM all_users ORDER BY username;
-- Show databases owned by current user
SELECT username AS schema_name FROM user_users ORDER BY username;

----- Tables
-- Show all tables
SELECT table_name FROM dba_tables ORDER BY table_name;
-- Show tables current user has access to
SELECT table_name FROM all_tables ORDER BY table_name;

------ Data
select * from mytable;
```

