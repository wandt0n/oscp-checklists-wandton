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
Alternative for whatweb: [[9_web_framework_identifiers]] or [Wappalyzer]([https://www.wappalyzer.com/](https://www.wappalyzer.com/))

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
Identify:
- Usage of multipe languages
- Error messages!
#### Cloud Storage
##### AWS
Identify:
	**Virtual Host Style Access**
	`https://BUCKET.s3.REGION.amazonaws.com/KEY`For example:
	`https://example-bucket.s3.us-west-2.amazonaws.com/puppy.png`
	In some regions, the legacy global endpoint (without a region) can be used: `BUCKET.s3.amazonaws.com` 
	**Path-Style Access**
	`https://s3.REGION.amazonaws.com/BUCKET/KEY`
	In some regions, the legacy global endpoint (without a region) can be used: `s3.amazonaws.com/BUCKET`

**Test with AWS CLI**
List objects: `aws s3 ls s3://<bucket-name>`
Upload: `aws s3 cp arbitrary-file s3://bucket-name/path-to-save`
Remove: `aws s3 rm s3://bucket-name/object-to-remove`

RFI attack: `DocumentPath=doucment.pdf&DocumentUrl=/uploads/user/blafasel/` -> `DocumentPath=aws.dat&DocumentUrl=http://0xA9FEA9FE/latest/meta-data/identity-credentials/ec2/security-credentials/ec2-instance`
##### Azure
Identify:
	`ACCOUNT.blob.core.windows.net`
#### Third Party Services and APIs
- [Active content](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content#mixed_active_content) (such as scripts, style sheets, fonts, and iframes)
- [Passive content](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content#mixed_passivedisplay_content) (such as images and videos)
- External APIs
- Social media buttons
- Advertising networks
- Payment gateways

#### Browser-based Storage
Check if:
- Tokens or session IDs persist after logout
- Sensitive data is stored in persistent storage without necessity
**LocalStorage**
key-value strings, persistent
**Session Storage**
key-value strings, ephemeral -> clears when browser tab is closed
**IndexedDB**
object-oriented, persistent. Allows to store complex objects like CryptoKeys. If there are, check if they are set as `extractable: true`
**WebSQL**
Deprecated since 2010. See [Testing for Client-side Web SQL Database](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.8-Testing_for_Client-side)
**Cookies**
key-value strings, persistence depends on attribute
**Global Window Object**
Developers can add custom attributes to the global window object, e.g., with `window.mystate = {isAdmin: yes}`
Customizable, ephemeral until page is refreshed
### Web Servers
Try to break the HTTP RFC, see whether this reveals info through error messages.
#### Caching
Responses that contain sensitive information should use HTTPS and  include a `Cache-Control: no-store` header
Can be tested by logging out and pressing the browser's back button. It should not reveal the sensitive information
Further info: about:cache (Firefox)
#### TLS
[KnowledgeBase - TLS](obsidian://open?vault=KnowledgeBase&file=03%20-%20Content%2F08%20-%20Courses%2F08%20-%20TLS)
- Subject Alternate Name (SAN) should match the system hostname
- Automate testing with nmap, sslscan, or sslyze
#### Server-Side Includes (SSI)
Disabled by default (`exec` directive). Allows to change returned HTML dynamically with simple and small pieces of code.
How to identify if enabled:
- Existance of `.shtml` files or
- SSI directives in server-side code #whitebox 
- Inject SSI into potential injection points
	- `<!--#echo var="VAR" -->`
	- `<!--#include virtual="FILENAME" -->` (shows content of file or output of CGI script, depending on the file type)
	- `<!--#exec cmd="OS_COMMAND" -->`

### Network Components
#### Reverse Proxy
Detect:
- Mismatch between HTTP Server header and the actual backend application -> Request Smuggling
- Multiple Server (or other) headers
- Multiple applications hosted on the same IP address or domain (especially if they use different languages)
#### Load Balancer
Detect by making multiple requests and look for differences in system times, IPs or hostnames in error messages or SSRF attack
F5 BIG-IP load balancers ->  `BIGipServer` cookie

#### Content Delivery Network (CDN)
Cloudflare, Akamai, Fastly, [...](https://en.wikipedia.org/wiki/Content_delivery_network#Notable_content_delivery_service_providers)
Identify backend systems:
- If webapp can send emails, e.g., "forgot password", use feature and check source of mail
- DNS grinding, zone transfers or certificate transparency lists for a domain may reveal it on a subdomain
- Scan IP ranges used by company
- Use SSRF or error messages to reveal IP address
#### DNS
`A`, `CNAME`, `MX`, `TXT`, and especially `NS` pointing to a target that can be registered by the attacker can lead to subdomain takeover
When querying with `dig`, look for `NXDOMAIN`, `SERVFAIL`, `REFUSED`, and `no servers could be reached`. These indicate invalid configurations

#### WebSockets
`ws://` -> Unencrypted (HTTP). `wss:\\` -> Encrypted (HTTPS)
The `Sec-WebSocket-Key` header is not used for authentication or session handling, just to prevent errors from caching proxies
To identify: `ws:\\` or `wss:\\` in client-side code, network tab in browser dev tools, Burp Suite
**Cross-Site WebSocket Hijacking (CSWSH)**
If `Origin` header is honored and not CSRF tokens are used for the WS handshake, a CSRF-style attack is possible. Lure a victim onto attacker.com, which requests a WS handshake with the real application. The Browser will send the victims session cookies, authenticating it. But because the `Origin` header is attacker.com, our site cannot only send requests but, in contrast to CSRF, also receives the WS responses. Also, WS is not restrained by SOP.
To test: Attempt to connect to the remove WS server using a [WS client](https://github.com/ethicalhack3r/scripts/blob/master/WebSockets.html). If a connection is established, they probably do not check the `Origin` header

#### Web Messaging
Also called Cross Document Messaging, introduced in HTML5. Allows for secure messaging between origins across iframes, tabs, and windows.
To identify: Look for `postMessage(...` or `addEventListener("message", ...` in client-side code
- Check that the handler of the event listener is filtering for `event.origin` and that neither it nor the second argument to `postMessage` uses wildcards
- Check that the handler of the event listener is sanitizing the received data properly, to prevent XSS and other vulnerabilities caused through this


### Administration
Options are:
- Additional Webapp, as with iPlanet web server
- Integrated Admin panels, as with Wordpress
- Plain text config files, as with Apache
- OS GUI tools, as with Microsoft IIS or ASP.Net
Files can be transfered through FTP servers, WebDAV, network file systems (NFS, CIFS)
#blackbox To identify the used option:
- [Google Dorks](https://www.exploit-db.com/google-hacking-database)
- Comments and Links in User-accessible site code
- Application Documentation
- Nmap
#whitebox Is admin functionality accessible when opening the correct urls or contacting the correct endpoints directly?
Also see [[9_admin_pages#List of Admin Pages|List of Admin Pages]]
### Database
Portscanning or triggering SQLi
- Windows, IIS and ASP.NET often use Microsoft SQL server
- Embedded systems often use SQLite
- PHP often uses MySQL or PostgreSQL
- APEX often uses Oracle
#### SQL Injections
Yet to evaluate: wfuzz
> [!important] Do not test in production environments! Even simple statements like `OR 1==1` could end up in more queries that anticipated and change data

[KnowledgeBase - SQLi](obsidian://open?vault=KnowledgeBase&file=03%20-%20Content%2F08%20-%20Courses%2F08%20-%20SQL%20Injection)

And specific OWASP guides, depending on the used DMBS:
- [Testing for Oracle](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.1-Testing_for_Oracle)
- [Testing for MySQL](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.2-Testing_for_MySQL)    
- [Testing for SQL Server](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.3-Testing_for_SQL_Server)
- [Testing PostgreSQL](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.4-Testing_PostgreSQL)
- [Testing for MS Access](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.5-Testing_for_MS_Access)
- [Testing for NoSQL Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.6-Testing_for_NoSQL_Injection)
- [Testing for ORM Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.7-Testing_for_ORM_Injection)
- [[#Browser-based Storage|Testing for Client-side Web SQL Database]]
### CMS
#### Wordpress
```bash
wpscan --update --url "https://$hip:80" --detection-mode aggressive --plugins-detection aggressive --disable-tls-checks --enumerate ap,vt,cb,dbe | tee wpscan_extended.txt
```
**RCE Wordpress**
Upload an new plugin, just containing your shell.php. Then, go to media center, grap the path to that file and access it. 
**Admin Pages**
```
wp-admin/
wp-admin/about.php
wp-admin/admin-ajax.php
wp-admin/admin-db.php
wp-admin/admin-footer.php
wp-admin/admin-functions.php
wp-admin/admin-header.php
```
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
**Admin Pages**
```
/administrator/index.php
/administrator/index.php?option=com_login
/administrator/index.php?option=com_content
/administrator/index.php?option=com_users
/administrator/index.php?option=com_menus
/administrator/index.php?option=com_installer
/administrator/index.php?option=com_config
```
**Loot Joomla**
```bash
find / -name "configuration.php" -o -name "diagnostics.php" -o -name "joomla.inc.php" -o -name "config.inc.php"
```
### Git
Dump: `/GitTools/Dumper/gitdumper.sh http://192.168.1.33:8080/.git/ git`
Extract: `/GitTools/Extractor/extractor.sh git retrieved`

### APIs

#### GraphQL
**Introspection**
Allows to request what queries are supported and with which data types
```http
POST /graphql? HTTP/1.1
...

{"query":"\n    query IntrospectionQuery {\r\n      __schema {\r\n        queryType { name }\r\n        mutationType { name }\r\n        subscriptionType { name }\r\n        types {\r\n          ...FullType\r\n        }\r\n        directives {\r\n          name\r\n          description\r\n          locations\r\n          args {\r\n            ...InputValue\r\n          }\r\n        }\r\n      }\r\n    }\r\n\r\n    fragment FullType on __Type {\r\n      kind\r\n      name\r\n      description\r\n      fields(includeDeprecated: true) {\r\n        name\r\n        description\r\n        args {\r\n          ...InputValue\r\n        }\r\n        type {\r\n          ...TypeRef\r\n        }\r\n        isDeprecated\r\n        deprecationReason\r\n      }\r\n      inputFields {\r\n        ...InputValue\r\n      }\r\n      interfaces {\r\n        ...TypeRef\r\n      }\r\n      enumValues(includeDeprecated: true) {\r\n        name\r\n        description\r\n        isDeprecated\r\n        deprecationReason\r\n      }\r\n      possibleTypes {\r\n        ...TypeRef\r\n      }\r\n    }\r\n\r\n    fragment InputValue on __InputValue {\r\n      name\r\n      description\r\n      type { ...TypeRef }\r\n      defaultValue\r\n    }\r\n\r\n    fragment TypeRef on __Type {\r\n      kind\r\n      name\r\n      ofType {\r\n        kind\r\n        name\r\n        ofType {\r\n          kind\r\n          name\r\n          ofType {\r\n            kind\r\n            name\r\n            ofType {\r\n              kind\r\n              name\r\n              ofType {\r\n                kind\r\n                name\r\n                ofType {\r\n                  kind\r\n                  name\r\n                  ofType {\r\n                    kind\r\n                    name\r\n                  }\r\n                }\r\n              }\r\n            }\r\n          }\r\n        }\r\n      }\r\n    }\r\n  ","variables":null}
```
[Source](https://the-bilal-rizwan.medium.com/graphql-common-vulnerabilities-how-to-exploit-them-464f9fdce696) 
The result can be visualized with [GraphQL Voyager](https://apis.guru/graphql-voyager), although this does not display everything, e.g., it does not list available mutations
If introspection is not available: Some implementations (e.g. Apollo Server and graphql-js) will return suggestions when misspelled field names are sent

## Web Logic
### Paths
https://filext.com/
#### Download
What kind of file extensions can I download?
How are their permissions set?
- File permissions may be part of EXIF data of files that can be accessed
[Path Traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal) 
#whitebox Are any backup files or archives accessible?
Can I circumvent checks abusing the Windows 8.3 legacy filename handling?
	DOS allowed max. 8 chars + optionally max. 3 chars extension. Chars must be ASCII (<0x80) and not contain more than one `.`. 
	Modern Windows generates shorthand file names for compatibility by:
	- Using up to the first six chars of the basename, appended by `~1`
	- If present, using up to the first three chars of the file extension
	- Removing incompatible characters and replacing spaces with underscores
	- Making all characters upper-case.
	For example:
	- `file.phtml` -> `FILE~1.PHT`
	- `secretfile.txt` -> `SECRET~1.txt`
	- `two.txt` -> `TWO~1.txt`
	- `filenamewithoutextension` -> `FILENA~1` or `FILENA~1.` (equivalent)
#### Upload
Can I change the MIME type, e.g., from text/plain to text/html?
```http
Content-Disposition: form-data; name="uploadfile1"; filename="\user\share\www\html\test.gif"
Content-Type: text/html

<script>alert(document.cookie)</script>
```
What kind of file extensions can I upload?
Does the app logic change file extensions of uploaded files? -> Polyglots?
Are magic bytes checked to verify that the file extension is correct for a given file?
[Path Traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)
[[#Local/Remote File Inclusion]]
#### Caching Confusion
Responses to paths with user-specific data usually contain do-not-cache headers.
Do the following URIs resolve to `https://example.com/home`?
- `https:// example.com/home`
- `https://example.com/home/nonexistant.css`
Then, we might lure a victim to one of these URIs, which will likely cache his personal data and make it available to us on this same URI.
#whitebox Is caching dependant on some Regex or file extensions, instead of the set do-not-cache headers?

### Parameters
#### HTTP Parameter Pollution (HPP)
Insert duplicate or unexpected HTTP or query parameters
##### Server-side HPP
If the same parameter is used multiple times, different webservers return different data
-> Same behavior between paths

Can be used to bypass filters, WAFs, or business logic.
To identify:
Send one request with a valid, one with an invalid, and one with both parameter values. If the response differs between all three, HPP is possible.
```http
POST /search HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "search_string": "kittens",
  "search_string": "puppies"
}
```
or `https://example.com/?mode=guest&search_string=kittens&num_results=100&search_string=puppies`

Also, try: `&param[0]=a&param[1]=b`

| Web Application Server Backend               | Parsing Result                            |
| -------------------------------------------- | ----------------------------------------- |
| ASP.NET / IIS                                | All occurrences concatenated with a comma |
| ASP / IIS                                    | All occurrences concatenated with a comma |
| .NET Core 3.1 / Kestrel                      | All occurrences concatenated with a comma |
| .NET 5 / Kestrel                             | All occurrences concatenated with a comma |
| PHP / Apache                                 | Last occurrence only                      |
| PHP / Zeus                                   | Last occurrence only                      |
| JSP, Servlet / Apache Tomcat                 | First occurrence only                     |
| JSP, Servlet / Oracle Application Server 10g | First occurrence only                     |
| JSP, Servlet / Jetty                         | First occurrence only                     |
| IBM Lotus Domino                             | Last occurrence only                      |
| IBM HTTP Server                              | First occurrence only                     |
| Node.js / express                            | First occurrence only                     |
| mod_perl, libapreq2 / Apache                 | First occurrence only                     |
| Perl CGI / Apache                            | First occurrence only                     |
| mod_wsgi (Python) / Apache                   | First occurrence only                     |
| Python / Zope                                | All occurrences in List data type         |
(Source: Appsec EU 2009 Carettoni & Paola)
##### Client Side HPP
If HTTP parameters are used in client-side code, e.g., inserted into an href, check whether `%26` is reflected url-decoded
Example: `par=val%26action=delete` could be reflected to `<a href="http://example.com?par=val&action=delete"> View </a>`
### HTTP Methods
To list supported methods:
```http
OPTIONS / HTTP/1.1
Host: example.org
```
Or just try them out. `405 Method Not Allowed` should be returned for non-supported methods.
```
curl -X FOO https://example.org
```
Possible WAF bypass: Unknown methods may be treated like GET.
If a method, e.g., DELETE, is not allowed, test whether the following HTTP Headers can change the 405 to a 200:
- `X-HTTP-Method: DELETE`
- `X-HTTP-Method-Override: DELETE`
- `X-Method-Override: DELETE`
#### PUT
```http
PUT /test.html HTTP/1.1
Host: example.org
Content-Length: 25

testfilePUTupload
```
#### CONNECT
```http
CONNECT 192.168.0.1:443 HTTP/1.1
Host: example.org
```

### HTTP Headers
[Mozilla HTTP Headers Doku](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)
#### Strict Transport Security (STS)
Tells browser to always use TLS. Should be present.
Identify:
`Strict-Transport-Security:` response header
- `max-age=1223;`: no. of seconds that this policy must be enforced from now
- `includeSubDomains;`: All related sub-domains must be accessed with HTTPS too
- `preload;` (Unofficial): Domains are on the [preload lists](https://hstspreload.org/). Browsers should never connect without HTTPS
Only effective if delivered over HTTPS; Ineffective if delivered over HTTP
#### Content Security Policy (CSP)
Restricts sources from which JavaScript, CSS, images, files etc. can be loaded. Should be present and configured well.
Identfiy
`Content-Security-Policy:` response header or equivalent `<meta>` element.
Misconfigurations:
	General:
	- `Content-Security-Policy-Report-Only` -> Does not enforce CSP
	- Missing `object-src 'none'`, `base-uri 'none'`, or `frame-ancestors`
	- `unsafe-inline`, `-eval`
	- Absence or misuse of `frame-ancestors` -> Clickjacking (if not X-Frame-Options DENY or SAMEORIGIN)
	- Missing `object-src`, `base-uri`, or restrictive `default-src`
	- Duplicate directives or conflicting policy definitions -> Unintended enforcement behavior possible
	- Do sources host JSONP endpoints (e.g. called with `?jsonp=` or `?callback=`) or user-controlled content
		Especially if sources use (partial-)wildcards
	If CSP is delivered via `<meta>` element, check that
	- `frame-ancestors`, `report-uri`, `report-to`, or `sandbox` are not used (not supported in meta tag definition)
	If `unsafe-hashes` or nonces are used, check that:
	- Hashes are not predictable and improperly scoped
	- Nonces are cryptographically random, never reused, and regenerated per response
	- If `strict-dynamic` is used, check that:
		- no trust chain allows attacker-controlled script loading (trust propagates from nonce- or hash-based scripts)
	If `report-uri` or `report-to` is configured, check that:
	- reporting endpoints are reachable and functional
	- reports do not expose sensitive information
	- reports do not introduce injection or DoS vectors 
	If target is high-risk application: 
	- Absence or lax usage of `require-trusted-types-for` and `trusted-types` -> DOM-based injection sinks
	
Easy eval: https://csp-evaluator.withgoogle.com/
#### Hop-By-Hop Headers
Not meant to be forwarded. Intended to be used in only the next hop (client<->proxy, proxy<->proxy, ...).  [RFC 2616](https://tools.ietf.org/html/rfc2616#section-13.5.1). 
Specific headers: `Keep-Alive`, `Transfer-Encoding`, `TE`, `Connection`, `Trailer`, `Upgrade`, `Proxy-Authorization`, and `Proxy-Authenticate`

Additional headers can be designated as hop-by-hop via the `Connection` header!
**IP Spoofing**
```http
GET / HTTP/1.1
Host: example.org
X-Forwarded-For: <fake-ip-address>
Connection: close, X-Forwarded-For
```
-> Misconfigured proxy might strip the X-Forwarded-For header and not include its own -> Web app thinks request comes from proxy itself or fake ip-address
**Cache Poisioning**
Setting the `Cookie` header as hop-by-hop may cause the response to be cached -> users get this cached version instead of the one tailored for their cookie
**Removing Security Headers**
```http
GET /api/user/profile HTTP/1.1
Host: example.com
X-Authenticated-User: victim_user
Connection: close, X-Authenticated-User
```
Could bypass IP-based ACLs, Identity/Authentication checks performed at the edge or disable security features enforced by intermediary headers
#### Deprecated Headers
- HTTP Public Key Pinning (HPKP)
- X-Frame-Options: ALLOW-FROM

#### Host Header Injection
If we set `Host: www.attacker.com` or `X-Forwarded-Host: www.attacker.com`:
- Does the webserver redirect us ?
- Is `attacker.com` added to client-side anchors?
- Is it used for the generated password reset links?
Can we access internal sites by setting the Host header to intranet.example.com? E.g. in Virtual Hosts in a split-horizon DNS setup#### HTTP Response Splitting Attacks
If user input is passed to a response header and does not sanitize `\r\n`, we can split the HTTP response in two, allowing to insert our own body into the first response.
Example: The Location header of a 302 reflects the interface parameter, e.g., `Location: https://victim.com/main.jsp?interface=advanced`
-> `advanced%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2035%0d%0a%0d%0a<html>Sorry,%20System%20Down</html>`
#### HTTP Response Smuggling (HRS)
See [KnowledgeBase - HTTP](obsidian://open?vault=KnowledgeBase&file=03%20-%20Content%2F08%20-%20Courses%2F08%20-%20Hyper%20Text%20Transfer%20Protocol)
Mitigated with HTTP/2
**CL.TE**
```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 35
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
Foo: x
```
If vulnerable: Subsequent legitimate requests are corrupted or return unexpected responses (e.g., 404)

**TE.CL**
```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 4
Transfer-Encoding: chunked

5c
GET /admin HTTP/1.1
Content-Length: 0

0
```
If vulnerable: Remaining payload is interpreted as a new request
**TE.TE**
```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 44
Transfer-Encoding:\tchunked
Transfer-Encoding: identity

0

GET /404 HTTP/1.1
Foo: bar
```
### Authentication
#### Basic Auth
`WWW-Authenticate: Basic` HTTP header
Ensure that HTTPS is used. Password is cleartext.
#### .htaccess files
#### Application specific
Stored in DB and requested via API
- Can we distinguish whether the user is non-existent or the password is wrong?
- Do default or test credentials work?
- Is a password policy enforced and sufficient?
- Can they be easily bruteforced? #whitebox 
- Does a password change require re-authentication? If not, CSRFing it might be possible
- If password is submitted via form: Does the form action specifies https?
- Can we bypass Authentication?
	- Manipulate password check through SLQi
	- If compared in php with `==`, not `===`: Is the password `1` or `true` accepted?
**Lock-out and throttling mechanisms**
Should be used and sufficient
AWS's Cognito is difficult to detect. To test for it:
	To test for this using a fuzzing tool, such as Burp Suite’s Intruder, navigate into the “Resource Pool”. Then set the maximum concurrent requests to 1 and the delay between requests to 2 seconds. Attempt the invalid authentication 200 times, then attempt to use the valid credentials 3 times directly after the fuzzing tool finishes. Wait 2 minutes and attempt to sign-in. If sign-in is then successful, Cognito may be in use. Further testing can then be performed to validate the use of Cognito by attempting to push the lockout time higher, but it may be easier to validate this information with the client.
Can an unlock mechanism be triggered?
Can we bypass the lock-out mechanism by changing `X-forwarded-For` on every request?
**Forgot password method**
- Is it different in the API or mobile app? 
- Does it bypass MFA?
- Are sufficient rate-limitings or lock-out mechanisms in place?
- Can the used identifiers (e.g. email) be changed without requiring re-authentication? If so, the re-auth on password change can be bypassed
- Is the user informed if their password is changed?
- Vulnerabilities in sending mail/sms/..
	- Exposure of information about the backend
	- Open Relay attacks, e.g., to drive up costs for the business
*Reset Links*
- Are they generated using the Host header? -> Host header injection to steal token
- Do they expire after time and after use?
- Does the target site embedd third-party sites -> If `Referrer-Policy` is not set, `Referrer` header might expose token to these third-parties
- Can the token be guessed? Can it be used to reset a different user's password?
MFA mechanisms should be tested in a similar fashion
**CAPTCHAs**
Should not replace lock-out mechanisms. Common weaknesses are:
- Easily defeated challenge, such as arithmetic or limited question set
- Solution to the captcha is contained in alt-text of image, filenames, or a hidden field
- captcha server-side logic does
	- not check for solve or defaults to a successful solve
	- check for HTTP response code instead of response success
	- only check for correct answer, not if it matches to the actual question
- captcha input field or parameter is improperly validated or escaped
- captcha is not asked for when clearing the cookies (e.g. if only shown on multiple wrong attempts) or when using the corresponding API

#### Central Authentication
May use NTLM: `WWW-Authenticate: NTLM` HTTP header
Or hints that the users domain is relevant for auth
##### LDAP Injection
Queries used to authenticate or search other users. Defined in [RFC2254](https://www.ietf.org/rfc/rfc2254.txt) (LDAPv3) and [RFC1960](https://www.ietf.org/rfc/rfc1960.txt) (LDAPv2).
Uses Polish notation: `find("name=a & pw=b")` is written as `find("(&(name=a)(pw=b))")`
Metachars: `&` (and), `|` (or), `!` (not), `=` (equals), `~=` (approx), `>=` (greater than), `<=` (less than), `*` (any char), `()` (grouping)
Variables: `cn` (name), `userPassword` (password)
Example: 
`searchlogin= "(&(uid="+user+")(userPassword={MD5}"+base64(pack("H*",md5(pass)))+"))";`
-> submit user=`*)(uid=*))(|(uid=*` and pass=asdf
#### SSO
OAuth, OpenID Connect, or SAML
[KnowledgeBase: Single Sign-On](obsidian://open?vault=KnowledgeBase&file=03%20-%20Content%2F08%20-%20Courses%2F03%20-%20Single%20Sign-On)
If client_secret is leaked, try OAuth Client Credential Flow
```bash
curl -X POST -d "grant_type=client_credentials&client_id=00000000-1111-2222-3333-444444444444&client_secret=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&scope=https://graph.microsoft.com/.default" https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token
```
Implicit code flows are generally more dangerous
Bad if leaked: `access_token`, `refresh_token`, `authorization_code`, `code_challenge` with `code_verifier`
### Authorization
Autorize BurpSuite Addon
Horizonal Access (Modify or View Content of different user of the same role/group)
Vertical Access (Forced Browsing)
Special Request Headers:
```http
GET / HTTP/1.1
Host: www.example.com
X-Original-URL: /donotexist1
```
And the same with `X-Rewrite-URL`. Does this lead to an 404, although `/` without these headers does not? -> Bypass Authorization through rewrite headers
Imitating local access:
`X-Forwarded-For`, `X-Forward-For`, `X-Remote-IP`, `X-Originating-IP`, `X-Remote-Addr`, `X-Client-IP` with `127.0.0.1:43982` or `169.254.0.5`
Does authorization rely on regex? See [[8_grep_cheatsheet#Web#Regex|grep regex]] #whitebox 

### Session Management
- If "Remember Me" option is available: Does that reflect user's password?
- Can we predict the Session ID? If some parts are static: Can we guess it?
- Are all `Set-Cookie` directives tagged as `Secure`?
- Is the cookies transported over TLS? Can we change that?
- Is the expire time reasonable?
- Do `Cache-Control` settings protect Cookies?

### Generic Inputs
In other sections: [[#SQL Injections]] [[#LDAP Injection]] 
#### XSS
- Identify variables that are reflected in responses, later used on the page, or directly injected into the DOM tree.
- Assess the input they accept and the encoding that gets applied on return (if any).

**Bypasses**
- Encoding
- `<scr<script>ipt>`
- `?param=<script&param=>[...]</&param=script>` ([[#HTTP Parameter Pollution (HPP)]])
[OWASP XSS Filter Evasion Cheat Sheet](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)

#### XSS Inclusion (XSSI)
Similar to CSRF, but with the goal of reading sensitive data in the context of a victim instead of performing actions in it.
Due to SOP, the response to cross-origin requests can't be read. Therefore, if we lure a victim to attacker.com and make a request to victim.com in their context, we cannot read the response from victim.com. However, if we embed a sensitive resource from victom.com into attacker.com, we might be able to leak its contents from the client-rendered version of attacker.com.

To Identify: Vulnerable files include sensitive data only if requested by an authenticated user. To be exploitable, the files must be JS files or an attacker must be able to inject JS code into the file such that the file can be interpreted as JS and sensitive data is assigned to a global variable or returned by a function.

Exploit: Embed the file with `<script src=...` on attacker.com and then leak its content with client-side source code after luring a victim on attacker.com. Leaking can be as easy as accessing the global variable, or it could mean overwriting browser functions like the array constructor or the forEach function.
#### Mass Assignment
With autobinding, user-provided data is automatically used for object creation in the backend, e.g., an object of class user. If that user has an isAdmin property and autobinding is used, we can this field during user creation to get admin.
An indicator are input names with brackets, e.g., `<input name="user[name]"...`
**Visibility in Code** #whitebox 
Java Spring MVC: `@RequestMapping`. 
	Verify that controls are in place , e.g., list of bindable fields via `setAllowedFields` or list of non-bindable fields via `setDisallowedFields`. Also, check `@ModelAttribute` that allows to specify a different name/key.
PHP Laravel Eloquent ORM: `create`
	The latest versions of Eloquent ORM provide default protection against mass assignment vulnerabilities requiring to explicitly specify allowed attributes that can be assigned automatically, through the `$fillable` array, or attributes that have to be protected (non-bindable), trough the `$guarded` array -> Analyze classes that extend the `Model` class to identify which attributes are allowed or denied and therefore point out potential vulnerabilities.
ASP .NET: Automatically.
	Works with complex types and it will automatically convert the input data to the properties if the properties’ names match with the input
	Verify that controls are in place, i.e., fields declared as `ReadOnly`, list of bindable fields via `Bind` attribute, via `includeProperties`, or via `TryUpdateModel` and list of non-bindable fields via `Bind` attribute, or via `excludeProperties`
#### CSRF
If CSRF not possible, check for [[#Client Side HPP]]

#### Server-Side Request Forgery (SSRF)
Trick server to perform arbitrary request. Can cause RFI, LFI, Path Traversal, RCE, ...
##### Local/Remote File Inclusion
If the file extension is appended to the string in the backend, we can...
- append our extension with `%00`
- Make our file name + extension 4096 bytes long. Many PHP installations truncate filenames longer than that without error
Can be combined with certain PHP wrappers, if used by the app, to increase impact. Examples are:
- *PHP Filter* allows to return PHP files to the browser through encoding them instead of executing them on the server
  `php://filter/convert.base64-encode/resource=FILE`
- *PHP ZIP* (included since PHP 7.2.0). Allows to use a file `internal_filename` in a zip archive located at `filename_path`. Since the archive must not have a zip extension, it may allow to bypass filters by renaming your zip with the malicious code in it to a .jpg or similar. The `#` might need to be encoded to `%23`.
  `zip:///filename_path#internal_filename`
- *PHP Data* (included since PHP 5.2.0, enabled with `allow_url_include`). 
  `data://text/plain;base64,PD9waHAgcGhwaW5mbygpOyA/Pg==` for phpinfo
- *PHP Expect*: Direct RCE. `expect://whoami`

PHP `include()` statements happily accept URLs to files hosted somewhere too
[PayloadsAllTheThings - LFI/RFI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)


#### Command Injection
Perl (pipe-character): `cgi-bin/userData.pl?doc=user1.txt` -> `cgi-bin/userData.pl?doc=|/bin/ls`
PHP (semi-colon): `something.php?dir=%3Bcat%20/etc/passwd`

**Filter Bypasses:**
Character replacement:
	Linux: `/` -> `${PATH:0:1}` |  `;` -> `${LS_COLORS:10:1}` | `[space]` -> `${IFS}` or through brace expansion (`{ls,-la}}`)
	Windows CMD: `\` -> `%HOMEPATH:~6,1%`
	Windows PowerShell: `\` ->  `$env:HOMEPATH[0]`
Command masking:
	Linux: Characters like `\`, `$@`, or `''` do not affect the exection, e.g., is `w$@ho'a'\mi` equivalent to whoami
	Windows CMD: `^` and `""`
Capitalization:
	Linux: Requires to be transferred back to normal: `;$(tr "[A-Z]" "[a-z]"<<<"WhoaMi")`
	Windows: Works without issue `WhoaMi`
Base64
	Linux: `bash<<<$(base64 -d<<< d2hvYW1p)`
#### XML Injection
If the sink is XML, e.g., an XML DB, try to invalidate the XML:
- If sink is within `"..."`, add `"`. Or `'` for `'...'`
- If sink is within `<tag>...</tag>`, add `<` or open a comment with `<!--`
- If sink is within `<!\[CDATA\[ ... ]]>`, add `]]>`
Depending on the success of these injections, you could try different attacks:
- `<![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/script<![CDATA[>]]>`
- `name</name><userid>0</userid><name>name`
	- if the same tag appears twice, it is common that the latter is used
	- That is not true if it is validated against an DTD that defines cardinality, e.g., with `<!ELEMENT user (name,password,userid,mail+) >`
	- But if a previous field can be controlled, the first of duplicate fields can be removed with an opening comment

Vulnerable to XXE are the following Java APIs: #whitebox 
```
javax.xml.parsers.DocumentBuilder
javax.xml.parsers.DocumentBuildFactory
org.xml.sax.EntityResolver
org.dom4j.*
javax.xml.parsers.SAXParser
javax.xml.parsers.SAXParserFactory
TransformerFactory
SAXReader
DocumentHelper
SAXBuilder
SAXParserFactory
XMLReaderFactory
XMLInputFactory
SchemaFactory
DocumentBuilderFactoryImpl
SAXTransformerFactory
DocumentBuilderFactoryImpl
XMLReader
Xerces: DOMParser, DOMParserImpl, SAXParser, XMLParser
```
For C Code, check
- `libxml2: xmlCtxtReadMemory,xmlCtxtUseOptions,xmlParseInNodeContext,xmlReadDoc,xmlReadFd,xmlReadFile ,xmlReadIO,xmlReadMemory, xmlCtxtReadDoc ,xmlCtxtReadFd,xmlCtxtReadFile,xmlCtxtReadIO`
- `libxerces-c: XercesDOMParser, SAXParser, SAX2XMLReader`

Also check [KnowledgeBase - XML](obsidian://open?vault=KnowledgeBase&file=03%20-%20Content%2F08%20-%20Courses%2F08%20-%20XML) for XEE and XXE attacks.
##### XPath Injection
Like [[#SQL Injections]], but for XML.
Example login query in XPath: `string(//user[username/text()='gandalf' and password/text()='!c3']/account/text())`
Could be injected with `' or '1' = '1`
Can also work blindly: [Blind XPath Injection](https://owasp.org/www-community/attacks/Blind_XPath_Injection)

#### Template Injection
To identify:
- Universal error-based polyglot: `<%'${{/#{@}}%>{{`
- If errors are not reflected, use universal non-error-based polyglots: `p ">[[${{1}}]]`, `<%=1%>@*#{1}`, `{##}/*{{.}}*/`
See the [hackmanit cheatsheet](https://cheatsheet.hackmanit.de/template-injection-table/index.html) and [KnowledgeBase - RCE](obsidian://open?vault=KnowledgeBase&file=03%20-%20Content%2F08%20-%20Courses%2F08%20-%20Remote%20Command%20Execution) 
When you found an client-side injection (CSTI), find out where you are: `{{this}}`, `{{this%2B''}}`, `{{constructor%2B''}}`, `{{constructor.constructor%2B''}}`, `{{eval%2B''}}`, check the CSP, find a sandbox bypass (many are known), and leverage to XSS
When you found an server-side injection (SSTI), find ways to leverage to RCE depending on the language used by the framework
#### SMTP/IMAP Injection
Only of interest when the mail server is not directly accessible, e.g., from the internet.
Terminate previous command with `%0d%0a`, for example, assume `message_id` is used in `FETCH XXX BODY[HEADER]`, an injection could look like this:
`read_email.php?message_id=4791 BODY[HEADER]%0d%0aV100 CAPABILITY%0d%0aV101 FETCH 4791`
The IMAP commands available to an unauthenticated user are: `CAPABILITY`, `NOOP`, `AUTHENTICATE`, `LOGIN`, and `LOGOUT`.

#### Format String Injection
Insert string formatting control characters, e.g., `?username=%25s%25s%25s%25n`
**Spot in Code** #whitebox 
Very bad on C/C++ (printf, fprintf, sprintf, snprintf) and Perl (printf and sprintf) because they can write to memory with `%n`
Okayish on Python (str.format), but can lead to memory read, see [here](https://lucumr.pocoo.org/2016/12/29/careful-with-str-format/)
Can lead to runtime errors in Java (String.format and PrintStream.format) and PHP (printf)
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

### White-Box Testing
Do [[#Best-Practice Guides]] exist for the used technologies? Are they followed?

#### Logging
- Do the logs contain sensitive information?
- Are the logs stored in a dedicated server?
- Can log usage generate a Denial of Service condition?
- How are they rotated? Are logs kept for the sufficient time?
- How are logs reviewed? Can administrators use these reviews to detect targeted attacks?
- How are log backups preserved?
- Is the data being logged data validated (min/max length, chars etc) prior to being logged?

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