<% tp.file.rename("1_Web_")%>

## Start scans
```bash
target=""; nikto -maxtime=60s -host=https://$target/ > nikto.txt & dirb http://$target -S > dirb.txt & whatweb -v -a 3 $target
```


##### Check interesting sites
```bash
for site in {"/robots.txt","/crossdomain.xml","/clientaccesspolicy.xml","/sitemap.xml","/.well-known/"}; do firefox "http://$hip$site"; done
```

##### Detect CMS Used
```
cmsmap -F -d $hip
```
Alternatives:
- `cmseek -u $hip`
- `pyenv global 2.7 && python ~/Documents/activeInformationGathering/clusterd/clusterd.py -i $hip --fingerprint`

##### Wordpress
```bash
wpscan --url "https://$hip:80" --detection-mode aggressive --plugins-detection aggressive --disable-tls-checks --enumerate ap,vt,cb,dbe | tee wpscan_extended.txt
```

