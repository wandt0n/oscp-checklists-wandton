<% tp.file.rename("2_loot-privEsc_AD_Template")%>

> First, domain controlers should be enumerated as any windows machine. Privilege escalation works the same as well (mostly). Here one can find AD specific enumeration and privilege escalation techniques.

# adPEAS script

# PowerView.ps1

##### Users
```powershell
Get-NetUser | select cn, pwdlastset,lastlogon
````

```powershell
Get-NetGroup | select cn, member
```
	
##### Systems
```powershell
Get-NetComputer | select operatingsystem, operatingsystemversion, dnshostname
```
	
##### Systems the current user has admin privs on
```powershell
Find-LocalAdminAccess
```
	
##### Logged-In users
```powershell
Get-NetSession -ComputerName $host -Verbose
```
(Only works below Windows Server 2019 build 1809 or Windows 10 build 1709)
	
##### AD-integrated services
```powershell
Get-NetUser -SPN | select samaccountname,serviceprincipalname
```
	

##### Shares
```powershell
Find-DomainShare
```
	
Use flag `-CheckShareAccess` to only list shares we have access to.

# Manual approach
##### LDAP path of AD
`$LDAP` = `LDAP://$PDC/$DN`
with Primary DC (PDC):
```powershell
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
```
and DN:
```powershell
([adsi]'').distinguishedName
```

##### Set up `$dirsearcher`
```powershell
$dirsearcher = New-Object System.DirectoryServices.DirectorySearcher(New-Object System.DirectoryServices.DirectoryEntry($LDAP))
```
##### Users
```powershell
$dirsearcher.filter="samAccountType=805306368"
$result = $dirsearcher.FindAll()
Foreach($obj in $result){ $obj.Properties | select {$_.name}, {$_.memberof}, {$_.logoncount} }
```
	
(Never tried it. If it does not work, look at page 701 of the manual for the correct command)
Alternative: `net user /domain` (Make sure that the primary DC is used, see above how to get its address)

##### Logged-In users
```powershell
.\PsLoggedon.exe \\$host
```
(SysInternals, requires _Remote Registry service_ on the target which is disabled by default since Windows 8, but enabled on Windows Server 2012 R2, 2016 (1607), 2019 (1809), and Server 2022 (21H2))
##### Groups
```powershell
$dirsearcher.filter="objectclass=group"
$result = $dirsearcher.FindAll()
Foreach($obj in $result){ $obj.Properties | select {$_.cn}, {$_.member} }
```
	
(Never tried it. If it does not work, look at page 701 of the manual for the correct command)
Alternative: `net group /domain`
List of default groups: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups
To get more details on a group: `$dirsearcher.filter="(&(objectCategory=group)(cn=Sales Department)); $dirsearcher.FindAll()"`

##### AD-integrated services
```powershell
setspn -L $user
```

##### ACEs to an object
```powershell
Get-ObjectAcl -Identity $object [ | ? {"GenericAll", "GenericWrite", "WriteOwner", "WriteDACL" -eq $_.ActiveDirectoryRights} ] | Select ActiveDirectoryRights, SecurityIdentifier
```
	
The middle part restricts the search to full-access but can be left out.
Permission descriptions:
	GenericAll: Full permissions on object
	GenericWrite: Edit certain attributes on the object
	WriteOwner: Change ownership of the object
	WriteDACL: Edit ACE's applied to object
	AllExtendedRights: Change password, reset password, etc.
	ForceChangePassword: Password change for object
	Self (Self-Membership): Add ourselves to for example a group
The SecurityIdentifier names the principal that has the listed permission to the queued object. As object, we can use the cleartext name to any AD object, e.g., an user, group or share name.
Make SID human readable: `Convert-SidToName $sid` (PowerView)

##### Add user to group
```powershell
net group $group $user /add /domain
```

##### Decrypt password set through GPP
```bash
gpp-decrypt "$hash"
```
