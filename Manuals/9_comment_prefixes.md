---
tag: 🔧
aliases: []
templateVersion: 0.3
Created: 2024-02-21
---

___
> [!info] What this does
>  

### JavaScript
`//`
`<!-- -->` (Inline comment! Legacy thing to work with old browsers)
`#!`

### CSS
`/* */`

### SQL
`#`
`/*`
`-- Comment` (Whitespace is only required in MySQL)
`;%00`

### PHP
`#`
`//`
`/* */`
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



