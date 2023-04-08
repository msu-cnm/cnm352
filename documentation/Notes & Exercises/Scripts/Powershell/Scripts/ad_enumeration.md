## Active Directory Enumeration Scripts

### Object Enumeration Template

```powershell
# Queries and builds the AD Provider, then searches the entire AD hierarchy and filters based on the $Searcher.filter and displays a summary of results
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()

$PDC = ($domainObj.PdcRoleOwner).Name

$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName
$SearchString

$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)

$objDomain = New-Object System.DirectoryServices.DirectoryEntry

$Searcher.SearchRoot = $objDomain

# Modify this to change filter
$Searcher.filter="name=Jeff_Admin"

# Modify this area to change how the results are displayed
$Searcher.FindAll()
# End of results section
```

#### Filter Examples

Below are some useful filters I've found.  Just replace the filter with these.  This page documents quite a few as well -> [Here](https://social.technet.microsoft.com/wiki/contents/articles/5392.active-directory-ldap-syntax-filters.aspx#Examples)

```powershell
# All objects
# Just delete the $Searcher.filter line

# name of object (user, object, computer, etc)
# No need to put quotes around names with spaces
$Searcher.filter="(name=Jeff_Admin)"

# User accounts only
$Searcher.filter="(samAccountType=805306368)"

# Group objects only
$Searcher.filter="(objectCategory=group)"

# computers only
$Searcher.filter="(objectCategory=computer)"

# Specify OS of computer
# No need to put quotes around text with spaces
$Searcher.filter="(&(objectCategory=computer)(operatingSystem=*Windows 10*))"

# Search for Service Principle Names (SPN)s
$Searcher.filter="serviceprincipalname=*http*"

```

#### Result Output Examples

Below are some different methods of displaying the output. 

```powershell
# Show one-line summary of results
$Searcher.FindAll()

# Displays 'names' property of each result
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
	$obj.Properties.name
}

# Displays the 'members' property of a result.  Requires a compatible object, like 'groups'
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
	$obj.Properties.member
}

# Displays details for each property of each result
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
	Foreach($prop in $obj.Properties)
	{
		$prop
	}
	
	Write-Host "-------------------------"
}

# Displays details for each property and also searches the 'serviceprincipalname' field and looks
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
	Foreach($prop in $obj.Properties)
	{
		$prop
		Foreach($hostName in ((($prop.serviceprincipalname -split "/")[1]) -split ":")[0])
        {
        	Resolve-DnsName -Name $hostName
        }
	}
	
	Write-Host "-------------------------"
}
```

### LDAP Clauses

An LDAP filter has one or more clauses, each enclosed in parentheses. Each clause evaluates to either True or False. An LDAP syntax filter clause is in the following form: 

(<AD Attribute><comparison operator><value>)

The <AD Attribute> must the the LDAP Display name of an Active Directory  attribute. The allowed comparison operators are as follows: 

| **Operator** | **Meaning**                                |
| ------------ | ------------------------------------------ |
| =            | Equality                                   |
| >=           | Greater than or equal to (lexicographical) |
| <=           | Less than or equal to (lexicographical)    |

Note that the operators  "<" and ">" are not supported.  The <value> in a clause will be the actual value of the Active Directory attribute. The value is not case sensitive and should not be quoted. An example LDAP syntax filter clause is:

(cn=Jim Smith) 

This filters on all objects where the value of the common name (cn) attribute is equal to the string "Jim Smith" (not case sensitive). Filter clauses can be combined using the following operators:

| **Operator** | **Meaning**                            |
| ------------ | -------------------------------------- |
| &            | AND, all conditions must be met        |
| \|           | OR, any of the conditions must be met  |
| !            | NOT, the clause must evaluate to False |

For example, the following specifies that either the cn attribute must be  "Jim Smith", or the givenName attribute must be  "Jim" and the sn attribute must be  "Smith": 

(|(cn=Jim Smith)(&(givenName=Jim)(sn=Smith)))

Conditions can be nested with parentheses, but make sure the parentheses match up.

### Special Characters

The LDAP filter specification assigns special meaning to the following characters:

\* ( ) \ NUL 

The NUL character is ASCII 00. In LDAP filters, these 5 characters should be  escaped with the backslash escape character, followed by the two  character ASCII hexadecimal representation of the character. The following table documents this: 

| **Character** | **Hex Representation** |
| ------------- | ---------------------- |
| *             | \2A                    |
| (             | \28                    |
| )             | \29                    |
| \             | \5C                    |
| Nul           | \00                    |

For example, to find all objects where the common name is  "James Jim*) Smith", the LDAP filter would be: 

(cn=James Jim\2A\29 Smith) 

Actually, the parentheses only need to be escaped if they are unmatched, as above. If instead the common name were "James (Jim) Smith", nothing would need to be escaped. However, any characters, including  non-display and foreign characters, can be escaped in a similar manner  in an LDAP filter. For example, here are a few foreign characters: 

| **Character** | **Hex Representation** |
| ------------- | ---------------------- |
| á             | \E1                    |
| é             | \E9                    |
| í             | \ED                    |
| ó             | \F3                    |
| ú             | \FA                    |
| ñ             | \F1                    |

### Filter on objectCategory and objectClass

When building filters, you can either filter by objectCategory or objectClass, and the name to filter by varies depending on which you pick.  See the table below for more information:

| **objectCategory**   | **objectClass**      | **Result**                          |
| -------------------- | -------------------- | ----------------------------------- |
| person               | user                 | user objects                        |
| person               |                      | user and contact objects            |
| person               | contact              | contact objects                     |
|                      | user                 | user and computer objects           |
| computer             |                      | computer objects                    |
| user                 |                      | user and contact objects            |
|                      | contact              | contact objects                     |
|                      | computer             | computer objects                    |
|                      | person               | user, computer, and contact objects |
| contact              |                      | user and contact objects            |
| group                |                      | group objects                       |
|                      | group                | group objects                       |
| person               | organizationalPerson | user and contact objects            |
|                      | organizationalPerson | user, computer, and contact objects |
| organizationalPerson |                      | user and contact objects            |

References:  https://social.technet.microsoft.com/wiki/contents/articles/5392.active-directory-ldap-syntax-filters.aspx#Examples

### AD Password Enumeration

The object enumeration script can be modified to enumerate passwords as well.  Be mindful of account lockout policies.

```powershell
# Enumerate AD passwords
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()

$PDC = ($domainObj.PdcRoleOwner).Name

$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName

New-Object System.DirectoryServices.DirectoryEntry($SearchString, "jeff_admin", "lab")
```

If the login is successful, a message about the object creation will be displayed.

If the login in unsuccessful, a login error will be displayed along with a message about not being able to create the object. `"The user name or password is incorrect."  `