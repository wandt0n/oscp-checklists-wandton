<% tp.file.rename("1_SMB-NetBIOS_")%>

## Get SMB version
```bash
crackmapexec smb $hip
```
Alternativ: [grab_smbversion.sh](file:////home/kali/Documents/activeInformationGathering/)
	

## Enum shares as user
```bash
crackmapexec smb $hip -u $user -p "$password" --shares
```
(`$users` can be `$(cat ../0_Users_Hashes.md | cut -d ":" -f 1)`)
Alternativ: `net view \\dc01 /all`
	

## Check for valid domain credentials
```bash
crackmapexec smb $hip -u /home/kali/oscp/Manuals/7_smb_default_usernames.txt -p /home/kali/oscp/Manuals/7_smb_default_passwords.txt --continue-on-success | grep '+'
```


# Findings