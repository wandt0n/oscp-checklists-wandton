Already integrated into [[0_SystemChecklist_Windows]]. If further enum is needed, this can help:
# Further Enumeration

##### NSE
```bash
nmap --script "safe and smb-enum-*" -p 445 $hip
```
##### rpcinfo
```bash
rpcinfo -p $hip
```
Look for nfs, ypbind and ruserd. Ignore nlockmgr, status, portmapper, and mountd
##### Enum shares as user
```bash
crackmapexec smb $hip -u $user -p "$password" --shares
```
(`$users` can be `$(find . -name "9*.md" -print | cut -d "/" -f2 | cut -d "." -f1 | cut -d "_" -f2 | cut -d " " -f 1)`)
Alternativ: `net view \\dc01 /all`
##### List files in SMB share
```bash
smbclient --no-pass -c 'recurse;ls' //$hip/$folder
```
##### Download all files from smb share
``` bash
smbclient //<IP>/<share>
> mask ""
> recurse
> prompt
> mget *
```
##### Check for valid domain credentials
```bash
crackmapexec smb $hip -u /home/kali/oscp/Manuals/9_smb_default_usernames.txt -p /home/kali/oscp/Manuals/9_smb_default_passwords.txt --continue-on-success | grep '+'
```
###### Find a specific SMB user in many machines
```bash
for i in $(nmap -p 445 192.168.222.1-253 -oG - | grep "open" | cut -d " " -f 2 | tr '\n' ' '); do enum4linux -a "$i" | grep -E "Target|alfred"; done
```
