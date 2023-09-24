#service
<% tp.file.rename("1_Web_")%>
☑️
## Start Enum
```bash
export target="http://192.168./"; \
for site in {"robots.txt","crossdomain.xml","clientaccesspolicy.xml","sitemap.xml",".well-known/"}; do firefox "$target$site"; done && \
whatweb -v -a 3 $target | tee webeval.txt && \
dirb $target -S | tee webeval.txt && \
nikto -maxtime=60s -host=$target | tee webeval.txt && \
cmsmap -F -d $target | tee webeval.txt
```
AUSPROBIEREN: `skipfish -o skipfish.txt $target`
Alternative for Dir Spidering:
`gobuster dir -w /usr/share/wordlists/dirb/common.txt -u $target -x php,html,htm,txt,pdf,config -k | tee gobuster.txt`

Alternatives for CMS checking:
- `cmseek -u $hip`
- `pyver=$(pyenv global) && pyenv global 2.7 && python ~/Documents/activeInformationGathering/clusterd/clusterd.py -i $hip --fingerprint && pyenv global $pyver`


### SQL Injections
AUSPROBIEREN: wfuzz

# CMS
### Wordpress
```bash
wpscan --update --url "https://$hip:80" --detection-mode aggressive --plugins-detection aggressive --disable-tls-checks --enumerate ap,vt,cb,dbe | tee wpscan_extended.txt
```
##### RCE Wordpress
Upload an new plugin, just containing your shell.php. Then, go to media center, grap the path to that file and access it. 
##### Loot Wordpress
```bash
find / -name "setup-config.php" -o -name "wp-config.php"
```

### Drupal

##### Version
`/CHANGELOG.txt` (Not accessible in newer versions)
##### User
Check if user exists by submitting it to /user/register or by requesting a password reset. ("Name is already taken" or "not recognized as user name")
Count up /user/1 until you get an 404 instead of access denied to get the number of users. (Use the script shown in the next section "Hidden pages")

##### Hidden pages
Drupal organizes all content in nodes. So try /node/1, /node/2, ... to find hidden pages.
`mkdir drupalFuzz && cd drupalFuzz && curl "http://$hip/node/[1-5]" -o "#1.html" --silent && grep -i -L "Not Found" -r .`

##### Installed Modules
Check `/config/sync/core.extension.yml` and `/core/core.services.yml`

##### Script
`droopescan scan drupal -u http://$hip`

##### Authenticated RCE
Drupal 8 and below has php module actived by default. Check if vulnerable (Access /modules/php, 403 -> vulnerable, 404 -> not installed).
Exploit: Login as Admin -> Modules -> Enable PHP Filter (Might need to enable it under People -> Permissions) -> Save Configuration. Add Content -> Write php shellcode on body -> Select PHP Code as Text Format -> Preview. 

##### Loot Drupal
`find / -name settings.php -exec grep "drupal_hash_salt\|'database'\|'username'\|'password'\|'host'\|'port'\|'driver'\|'prefix'" {} \; 2>/dev/null`

### Joomla
```bash
./joomlascan.py $hip
```
##### Loot Joomla
```bash
find / -name "configuration.php" -o -name "diagnostics.php" -o -name "joomla.inc.php" -o -name "config.inc.php"
```


