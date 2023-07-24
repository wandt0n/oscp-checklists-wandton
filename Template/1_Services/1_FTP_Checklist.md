<% tp.file.rename("1_FTP_")%>
> With IIS: `C:\inetpub\wwwroot`

## Bruteforce default credentials
```
hydra -C /usr/share/wordlists/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt $service://$hip -vV -t 1 -I -s $hport
```
	

# Findings