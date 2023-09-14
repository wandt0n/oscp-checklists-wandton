### For Loop
`for i in $(command | tr '\n' ' '); do command; done`

### Set Terminal name
```bash
export PS1="$PS1\[\e]0;NAME\a\]"
```
### Fix Metasploit
sudo gem install bundler -v 2.2.4; sudo msfdb reinit; sudo msfconsole
(https://stackoverflow.com/questions/65941317/metasploit-crashed-after-upgrade)


### Switch Python versions
List available python installs: `pyenv versions`
Set version as default for python: `pyenv global [version]`
(https://www.kali.org/docs/general-use/using-eol-python-versions/)
