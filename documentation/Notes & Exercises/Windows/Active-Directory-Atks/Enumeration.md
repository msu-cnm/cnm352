## Active Directory Enumeration

Assume we have compromised a domain-joined Windows 10 Client and a local admin account on the client.

### Users & Groups

Enumerate domain users and their group memberships to find high-value targets

#### Traditional Approach:  net.exe

##### Users

```powershell
# Enumerate all local accounts
net user
# Enumerate all users in the domain
net user /domain
# Enumerate a specific user (ex. jeff_admin)
net user jeff_admin /domain
```

##### Groups

Net.exe is incapable of showing nested group membership, it will only show user members of groups.

```powershell
# Enumerate domain groups
net group /domain
```

#### Modern Approach: Powershell Walkthrough

Powershell does not contain an actual do-it-all command, but a script can be built to do it.  This section documents the process of building the script for the sake of understanding the components.

NOTE:  You must be logged in as a domain user to use some of these commands.

##### Full Script

NOTE: This is just a walkthrough.  For more details on building Powershell AD Enumeration Scripts, see `Scripts\Powershell\Scripts\ad_enumeration.md`

##### Object Enumeration with Filter

1. Determining the domain name

   ```powershell
   [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
   
   # Options
   [System.DirectoryServices.ActiveDirectory.Domain]	# Using the `Domain` class of the `System.DirectoryServices.ActiveDirectory` namespace.
   ::GetCurrentDomain()	# Method of the Domain class that retrieves the domain object for the currently logged in user. 
   ```

   `Name` gives the name of the Domain

   `PdcRoleOwner` gives the distinguished name of the Domain Controller.

2. Use this command in a Powershell Script that builds the provider path:

   ```powershell
   # Store the domain object
   $domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
   
   # Store the name of the PDC
   $PDC = ($domainObj.PdcRoleOwner).Name
   
   # Use $SearchString to build the provider path
   $SearchString = "LDAP://"
   $SearchString += $PDC + "/"
   # Break down domain name into individual components
   $DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
   $SearchString += $DistinguishedName
   $SearchString
   # Example output of provider path
   # LDAP://DC01.corp.com/DC=corp,DC=com
   ```

3. Now add lines to create the DirectorySearcher object and set its root location.

   ```powershell
   # Create a new object for DirectorySearcher class using our provider path
   $Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)
   
   # Create a DirectoryEntry object to be used as the directory root.  Since no arguments are provided, it will default to the top-most root of AD
   $objDomain = New-Object System.DirectoryServices.DirectoryEntry
   
   # Set the root location of our DirectorySearcher object
   $Searcher.SearchRoot = $objDomain
   ```

4. The code above will return every object in Active Directory, so we want to filter this out by the samAccountType attribute of 805306368, which corresponds to 'All user objects'.  More filter examples [can be found here](https://social.technet.microsoft.com/wiki/contents/articles/5392.active-directory-ldap-syntax-filters.aspx?Sort=MostRecent#Examples)

   ```powershell
   # Set the filter for the DirectorySearcher object
   $Searcher.filter="samAccountType=805306368"
   
   # Initiate the search
   $Searcher.FindAll()
   ```

5. The FindAll() function shows each user, but we would like to refine the search to show the properties of the attributes for each user.  Now we replace the FindAll() function with a For loop to expand each attribute:

   ```powershell
   # Replace $Searcher.FindAll() with this:
   # Run a search and store in $Results
   $Result = $Searcher.FindAll()
   
   Foreach($obj in $Result)
   {
   	Foreach($prop in $obj.Properties)
   	{
   		$prop
   	}
   	
   	Write-Host "-------------------------"
   }
   ```

6. If we want to further refine this enormous amount of output down to a specific user, we can simply modify our DirectorySearcher filter:

   ```powershell
   # Modify $Searcher.filter="samAccountType=805306368"
   $Searcher.filter="name=Jeff_Admin"
   ```

7. This completed script is found in `Scripts\Powershell\Scripts\ad_enumeration.md`

##### Resolving Nested Groups

1. We can reuse the script above to list all groups in the AD structure:

   ```powershell
   # Replace from $Searcher.filter="name=Jeff_Admin" and below:
   # Filter by group objects
   $Searcher.filter="(objectClass=Group)"
   
   # Run the search and store in $Results
   $Result = $Searcher.FindAll()
   
   # For each object in the results, show the name only
   Foreach($obj in $Result)
   {
   	$obj.Properties.name
   }
   ```

2. When finding a group of interest, enumerate all members of that group, which will include any nested groups:

   ```powershell
   # Replace from $Searcher.filter="(objectClass=Group)" and below:
   # Filter by only the Secret_Group group object
   $Searcher.filter="(name=Secret_Group)"
   
   # Run the search and store in $Results
   $Result = $Searcher.FindAll()
   
   # For each object in the results, show the members only
   Foreach($obj in $Result)
   {
   	$obj.Properties.member
   }
   
   # Output is in the DistinguishedName form.  Example:
   # CN=Nested_Group,OU=CorpGroups,DC=corp,DC=com
   # "CN" or "Common Name" is the name of the group member
   ```

   Continue to repeat this process to traverse the nested members

### Logged On Users

Use PowerView from PowerShell Empire, which must be loaded onto the target, then import the module:

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
Import-Module .\PowerView.ps1
```

#### NetWkstaUserEnum

Requires Admin rights on the host

```powershell
# Enumerate users currently logged onto a host
Get-NetLoggedon -ComputerName client251
```

#### NetSessionEnum

```powershell
# Enumerate the active sessions on the DC
Get-NetSession -ComputerName dc01
```

### Service Principal Names

Enumerating service accounts can reveal services that have been integrated with AD.

Can use the same script as above.

Another resource is [Get-SPN](https://github.com/EmpireProject/Empire/blob/master/data/module_source/situational_awareness/network/Get-SPN.ps1)

```bash
# Show all SPN's
Get-SPN -type service -search *
```

### Domain Controller

```powershell
# Enter nslookup
nslookup
# Set the lookup type to all
set type=all
# Show hostname of the DC
_ldap._tcp.dc._msdcs.sandbox.local

# Output
Server:  UnKnown
Address:  10.5.5.30

_ldap._tcp.dc._msdcs.sandbox.local      SRV service location:
          priority       = 0
          weight         = 100
          port           = 389
          svr hostname   = SANDBOXDC.sandbox.local
SANDBOXDC.sandbox.local internet address = 10.5.5.30
```

