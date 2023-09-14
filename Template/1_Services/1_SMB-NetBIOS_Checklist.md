#service 
<% tp.file.rename("1_SMB-NetBIOS_")%>
☑️

# Initial Enumeration
## Enum4Linux
```bash
enum4linux -a $hip
```
## Get SMB version
```bash
crackmapexec smb $hip
```
Alternativ: [grab_smbversion.sh](file:////home/kali/Documents/activeInformationGathering/)
## NSE
```bash
nmap --script "safe and smb-enum-*" -p 445 $hip
```
## Connect RPC
```bash
rpcclient -U "username%passwd" $hip
```
Without creds: `-U "" -N`. Then use `querydispinfo` and `enumdomusers`

# Further Enumeration
## Enum shares as user
```bash
crackmapexec smb $hip -u $user -p "$password" --shares
```
(`$users` can be `$(cat ../0_Users_Hashes.md | cut -d ":" -f 1)`)
Alternativ: `net view \\dc01 /all`
## List files in SMB share
```bash
smbclient --no-pass -c 'recurse;ls' //$hip/$folder
```
## Download all files from smb share
``` bash
smbclient //<IP>/<share>
> mask ""
> recurse
> prompt
> mget *
```
## Check for valid domain credentials
```bash
crackmapexec smb $hip -u /home/kali/oscp/Manuals/9_smb_default_usernames.txt -p /home/kali/oscp/Manuals/9_smb_default_passwords.txt --continue-on-success | grep '+'
```

Find a specific SMB user in many machines
```bash
for i in $(nmap -p 445 192.168.222.1-253 -oG - | grep "open" | cut -d " " -f 2 | tr '\n' ' '); do enum4linux -a "$i" | grep -E "Target|alfred"; done
```


# Findings