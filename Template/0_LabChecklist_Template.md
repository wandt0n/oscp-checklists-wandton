<%*
const path = tp.file.folder(true).split('/');
const filename = "0_A_Network" + path[path.length - 1];
await tp.file.rename(filename)
-%>
#show

1. Clean the `/etc/hosts` file and link it with `ln -s /etc/hosts ./0_Hosts`
2. Update Searchsploit with `searchsploit -u`
3. Update Windows Exploit suggester with `wes --update`
4. Remove neo4j db with `sudo rm -rf /etc/neo4j/data/*`

<font style="color:red">NEXT TIME, START TO LOG TO TRAFFIC VOLUME WITH IPTABLES, SO I CAN CHECK WHETHER MY MOBILE FLAT WOULD BE SUFFICIENT</font>
# Find Hosts in Subnet

Change SUBNET in the following line (keep the point)

```
export subnet="SUBNET." && nmap -T 5 -sn $subnet"1-254" -oG - | grep "Status: Up" | cut -d " " -f 2 >> 0_Hosts.txt
```
---
Find local DNS Servers and attempt transfer
```
sudo nmap -T 5 -sU -p U:53 T:53 --open -iL 0_Hosts.txt
```
   DNS Server found:

##### Probe DNS Server
Zone Transfer
```
sudo nmap -T 5 --script=dns-zone-transfer -p 53 $dnsserver -oN nmap_dnsZoneTransfer
```
   Findings:
   
Reverse DNS Lookup
```
for ip in $(seq 1 254); do host $subnet$ip $dnsserver; done | grep "arpa" | grep -v "not found"
```
   Findings:
##### Continue
---
##### Search SMB Shares
```bash
sudo nbtscan -r $subnet0/24
```

```bash
/home/kali/Documents/activeInformationGathering/snaffler.exe -s -o snaffler.log -n $(cat 0_Hosts.txt | sed -z 's/\n/,/g;s/,$/\n/')
```
> Does not work on Linux, as of yet
##### Create working dirs and start individual system scanning

```bash
for i in $(tail -n 4 0_Hosts); do (mkdir -p "A_$i" && cd $_ && sudo nmap --osscan-guess -T 5 -A -p- $i -oX - | xsltproc -o 0_overview.html - && firefox 0_overview.html && sudo nmap -sUV --top-ports 100 $i -oN 0_udp_top100.txt)& done
```

For single host:
```bash
i="";sudo nmap --osscan-guess -T 5 -A -p- $i -oX - | xsltproc -o 0_overview.html - && firefox 0_overview.html && sudo nmap -T 5 -sUV --top-ports 100 $i -oN 0_udp_top100.txt
```

---
# Findings