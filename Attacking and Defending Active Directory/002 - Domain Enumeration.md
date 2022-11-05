### Domain Enumeration

![[Pasted image 20210705174803.png]]
```powershell
# Enumerate Domain information
$ADClass=[System.DirectoryServices.ActiveDirectory.Domain]
$ADClass::GetCurrentDomain()
```
![[Pasted image 20210804111840.png]]

Both PowerView and Microsoft's Active Directory module can be used to enumerate a domain. Below are some commands of PowerView and AD module for domain enumeration

To load the Active directory module go into the directory where you have stored the Active directory DLL and type `Import-Module .\Microsoft.ActiveDirectory.Management.DLL`

```powershell
# Get Current Domain
Get-NetDomain # PowerView
Get-ADDomain # ActiveDirecory Module

# Get Object of another domain
Get-NetDomain -Domain moneycorp.local # PowerView
Get-ADDomain -Identify moneycorp.local # ActiveDirectory Module

# Get Domain SID for the current domain
Get-DomainSID # PowerView
(Get-ADDomain).DomainSID # ActiveDirectory Module

# Get Domain Policy of current domain
Get-DomainPolicy # PowerView
(Get-DomainPolicy)."system access" # PowerView

# Get Domain Policy of another domain
(Get-DomainPolicy -domain moneycorp.local)"system access" # PowerView

# Get Domain Controller for current domain
Get-NetDomainController # PowerView
Get-ADDomainController # ActiveDirectory Module

# Get Domain Controllers for another domain
Get-NetDomainController -Domain moneycorp.local # PowerView
Get-ADDomainController -DomainName moneycorp.local -Discover # ActiveDirectory Module

# Get a list of users in the current domain
Get-NetUser  # PowerView
Get-NetUser -Username student1  # PowerView
Get-ADUser -Filter * -Properties * # ActiveDirectory Module
Get-ADUser -Identity student1 -Properties * # ActiveDirectory Module

# Get list of all properties for users in current domain
Get-UserProperty # PowerView
Get-UserProperty -Properties pwdlastset # PowerView
Get-ADuser -Filter * -Properties * | select -First 1 | Get-Member -MemberType *Property | select Name # ActiveDirectory Module
Get-ADuser -Filter * -Properties * | select name, @{expression={[datetime]::fromFileTime($_.pwdlastset)}} # ActiveDirectory Module

# Search for a particular string in a user's attributes
Find-UserField -SearchField Description -SearchTerm "built" # PowerView
Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | select name,Description # ActiveDirectory Module
```

Organization might put decoy users in place so that when you try to enumerate those users it will alert the organization that there has been a breach. Always check the bad password policy and password last set policy . The user whose password is last set 2 to  3 years back or whose bad password count is way too high then that user might be a decoy user

