`for i in $(command | tr '\n' ' '); do command; done`

# Examples:
- Find a specific SMB user in many machines
```bash
for i in $(nmap -p 445 192.168.222.1-253 -oG - | grep "open" | cut -d " " -f 2 | tr '\n' ' '); do enum4linux -a "$i" | grep -E "Target|alfred"; done
```
