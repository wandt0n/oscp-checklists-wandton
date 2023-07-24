Enable Telnet
```powershell
dism /online /Enable-Feature /FeatureName:TelnetClient
```

Portscan LOLBAS
```powershell
1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("undefined", $_)) "TCP port $_ is open"} 2>$null
```
