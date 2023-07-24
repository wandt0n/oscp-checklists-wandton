<% tp.file.rename("1_SSH_")%>

- [ ] Banner grabbed: 
- [ ] Bad keys? (https://github.com/rapid7/ssh-badkeys)

## Bruteforce default credentials
```bash
hydra -C /usr/share/wordlists/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt $service://$hip -V -t 4 -I -s $hport
```


# Findings