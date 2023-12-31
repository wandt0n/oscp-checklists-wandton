### For Loop
`for i in $(command | tr '\n' ' '); do command; done`
e.g. iterate over found ips in 172er subnet: `for hip in $(cat /etc/hosts | grep 172 | cut -f 1 | tr '\n' ' '); do command $hip; done`
### Create list of usernames from Obsidian Notes
```bash
find . -name "9*.md" -print | cut -d "/" -f2 | cut -d "." -f1 | cut -d "_" -f2 | cut -d " " -f 1 > users.txt
```

### Multiple Layers of quotes
Linux: Use Arrays.
```bash
ARG1=("works"); yes "that ${ARG1[@]}"
```

Windows: Escape quotes with `\"` in CMD and with a backtick \`" in Powershell. Or use variables like `$ARG1 = "works" ; yes "that $ARG1"`

### Change IP-Tupel in /etc/hosts
```bash
sudo sed -i 's/old/new/g' /etc/hosts
```
### Find Flag
```bash
grep -rn / -e "OS{" 2>/dev/null
```
### Set Terminal name
```bash
export PS1="$PS1\[\e]0;NAME\a\]"
```
### Fix Metasploit
sudo gem install bundler -v 2.2.4; sudo msfdb reinit; sudo msfconsole
(https://stackoverflow.com/questions/65941317/metasploit-crashed-after-upgrade)

### Windows
Restart Computer with Powershell
```powershell
Restart-Computer -Force
```
Create New Admin User
```powershell
$pw=ConvertTo-SecureString -AsPlainText -Force -String "Admin1234=" ; New-LocalUser "tempadmin" -Password $pw -FullName "Petrus Testus" -Description "Temporary local admin"; Add-LocalGroupMember -Group "Administrators" -Member "tempadmin"
```
Change Group membership
`Add-LocalGroupMember -Group "Remote Desktop Users" -Member foo`
### Switch Python versions
List available python installs: `pyenv versions`
Set version as default for python: `pyenv global [version]`
(https://www.kali.org/docs/general-use/using-eol-python-versions/)
