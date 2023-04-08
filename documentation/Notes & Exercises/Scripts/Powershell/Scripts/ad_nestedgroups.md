```powershell
# This script will list members of a given group, then recurse and find nested groups and list their members
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

$max_depth=10

function NestMembers{
	Param ($GroupName, $depth)
    $Searcher.filter="(name=$GroupName)"
    # Run the search and store in $Results
    $Result = $Searcher.FindAll()	
    Foreach($groupMember in $Result)
    {
        "$('	' * $depth)$($groupMember.Properties.name)"
			if ($groupMember.Properties.member -AND $depth -lt $max_depth)
			{
                Foreach($child in $groupMember.Properties.member)
                {
					$childName=($child.substring(3) -split ',')[0]
					NestMembers $childName ($depth + 1)
                }
			}
    }
}

NestMembers secret_group 0
```

