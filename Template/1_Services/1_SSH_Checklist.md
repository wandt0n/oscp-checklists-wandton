<% tp.file.rename("1_SSH_")%>

- [ ] Banner grabbed: 
- [ ] Bad keys? (https://github.com/rapid7/ssh-badkeys)

## Bruteforce default credentials
```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt $service://$hip -V -t 4 -I -s $hport
```

## Encrypted SSH Key
```bash
ssh2john $file > id_rsa.hash
```
With this, you can pass id_rsa.hash to john as regular. See [[7_crack_Hashes]]


# Findings