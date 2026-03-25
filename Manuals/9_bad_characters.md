---
tags:
  - oscp
  - 🔧
aliases: 
templateVersion: 0.3
Created: 2024-02-21
---

---
> [!info] What this does


`nvgt;` [[02 - Hyper Text Markup Language|HTML]] Entity consists of two Unicode characters
	U+0003E -> `>`
	U+020D2 -> Control character to negate
=> Can be used to bypass sanitization of `>`

`NULL`, `null`
	- Possible error messages returned.
`' , " , ; , <!`
	- Breaks an SQL string or query; used for SQL, XPath and XML Injection tests.
`– , = , + , "`
	- Used to craft SQL Injection queries.
`‘ , &, ! , ¦ , < , >`
	- Used to find command execution vulnerabilities.
`"><script>alert(1)</script>`
	- Basic Cross-Site Scripting Checks.
`%0d%0a`
	- Carriage Return (%0d) Line Feed (%0a)
`%7f` , `%ff`
	- byte-length overflows; maximum 7- and 8-bit values.
- -1, other
	- Integer and underflow vulnerabilities.
`%n` , `%x` , `%s`
	- Testing for format string vulnerabilities.
- `../`
	- Directory Traversal Vulnerabilities.
- `%` , `_`, `*`
	- Wildcard characters can sometimes present DoS issues or information disclosure.
- `Ax1024+`
	- Overflow vulnerabilities.

# Web

| Zeichen | HTML     | JS     | URL |
| ------- | -------- | ------ | --- |
| <       | `&lt;`   | \u003c | %3c |
| >       | `&gt;`   | \u003e | %3e |
| "       | `&quot;` | \u0022 | %22 |
| '       | `&apos;` | \u0027 | %27 |
| &       | `&amp;`  | \u0026 | %26 |
| ` `     |          |        | %20 |
| #       |          |        | %23 |
| .       |          |        | %2e |
| ;       |          |        | %3b |
| \|      |          |        | %7c |
### Where can I escape how?

| Context     | Encoding  | a =      | Description             |
| ----------- | --------- | -------- | ----------------------- |
| Everywhere* | \uCODE    | \u0061   | Unicode escape          |
| String      | \xHEX     | \x61     | Hexadecimal escape      |
| String      | \OCT      | \141     | Octal escape            |
| URL**       | %         |          | URL encoding, see above |
| HTML***     | `&amp;`   |          | Keyword entities        |
| HTML***     | `&#DEC;`  | `&#97;`  | Decimal entities        |
| HTML***     | `&#xHEX;` | `&#x61;` | Hexadecimal entities    |


|      |                                                                                                                                                                                               |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| \*   | cannot do whitespace, brackets, ...                                                                                                                                                           |
| \**  | cannot encode the protocol directly                                                                                                                                                           |
| \*** | - e.g., `<svg><script>here` (html-enc due to the svg)<br>- Semicolon can often be omitted<br>- Encoding in attribute values will be decoded first<br>- `/` can be used to separate attributes |


Unicode spaces: https://www.jkorpela.fi/chars/spaces.html



