---
tags:
  - 🔧
aliases: 
templateVersion: 0.4
Created: 2024-10-22
course: Red and Blue Teaming - 141254
lecturer:
  - Martin Grothe
---

___
> [!info] What this does
>  

```Bash
#!/bin/bash
# Bash script to search for subdomains.
# Accepts Domainname

fname="tmp_$1_$RANDOM"
wget -q "https://www.$1" -O $fname

for dn in $(cat $fname | grep -o "[^/]*\.$1" | sort | uniq)
do
	host $dn | awk -F " " '{print $1, " ", $4}'
done
rm $fname
```

## Source


---
```dataview
table without id
    file.link as "Related Topics"
FROM "02 - Secondary Categories"
WHERE contains(file.outlinks, this.file.link) OR contains(file.inlinks, this.file.link) AND file.name != this.file.name
SORT file.name
SORT !(contains(primaryTopic, this.file.link))
```
```dataview
table without id
    file.link as "Related Content", primaryTopic as "Primary Topic"
FROM "03 - Content"
WHERE contains(file.outlinks, this.file.link)
SORT file.name
SORT !(contains(primaryTopic, this.file.link))
```
___



