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


# Findings
