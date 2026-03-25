---
tag: 🔧
aliases: []
templateVersion: 0.4
Created: 2024-10-14
---

___
> [!info] What this does
>  


1. Create Payload
2. rename it, e.g. in bash with `cp test.txt $'kompl\u202efdp.exe`
3. It will appear as komplexe.pdf

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



