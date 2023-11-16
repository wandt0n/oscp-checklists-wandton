#service 
<% tp.file.rename("1_MySQL_")%>
☑️

## Connect to MySQL
```bash
mysql -u root -p -h $hip
```
(-h : hostname)

## Get Tablenames
### MySQL
```
SELECT * FROM information_schema.columns where table_schema=database()
```
### SQLite
```
select * from sqlite_master where type = 'table'
```
### Postgres
```
SELECT distinct table_name FROM information_schema.columns
```


# Findings
