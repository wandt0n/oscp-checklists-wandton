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

Also: Useful webencoded chars:
- `%20` == ` `
- `%23` == `#`
- `%2E` == `.`
- `%3B` == `;`
- `%3E` == `<`
- `%26` == `&`
- `%7C` == `|`