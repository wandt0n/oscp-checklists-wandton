### Cookies

| Framework         | Cookie name                       |
| ----------------- | --------------------------------- |
| Zope              | zope3                             |
| CakePHP           | cakephp                           |
| Kohana            | kohanasession                     |
| Laravel           | laravel_session                   |
| phpBB             | phpbb3_                           |
| WordPress         | wp-settings                       |
| 1C-Bitrix         | BITRIX_                           |
| AMPcms            | AMP                               |
| Django CMS        | django                            |
| DotNetNuke        | DotNetNukeAnonymous               |
| e107              | e107_tz                           |
| EPiServer         | EPiTrace, EPiServer               |
| Graffiti CMS      | graffitibot                       |
| Hotaru CMS        | hotaru_mobile                     |
| ImpressCMS        | ICMSession                        |
| Indico            | MAKACSESSION                      |
| InstantCMS        | InstantCMS[logdate]               |
| Kentico CMS       | CMSPreferredCulture               |
| MODx              | SN4[12symb]                       |
| TYPO3             | fe_typo_user                      |
| Dynamicweb        | Dynamicweb                        |
| LEPTON            | lep[some_numeric_value]+sessionid |
| Wix               | Domain=.wix.com                   |
| VIVVO             | VivvoSessionId                    |
| Tiny File Manager | filemanager                       |
| Zenphoto          | zenphoto_auth                     |
### HTML Source Code

|Application|Keyword|
|---|---|
|WordPress|`<meta name="generator" content="WordPress 3.9.2" />`|
|phpBB|`<body id="phpbb"`|
|Mediawiki|`<meta name="generator" content="MediaWiki 1.21.9" />`|
|Joomla|`<meta name="generator" content="Joomla! - Open Source Content Management" />`|
|Drupal|`<meta name="Generator" content="Drupal 7 (https://drupal.org)" />`|
|DotNetNuke|`DNN Platform - [https://www.dnnsoftware.com](https://www.dnnsoftware.com)`|

#### General Markers

- `%framework_name%`
- `powered by`
- `built upon`
- `running`

#### Specific Markers

|Framework|Keyword|
|---|---|
|Adobe ColdFusion|`<!-- START headerTags.cfm`|
|Microsoft ASP.NET|`__VIEWSTATE`|
|ZK|`<!-- ZK`|
|Business Catalyst|`<!-- BC_OBNW -->`|
|Indexhibit|`ndxz-studio`|


Sources:
- [OWASP WSTG](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/08-Fingerprint_Web_Application_Framework)