### More Enumeration
```powershell
# Get a list of all computers in current domain
Get-NetComputer # PowerView
Get-ADComputer -Filter * | select Name # ActiveDirectory Module

Get-NetComputer -OperatingSystem "*Server 2016" # PowerView
Get-ADComputer -Filter 'OperatingSystem -like "*Server 2016*"' -Properties OperatingSystem | select Name,OperatingSystem # ActiveDirectory Module

Get-NetComputer -Ping # PowerView
Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName} # ActiveDirectory Module

Get-NetComputer -FullData # PowerView
Get-ADComputer -Filter * -Properties * # ActiveDirectory Module

# Get all yhe group in current domain
Get-NetGroup # PowerView
Get-ADGroup # ActiveDirectory Module

Get-NetGroup -Domain <targetdomain> # PowerView
Get-ADGroup -Filter * | select Name # ActiveDirectory Module

Get-NetGroup -FullData # PowerView
Get-ADGroup -Filter * -Properties * # ActiveDirectory Module

# Get all groups containing the word "admin" in group name
Get-NetGroup *admin* # PowerView
Get-ADGroup -Filter 'Name -like "*admin*"' | select Name # ActiveDirectory Module

# Note that the 'enterprise key admins' and 'enterprise admins' group will only be available at the forest root. The command will be

Get-NetGroup *admin* -Domain moneycorp.local # PowerView

# Get all the members of the domain admins group
Get-NetGroupMember -GroupName "Domain Admins" -Recurse # PowerView
Get-ADGroupMember -Identity "Domain Admins" -Recursive # ActiveDirectory Module

# Get all the group memberships of a user
Get-NetGroup -UserName "student1" # PowerView
Get-ADPrincipalGroupMembership -Identity student1 # ActiveDirectory Module

# List all the local groups on a machine (needs administrator privs on non-dc machines)
Get-NetLocalGroup -ComputerName dcorp-dc.dollarcorp.moneycorp.local -ListGroups # PowerView

# Get members of all the local groups on a machine (needs administrator privs on non-dc machines)
Get-NetLocalGroup -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Recurse # PowerView

# Get Actively logged users on a computer (needs local admin rights on the target)
Get-NetLoggenon -ComputerName <servername> # PowerView

# Get locally logged users on a computer (needs remote registry on the target -started by-default on Server OS)
Get-LoggedonLocal -ComputerName dcorp-dc.dollarcorp.local # PowerView

# Get the last logged user on a computer (needs administrative rights and remote registry on the target)
Get-LastLoggedOn -ComputerName <servername> # PowerView

# Find shares on hosts in current domain
Invoke-ShareFinder -Verbose # PowerView

# Find sensitive files on computers in the domain
Invoke-FileFinder -Verbose # PowerView

# Get all fileservers of the domain
Get-NetFileServer # PowerView
```

### GPO
![[Pasted image 20210805110732.png]]

##### GPO enumeration
This can be done with PowerView as well as with Group Policy Module provided by microsoft
```powershell
# Get list of GPO in current domain
Get-NetGPO # PowerView
Get-GPO -All # GroupPolicy Module

Get-NetGPO -ComputerName dcorp-student1.dollarcorp.moneycorp.local # PowerView
Get-GPResultantSetOfPolicy -ReportType Html -Path C:\Users\Administrator\report.html (Provides RSoP)

# Get GPO(s) which use Restricted Groups or groups.xml for interesting users 
Get-NetGPOGroup # PowerView

# Get users which are in a local group of a machine using GPO
Find-GPOComputerAdmin -ComputerName dcorp-student1.dollarcorp.moneycorp.local # PowerView

# Get machines where the given user is member of a specific group
Find-GPOLocation -UserName student1 -Verbose # PowerView

# Get OUs in a domain
Get-NetOU -Fulldata # PowerView
Get-ADOrganizationalUnit -Filter * -Properties * # ActiveDirectory Module

# Get GPO applied on an OU. Read GPOname from gplink attribute from Get-NetOU
Get-NetGPO -GPOname "{AB306569-220D-43FF-B03B-83E8F4EF8081}" # PowerView
Get-GPO -Guid AB306569-220D-43FF-B03B-83E8F4EF8081 # GroupPolicy module
```

### ACL
![[Pasted image 20210805113748.png]]

![[Pasted image 20210805114014.png]]

![[Pasted image 20210805114036.png]]

##### ACL enumeration
This can be done with PowerView as well as with ActiveDirectory Module provided by microsoft
```powershell
# Get the ACLs associated with the specified object
Get-ObjectAcl -SamAccountName student1 -ResolveGUIDs # PowerView

# Get the ACLs associated with the specified prefix to be used for search
Get-ObjectAcl -ADSprefix 'CN=Administrator,CN=Users' -Verbose # PowerView

# We can also enumerate the ACLs using ActiveDirectory Module but without resolving GUIDs
(Get-Acl 'AD:\CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local').Access # ActiveDirectory Module

# Get the ACLs associated with the specified LDAP path to be used for the search
Get-ObjectAcl -ADSpath "LDAP://CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs -Verbose # PowerView

# Search for interesting ACEs
Invoke-ACLScanner -ResolveGUIDs # PowerView

# Get the ACLs asssociated with the specified path
Get-PathAcl Path "\\dcorp-dc.dollarcorp.moneycorp.local\sysvol" #PowerView
```

### Trusts
![[Pasted image 20210806084824.png]]

