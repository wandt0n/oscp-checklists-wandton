--script-help
--script-args

HowTo: Get a list of ip-adresses with a certain port open.
```bash
nmap -p 445 192.168.222.1-253 -oG - | grep "open" | cut -d " " -f 2
```

