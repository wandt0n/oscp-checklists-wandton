<% tp.file.rename("1_SSH_")%>

- [ ] Banner grabbed: 
- [ ] Bad keys? (https://github.com/rapid7/ssh-badkeys)

## Bruteforce default credentials
```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt ssh://$hip -V -t 4 -I -s $hport
```

For a given user:
```bash
hydra -l $user -P /usr/share/wordlists/rockyou.txt ssh://$hip$ -V -t 4 -I -s 22
```

## Encrypted SSH Key
```bash
ssh2john $file > id_rsa.hash
```
With this, you can pass id_rsa.hash to john as regular. See [[8_Hashes_cheatsheet]]


# Findings