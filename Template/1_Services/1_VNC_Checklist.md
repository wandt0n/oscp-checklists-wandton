<% tp.file.rename("1_VNC_")%>

Use VNC client
```bash
xtightvncviewer $hip
```

```bash
vncviewer [-passwd passwd.txt] $hip::5901
```

NSE Scripts
```bash
nmap -sV --script vnc-info,realvnc-auth-bypass,vnc-title -p $hport $hip
```


### Default password storage locations
- UNIX: ~/.vnc/passwd
- RealVNC: `HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\vncserver` Value: Password
- TightVNC: `HKEY_CURRENT_USER\Software\TightVNC\Server` Value: Password or PasswordViewOnly
- TigerVNC: `HKEY_LOCAL_USER\Software\TigerVNC\WinVNC4` Value: Password
- UltraVNC: `C:\Program Files\UltraVNC\ultravnc.ini` Value: passwd or passwd2

Some passwords may be encrypted with 3des. Decryptor:  [https://github.com/jeroennijhof/vncpwd](https://github.com/jeroennijhof/vncpwd)


# Findings