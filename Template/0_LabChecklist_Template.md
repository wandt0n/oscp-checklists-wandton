<% tp.file.rename("0_LabChecklist")%>
Change SUBNET in the following line (keep the point)

```
export subnet="SUBNET." && nmap -T 5 -sn $subnet"1-253" -oG - | grep "Status: Up" | cut -d " " -f 2 > 0_Hosts.txt
```
---
- [ ] Find local DNS Servers and attempt transfer
```
sudo nmap -T 5 -sU -p U:53 T:53 --open -iL 0_Hosts.txt
```
   DNS Server found:

##### Probe DNS Server
- [ ] Zone Transfer
```
sudo nmap -T 5 --script=dns-zone-transfer -p 53 $dnsserver -oN nmap_dnsZoneTransfer
```
   Findings:
   
- [ ] Reverse DNS Lookup
```
for ip in $(seq 1 254); do host $subnet$ip $dnsserver; done | grep "arpa" | grep -v "not found"
```
   Findings:
   
##### Continue
---
- [ ] ARP bruteforce
```
for ip in $(seq 1 255);do host $subnet$ip $dns_server | grep "arpa" | grep -v "not found"; done
```
   Findings:
   
---
Create working dirs and start individual system scanning

```
for i in $(cat 0_Hosts.txt); do (mkdir $i && cd $_ && sudo nmap --osscan-guess -T 5 -A -p- $i -oX - | xsltproc -o 0_overview.html - && firefox 0_overview.html && sudo nmap -T 5 -sUV --top-ports 100 $i -oN 0_udp_top100.txt)& done
```

---
### If you're stuck

NBTScan
`sudo nbtscan -r $subnet0/24`

https://book.hacktricks.xyz/
