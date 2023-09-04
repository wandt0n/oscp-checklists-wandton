<% tp.file.rename("1_New_")%>

> Check https://web.archive.org/web/20200309204648/http://0daysecurity.com/penetration-testing/enumeration.html 

```bash

```

#### Bruteforce default credentials
```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt protocol://$hip -V -t 4 -I -s $hport
```

Or for a given user:
```bash
hydra -l $user -P /usr/share/wordlists/rockyou.txt protocol://$hip -V -t 4 -I -s $hport
```


# Findings

# Loot
```bash
find / -name "name1" -o -name "name2" -o -name "name3" -o -name "name4" -o -name "name5" -o -name "name6"
```