# Whitebox
## General
`rg -n --hidden --follow -S   '(?i)\b(pass(word)?|pwd|secret|api[_-]?key|token)\b\s*[:=]\s*["'\'']?[^"'\''\s]{6,}'   .`
## Web
### File Inclusions
PHP: `--include "*.php" -e "include(" -e "include_once(" -e "require(" -e "require_once(" -e "fopen(" -e "readfile(`
JSP/Servlet: `-e "java.io.File(" -e "java.io.FileReader("`
ASP: `-e "include file" -e "include virtual"`