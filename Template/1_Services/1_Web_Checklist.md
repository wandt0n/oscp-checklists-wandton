#service # login
<% tp.file.rename("1_Web_")%>
☑️
# Start Enum
```bash
export target="http://192.168./"; \
for site in {"robots.txt","crossdomain.xml","clientaccesspolicy.xml","sitemap.xml",".well-known/", "security.txt", "humans.txt"}; do chromium "$target$site" &> /dev/null & done && \
whatweb -v -a 3 $target | tee -a webeval.txt && \
dirb $target -S | tee -a webeval.txt && \
nikto -maxtime=60s -host=$target | tee -a webeval.txt && \
cmsmap -F -d $target | tee -a webeval.txt
```

For robots.txt analysis: https://www.google.com/webmasters/tools
If .well-known is not listable: [here on WikiPedia](https://en.wikipedia.org/wiki/List_of_/.well-known/_services_offered_by_webservers) or [here via IANA](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml)

Check sourcecode for:
- Comments `<!--`
- HTML version number and DTD Url in `<!DOCTYPE`
- `<meta>` tags. Should be independent of the path
- `<script>` tags including externally loaded scripts (especially if self-developed)

Check JS Sourcefiles
- Can be found in Sources tab of browser DevTools
- To find entrypoints, either
	- Check the Event Listeners tab after using the Inspector on, e.g., a button
	- Set a breakpoint on "Any XHR/fetch" via the toolbar in Sources tab
- Minified JS files often have the extension `.min.js`. This means that variable names are shortened and whitespaces are removed. To revert ("beautify") them:
	- Chromium debugger or https://github.com/beautify-web/js-beautify
	- Variable names can be restored with source maps (sourcefilename.js.map)
		- https://sokra.github.io/source-map-visualization/
		- Sometimes, minified code contains comment to path of source map. Then, Chromium Dev Tools will automatically fetch this and reconstruct a name.js, that is displayed along the name.min.js in the Sources Tab
- Deobfuscation: [javascript-deobfuscator](https://github.com/ben-sb/javascript-deobfuscator) or [JSNice](http://jsnice.org/)

Check downloadable content with [exiftool](https://exiftool.org/)

To find apps on **other subdomains**: 
`gobuster dns -do example.com -w /path/to/wordlist.txt` 

To find apps on **other virtual hosts**: 
- `gobuster vhost -u https://example.com -w /path/to/wordlist.txt`
	- Alternatives: `amass`, `subfinder`, `dnsrecon`, and `fierce`
- Certificate Transparency Logs like https://crt.sh/
- Public DNS records (A, AAAA, MX, TXT, NS) like with nslookup, dig, and host
	- Reverse DNS lookup (PTR record, does not always work)
- Public Databases like [Netcraft Search DNS](https://searchdns.netcraft.com/?host)
- Reverse IP Services like [MxToolbox Reverse IP](https://mxtoolbox.com/ReverseLookup.aspx)
- Zone transfer

To find apps on **other ports**, do not forget nmap scan
## Alternatives
Manual alternatives: 
- HTTP response headers of malformed HTTP requests (e.g. invalid Method) -> Does not work with some reverse proxies
- Google search with `site: target.com`

Alternative für nikto: `skipfish -o skipfish.txt $target`
Alternative for Dir Spidering: `gobuster dir -w /usr/share/wordlists/dirb/common.txt -u $target -x php,html,htm,txt,pdf,config -k | tee gobuster.txt`
Alternative for whatweb: [[8_Web_Identifiers]] or [Wappalyzer]([https://www.wappalyzer.com/](https://www.wappalyzer.com/))

Alternatives for CMS checking:
- `cmseek -u $hip`
- `pyver=$(pyenv global) && pyenv global 2.7 && python ~/Documents/activeInformationGathering/clusterd/clusterd.py -i $hip --fingerprint && pyenv global $pyver`

## App Components

### Architecture
#### PaaS
`*.azurewebsites.net` Domain
#### Serverless
`X-Amz-Invocation-Type` HTTP header (AWS) or usage of Kestrel (Azure)
#### Microservices
Usage of multipe languages
#### Cloud Storage
`BUCKET.s3.amazonaws.com` or `s3.REGION.amazonaws.com/BUCKET` (Amazon S3)
`ACCOUNT.blob.core.windows.net` (Azure Storage)
Also see [Testing Cloud Storage Guide](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/11-Test_Cloud_Storage)
#### Third Party Services and APIs
- [Active content](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content#mixed_active_content) (such as scripts, style sheets, fonts, and iframes)
- [Passive content](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content#mixed_passivedisplay_content) (such as images and videos)
- External APIs
- Social media buttons
- Advertising networks
- Payment gateways
#### Network Components
##### Reverse Proxy
Detect:
- Mismatch between HTTP Server header and the actual backend application -> Request Smuggling
- Multiple Server (or other) headers
- Multiple applications hosted on the same IP address or domain (especially if they use different languages)
##### Load Balancer
Detect by making multiple requests and look for differences in system times, IPs or hostnames in error messages or SSRF attack
F5 BIG-IP load balancers ->  `BIGipServer` cookie

##### Content Delivery Network (CDN)
Cloudflare, Akamai, Fastly, [...](https://en.wikipedia.org/wiki/Content_delivery_network#Notable_content_delivery_service_providers)
Identify backend systems:
- If webapp can send emails, e.g., "forgot password", use feature and check source of mail
- DNS grinding, zone transfers or certificate transparency lists for a domain may reveal it on a subdomain
- Scan IP ranges used by company
- Use SSRF or error messages to reveal IP address
#### Authentication
##### Basic Auth
`WWW-Authenticate: Basic` HTTP header
##### .htaccess files
##### Application specific
Stored in DB and requested via API
##### Central Authentication
May use NTLM: `WWW-Authenticate: NTLM` HTTP header
Or hints that the users domain is relevant for auth
##### SSO
OAuth, OpenID Connect, or SAML
### Administration
Options are:
- Additional Webapp, as with iPlanet web server
- Integrated Admin panels, as with Wordpress
- Plain text config files, as with Apache
- OS GUI tools, as with Microsoft IIS or ASP.Net
Files can be transfered through FTP servers, WebDAV, network file systems (NFS, CIFS)
### Database
Portscanning or triggering SQLi
- Windows, IIS and ASP.NET often use Microsoft SQL server
- Embedded systems often use SQLite
- PHP often uses MySQL or PostgreSQL
- APEX often uses Oracle
#### SQL Injections
AUSPROBIEREN: wfuzz

### CMS
#### Wordpress
```bash
wpscan --update --url "https://$hip:80" --detection-mode aggressive --plugins-detection aggressive --disable-tls-checks --enumerate ap,vt,cb,dbe | tee wpscan_extended.txt
```
**RCE Wordpress**
Upload an new plugin, just containing your shell.php. Then, go to media center, grap the path to that file and access it. 
**Loot Wordpress**
```bash
find / -name "setup-config.php" -o -name "wp-config.php"
```
#### Drupal
**Version**
`/CHANGELOG.txt` (Not accessible in newer versions)
**User**
Check if user exists by submitting it to /user/register or by requesting a password reset. ("Name is already taken" or "not recognized as user name")
Count up /user/1 until you get an 404 instead of access denied to get the number of users. (Use the script shown in the next section "Hidden pages")

**Hidden pages**
Drupal organizes all content in nodes. So try /node/1, /node/2, ... to find hidden pages.
`mkdir drupalFuzz && cd drupalFuzz && curl "http://$hip/node/[1-5]" -o "#1.html" --silent && grep -i -L "Not Found" -r .`

**Installed Modules**
Check `/config/sync/core.extension.yml` and `/core/core.services.yml`

**Script**
`droopescan scan drupal -u http://$hip`

**Authenticated RCE**
Drupal 8 and below has php module actived by default. Check if vulnerable (Access /modules/php, 403 -> vulnerable, 404 -> not installed).
Exploit: Login as Admin -> Modules -> Enable PHP Filter (Might need to enable it under People -> Permissions) -> Save Configuration. Add Content -> Write php shellcode on body -> Select PHP Code as Text Format -> Preview. 

**Loot Drupal**
`find / -name settings.php -exec grep "drupal_hash_salt\|'database'\|'username'\|'password'\|'host'\|'port'\|'driver'\|'prefix'" {} \; 2>/dev/null`
#### Joomla
```bash
./joomlascan.py $hip
```
**Loot Joomla**
```bash
find / -name "configuration.php" -o -name "diagnostics.php" -o -name "joomla.inc.php" -o -name "config.inc.php"
```
### Git
Dump: `/GitTools/Dumper/gitdumper.sh http://192.168.1.33:8080/.git/ git`
Extract: `/GitTools/Extractor/extractor.sh git retrieved`
## Results
### App Logic
Note down interesting facts about different paths for later evaluation. Including:
- Whether it sets new or modifies cookies (`Set-Cookie` header)
- Interesting sounding params
	- Can be retrieved with [OWASP Attack Surface Detector](https://github.com/secdec/attack-surface-detector-cli/releases) that is also installed in Burp
	- Can be found in query string, cookie header or request body
- Status Codes like 403, 3XX or 5XX

| Path | ReqID Burp | Method | Auth. o. Crypto.? | Interesting |
| ---- | ---------- | ------ | ----------------- | ----------- |
|      |            |        |                   |             |
|      |            |        |                   |             |
### Used Technologies
#### Architectures
[[#PaaS]] / [[#Serverless]] / [[#Microservices]] ?
#### External Dependencies
[[#Cloud Storage]]?
[[#Third Party Services and APIs]] ?
#### Authentication

#### Administration
How does access work?
Default credentials?
#### Software

| Type      | Product | Version | Vulnerable | Source of Version |
| --------- | ------- | ------- | ---------- | ----------------- |
| Database  |         |         |            |                   |
| Webserver |         |         |            |                   |
| CMS       |         |         |            |                   |
|           |         |         |            |                   |

### Grey-Box Testing
Do [[#Best-Practice Guides]] exist for the used technologies? Are they followed?

#### Logging
- Do the logs contain sensitive information?
- Are the logs stored in a dedicated server?
- Can log usage generate a Denial of Service condition?
- How are they rotated? Are logs kept for the sufficient time?
- How are logs reviewed? Can administrators use these reviews to detect targeted attacks?
- How are log backups preserved?
- Is the data being logged data validated (min/max length, chars etc) prior to being logged?

CONTINUE HERE: https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/03-Test_File_Extensions_Handling_for_Sensitive_Information

# References
## Best-Practice Guides
- Apache
    - Apache Security, by Ivan Ristic, O’reilly, March 2005.
    - [Apache Security Secrets: Revealed (Again), Mark Cox, November 2003](https://awe.com/mark/talks/apachecon2003us.html)
    - [Apache Security Secrets: Revealed, ApacheCon 2002, Las Vegas, Mark J Cox, October 2002](https://awe.com/mark/talks/apachecon2002us.html)
    - [Performance Tuning](https://httpd.apache.org/docs/current/misc/perf-tuning.html)
- Lotus Domino
    - Lotus Security Handbook, William Tworek et al., April 2004, available in the IBM Redbooks collection
    - Lotus Domino Security, an X-force white-paper, Internet Security Systems, December 2002
    - Hackproofing Lotus Domino Web Server, David Litchfield, October 2001
- Microsoft IIS
    - [Security Best Practices for IIS 8](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj635855\(v=ws.11\))
    - [CIS Microsoft IIS Benchmarks](https://www.cisecurity.org/benchmark/microsoft_iis/)
    - Securing Your Web Server (Patterns and Practices), Microsoft Corporation, January 2004
    - IIS Security and Programming Countermeasures, by Jason Coombs
    - From Blueprint to Fortress: A Guide to Securing IIS 5.0, by John Davis, Microsoft Corporation, June 2001
    - Secure IIS 5 Checklist, by Michael Howard, Microsoft Corporation, June 2000
- Red Hat’s (formerly Netscape’s) iPlanet
    - Guide to the Secure Configuration and Administration of iPlanet Web Server, Enterprise Edition 4.1, by James M Hayes, The Network Applications Team of the Systems and Network Attack Center (SNAC), NSA, January 2001
- WebSphere
    - IBM WebSphere V5.0 Security, WebSphere Handbook Series, by Peter Kovari et al., IBM, December 2002.
    - IBM WebSphere V4.0 Advanced Edition Security, by Peter Kovari et al., IBM, March 2002.
- General
    - [Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html), OWASP
    - [SP 800-92](https://csrc.nist.gov/publications/detail/sp/800-92/final) Guide to Computer Security Log Management, NIST
    - [PCI DSS v3.2.1](https://www.pcisecuritystandards.org/document_library) Requirement 10 and PA-DSS v3.2 Requirement 4, PCI Security Standards Council
- Generic:
    - [CERT Security Improvement Modules: Securing Public Web Servers](https://resources.sei.cmu.edu/asset_files/SecurityImprovementModule/2000_006_001_13637.pdf)