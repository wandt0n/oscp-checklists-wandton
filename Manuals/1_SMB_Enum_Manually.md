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
(`$users` can be users.txt. Fill it with `find .. -name "9*.md" | rev | cut -d "/" -f1 | rev | cut -d "." -f1 | cut -d "_" -f2 | cut -d " " -f 1 > users.txt`)
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
> Be ware for the account lockout threshold! Don't have more then 
```bash
i=$hip; find .. -name "9*.md" | rev | cut -d "/" -f1 | rev | cut -d "." -f1 | cut -d "_" -f2 | cut -d " " -f 1 > users.txt; \
# echo /home/kali/oscp/Manuals/9_smb_default_usernames.txt >> users.txt ; \
find .. -name "9*.md" -exec grep "Password: " {} \; | cut -d " " -f2 > passwords.txt; \
# echo /home/kali/oscp/Manuals/9_smb_default_passwords.txt >> passwords.txt ; \
find .. -name "9*.md" -exec grep "Hash: " {} \; | cut -d " " -f2 > hashes.txt ; \
find .. -name "*.ntlm" -exec cat {} \; >> hashes.txt ; \
sort -u users.txt -o users.txt ; sort -u passwords.txt -o passwords.txt; sort -u hashes.txt -o hashes.txt ; \
sed -i '/^[[:space:]]*$/d' users.txt passwords.txt hashes.txt; \
echo "Going to try $(wc -l hashes.txt passwords.txt | grep total | cut -d " " -f2) logins per user. If this is not within the account lockout threshold, abort now! Hit Enter to continue" ; read a; \
crackmapexec smb $i -u users.txt -H hashes.txt --continue-on-success 2>/dev/null | grep '+' ; \
crackmapexec smb $i -u users.txt -p passwords.txt --continue-on-success 2>/dev/null | grep '+'
```

###### Find a specific SMB user in many machines
```bash
for i in $(nmap -p 445 192.168.222.1-253 -oG - | grep "open" | cut -d " " -f 2 | tr '\n' ' '); do enum4linux -a "$i" | grep -E "Target|alfred"; done
```
