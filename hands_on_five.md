## Hands On 5 - Active Directory Enumeration

### Use CIM cmdlets for all the AD enumeration done in the lecture slides
#### Get the current domain
`Get-CimInstance -namespace root/directory/ldap -class ds_domain | select -ExpandProperty ds_dc`

#### Get the current domain policy	
`Get-CimInstance -namespace root/directory/ldap -class ds_domain | select DS_Lockoutduration, DS_lockoutObservationwindow, DS_lockoutThreshold, DS_maxpwdage, ds_minpwdage, ds_minpwdlength, ds_pwdhistorylength, ds_pwdproperties`

#### Get the domain controller	
`Get-CimInstance -namespace root/directory/ldap -class ds_computer | Where-Object {$_.ds_userAccountControl -eq 532480}`

`Get-CimInstance -namespace root/directory/ldap -query "select * from ds_computer where DS_userAccountControl = 532480"`

#### Get all domain users
`Get-CimInstance -namespace root/cimv2 -class win32_useraccount`

#### Get all domain groups
`Get-CimInstance -namespace root/cimv2 -class win32_group -Filter "Domain = 'home'"`

#### Get group membership of the domain admins group for the current and all trusted domains
`Get-CimInstance -namespace root/cimv2 -class Win32_Groupuser | where {$_.GroupComponent -match "Domain Admins"}  | Foreach-Object {$_.PartComponent}`

#### Get group membership of a particular user
`Get-CimInstance -namespace root/cimv2 -class Win32_Groupuser | where {$_.PartComponent -match "user3"} | ForEach-Object {$_.GroupComponent}`

#### Get all domain computers
`Get-CimInstance -Namespace root\directory\ldap -class ds_computer | select DS_name`

#### Get all non-empty properties of a computer:
`(Get-CimInstance -Namespace root\directory\ldap -class ds_computer | where-object {$_.ds_name -eq "WIN7X64"}).CimInstanceProperties | Foreach-Object {If($_.value -AND $_.name -notmatch "__"){@{ $($_.name) = $($_.value)}}}`

### More Active Directory Enumeration
#### Get list of users (local and domain)
`Get-WmiObject -Namespace root\directory\ldap -Class ds_user | select DS_sAMAccountName,DS_userPrincipalName,DS_memberOf`

#### Get details of a particular user
`Get-WmiObject -Namespace root\directory\ldap -Class ds_user -Filter 'DS_sAMAccountName = "user3"'`

`Get-WmiObject -Namespace root\directory\ldap -Query "select * from ds_user where DS_sAMAccountName='user3'"`

#### Get Associators of for user
`Get-WmiObject -Namespace root\directory\ldap -Class ds_user -Filter 'DS_sAMAccountName = "user3"' | select __RELPATH`

`Get-WmiObject -Namespace root\directory\ldap -Query 'Associators of {ds_user.ADSIPath="LDAP://CN=<user name>,CN=Users,DC=home,DC=local"} where classdefsonly'`

#### List Computers with OS Server 2012
`Get-WmiObject -Namespace root\directory\ldap -Class ds_computer -Filter "DS_operatingSystem like '%2012%'"`

`Get-WmiObject -Namespace root\directory\ldap -Query "select * from ds_computer where DS_operatingSystem like '%2012%'"`

#### List all logged on users
`Get-WmiObject -class win32_loggedonuser`

`Get-WmiObject -class win32_loggedonuser -ComputerName <computer name>`

#### List domains/forests which the current domain trust
`[DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest()`

`[DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`

`[DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().GetAllTrustRelationships()`

#### Get list of computers that the local user has admin privs; surpress error messages
```
$computers = Get-WmiObject -Namespace root\directory\ldap -Class ds_computer | Select ExpandProperty ds_computer

foreach ($computer in $computers) {
  try {
    (Get-Wmiobject -class win32_computersystem -computer $computer).Name
  } catch {
    continue
  }
}
```
