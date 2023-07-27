<% tp.file.rename("2_loot-privEsc_Linux")%>
Hostname `hostname`
	

## PrivEsc Relevant (linpeas script?)
##### Users
```bash
cat /etc/passwd
```
	
##### Hashes
```bash
cat /etc/shadow 
```
`unshadow passwd shadow > unshadowed.txt` 
	
##### Privileges
```bash
sudo -l
```
Search programs on https://gtfobins.github.io/
	
##### Working Dir for attack
```bash
find / -writable -type d 2>/dev/null
```
	
##### SysInfo
```bash
cat /etc/*-release; uname -a
```
	
##### Files with SUID-Bit set
```bash
find / -perm -u=s -type f 2>/dev/null
```
	
##### Network interfaces
```bash
ip a
```
	
##### Gateways
```bash
/sbin/route
```
	
##### Running processes
```bash
ps aux
```
	
##### Installed programs
```bash
dpkg -l
```
	
##### Scheduled tasks
```bash
ls -lah /etc/cron* ; cat /etc/crontab
```
Are any of those or the scripts within writeable?
	

## Loot for later
##### GPG
```bash
ls /home/*/.gpg/
```
	
##### SSH
```bash
ls /home/*/.ssh/
```
	
##### History files
```bash
cat /home/*/.*_history
```
	
##### Home folder
```bash
ls -al /home/*/; ls -al /root/
```

```bash
find / -user $(id -un) -type f 2>/dev/null | grep -Ev "^/sys|^/run|^/proc"
```
	
##### Test for X-Server
```bash
pidof X
```
Is there e.g. a browser we can grab cookies/passwords from?
	
##### Extract Cookies
```
pasco index.dat
```
	
##### Firewall settings
```bash
sudo iptables -S
```
	
##### Listeners/Connections
```bash
netstat -anp
```
	
Git
```bash
find / -name ".git" 2>/dev/null (cd to the directories and do "git config --list" )
```
	
Mounted Volumes
```bash
cat /etc/fstab; mount; /bin/lsblk
```
	
Drivers
```bash
lsmod
```

```bash
/sbin/modinfo [module]
```
	
If Wordpress:
	Look at `wp-config.php`. It contains hardcoded MySQL credentials


# Findings