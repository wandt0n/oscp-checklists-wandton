# Whitebox
## General
### Secrets
`rg -n --hidden --follow -S   '(?i)\b(pass(word)?|pwd|secret|api[_-]?key|token)\b\s*[:=]\s*["'\'']?[^"'\''\s]{6,}'   .`
### Weak Crypto Algorithms
`-e MD4 -e MD5 -e RC4 -e RC2 -e DES -e Blowfish -e SHA-1 -e ECB -e IV -e salt`
## Web
### File Inclusions
PHP: `--include "*.php" -e "include(" -e "include_once(" -e "require(" -e "require_once(" -e "fopen(" -e "readfile(`
JSP/Servlet: `-e "java.io.File(" -e "java.io.FileReader("`
ASP: `-e "include file" -e "include virtual" -e "OpenTextFile("`

### Code Execution
ASP: `-e "Execute("`
Java: `-e "Runtime.exec("`
C/C++: `-e system -e exec -e ShellExecute`
Python: `-e exec -e eval -e os.system -e os.popen -e subprocess.popen -e subprocess.call`
PHP: `-e system -e shell_exec -e exec -e proc_open -e eval`
Node.js:`-e "child_process.exec(" -e "execSync(" -e "spawn("`
Ruby: `-e "system(" -e "exec(" -e "Open3.popen3("`
Go: `-e "exec.Command("`
C#: `-e "Process.Start("`
### Regex
PHP: `--include "*.php" -e "startswith(" -e "endswith(" -e "contains(" -e "indexOf("`