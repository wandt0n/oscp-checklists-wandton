


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

### Remote Dynamic

```bash
sudo systemctl start ssh
```
(kali machine)

```bash
ssh -N -R 9999 kali@$ip_kali
```
(Machine A)

```bash
nano /etc/proxychains4.conf
```
(kali machine)

##### Use ProxyChains
With it, tools that do not support socks proxys by default, can be made compatible with the tunnel IF they are (most) dynamically-linked binaries. After configuring it (see below), you can use it by prepending the tool's command with `proxychains`.
```bash
nano /etc/proxychains4.conf
```
> For local redirects add `socks5 $hip 9999` to the end.
> For remote redirects use `socks5 127.0.0.1 9999`.
> If socks4 is required, UDP and IPv6 won't be supported.

In use with port scanning, you might lower `tcp_read_time_out` and `tcp_connect_time_out` in the proxychains config file to dramatically increase scanning speed.

### sshuttle
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