##### Trust Directions
![[Pasted image 20210806085011.png]]

![[Pasted image 20210806085123.png]]

##### Trust Transitivity
![[Pasted image 20210806085300.png]]

If domain A has bi-directional trust with domain B and domain B has bi-directional trust with domain C then domain A can access resources of domain C and vice versa

![[Pasted image 20210806085516.png]]

### Domain Trusts
![[Pasted image 20210806090226.png]]

![[Pasted image 20210806090348.png]]

![[Pasted image 20210806090418.png]]

![[Pasted image 20210806090451.png]]

![[Pasted image 20210806090508.png]]

![[Pasted image 20210806090541.png]]

![[Pasted image 20210806090629.png]]

### Forest Trusts
![[Pasted image 20210806090935.png]]

### Domain Trust Mapping
```powershell
# Get a list of all domain trusts for the current domain
Get-NetDomainTrust # PowerView
Get-ADTrust # ActiveDirectory Module

Get-NetDomainTrust -Domain us.dollarcorp.moneycorp.local # PowerView
Get-ADTrust -Identity us.dollarcorp.moneycorp.local # ActiveDirectory Module
```

### Forest Mapping
```powershell
# Get details of the current forest
Get-NetForest # PowerView
Get-ADForest # ActiveDirectory Module

Get-NetForest -Forest eurocorp.local # PowerView
Get-ADForest -Identity eurocorp.local # ActiveDirectory Module

# Get all the domains in current forest
Get-NetForestDomain # PowerView
(Get-ADForest).Domains # ActiveDirectory Module

Get-NetForestDomain -Forest eurocorp.local # PowerView

# Get all global catalogs for the current forest
Get-NetForestCatalog # PowerView
Get-ADForest | select -ExpandProperty GlobalCatalogs # ActiveDirectory Module
Get-NetForestCatalog -Forest eurocorp.local # PowerView

# Map trusts of a forest
Get-NetForestTrust # PowerView
Get-ADTrust -Filter 'msDS-TrustForestTrustInfo -ne "$null"' # ActiveDirectory Module
Get-NetForestTrust -Forest eurocorp.local # PowerView
```

### User Hunting
```powershell
# Find all machines on the current domain where the current user has local admin access
Find-LocalAdminAccess -Verbose
```

The above function queries the DC of the current or provided domain for a list of computers `(Get-NetComputer)` and then use multi-threaded `Invoke-CheckLocalAdminAccess` on each machine

![[Pasted image 20210806093456.png]]

This can also be done with the help of remote administration tools like WMI and powershell remoting . Pretty useful in cases ports (RPC and SMB) used by `Find-LocalAdminAccess` are blocked
See `Find-WMILocalAdminAccess.ps1`

```powershell
# Find local admins on all machines of the domain (needs administrator privs on non-dc machines)
Invoke-EnumerateLocalAdmin -Verbose
```

The above function queries the DC of the current or provided domain for a list of computers `(Get-NetComputer)` and then use multi-threaded `Get-NetLocalGroup` on each machine

![[Pasted image 20210806094754.png]]

```powershell
# Find computers where a domain admin (or specified user/group) has sessions:
Invoke-UserHunter 
Invoke-UserHunter -GroupName "RDPUsers"
```

This function queries the DC of the current or provided domain for members of the given group (Domain Admins by default) using `Get-NetGroupMember`, gets a list of computers `(Get-NetComputer)` and list sessions and logged on users `(Get-NetSession/Get-NetLoggedon` from each machine

To confirm the admin access
```powershell
Invoke-UserHunter -CheckAccess
```

![[Pasted image 20210806095728.png]]

```powershell
# Find computers where a domain admin is logged in
Invoke-UserHunter -Stealth
```

This option queries the DC of the current or provided domain for members of the given group (Domain Admins by default) using `Get-NetGroupMember`, gets a list_only_of high traffic servers (DC,File Servers and Distributed File Servers) for less traffic generation and list sessions and logged on users `(Get-NetSession/Get-NetLoggedon)` from each machine

### Defenses
![[Pasted image 20210806100632.png]]

![[Pasted image 20210806100759.png]]