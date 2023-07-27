<% tp.file.rename("1_Web_")%>

## Subdomains
[getSubdomains.sh](file:////home/kali/Documents/passiveInformationGathering/)
	

## Get pictures to multiple websites
[ip2webOverview.sh](file:////home/kali/Documents/passiveInformationGathering/)
	

## Technologies

```
whatweb -v -a 3 $hip:80 > technologies.txt
```
Ausprobieren: https://github.com/projectdiscovery/httpx

## Nikto
```
nikto -host=http://$hip/ > nikto.txt
```
	-T for smaller test, -maxtime=30s for setting maxtime



## DIRB
```
dirb http://$hip:80 > dirb.txt
```

##### Alternative
```
gobuster dir -u http://$hip -w /usr/share/wordlists/dirb/common.txt -o gobuster.txt -x txt,pdf,config 
```


##### Files of possible relevance
`/robots.txt` , `/crossdomain.xml` `/clientaccesspolicy.xml` `/sitemap.xml` and `/.well-known/`

Ausprobieren:
Check potential vulnerable urls with https://github.com/1ndianl33t/Gf-Patterns
https://github.com/stevenvachon/broken-link-checker
https://github.com/projectdiscovery/nuclei-templates/tree/4e3f843e15c68f816f0ef6abce5d30b6cf6d4a30/exposures/tokens


## WPSCAN
Extended scan:
```
wpscan --url "https://$hip:80" --detection-mode aggressive --plugins-detection aggressive --disable-tls-checks --enumerate ap,vt,cb,dbe > wpscan_extended.txt
```

##### Fast Scan
```
wpscan --url "https://$hip:80" --detection-mode aggressive --plugins-detection aggressive --disable-tls-checks --enumerate p,vt,cb,dbe
```

# Findings

