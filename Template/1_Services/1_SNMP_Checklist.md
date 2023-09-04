<% tp.file.rename("1_SNMP_")%>

## Find hosts with onesixtyone
```bash
echo -e "private\npublic\nmanager\ncable-docsis\nILMI" > snmp_communityStrings.txt
```

```bash
onesixtyone -c snmp_communityStrings.txt -i ips.txt
```
	

## Enumerate MIB Tree
The whole Tree
```bash
snmpwalk -c public -v1 -t 10 $hip > snmp_tree.txt
```
Or specific lleafs of interest:
```bash
snmpwalk -c public -v1 $ip 
```
("public" is the default community string. It might be changed by the netadmins)

| Target | Leaf | Finding | 
| - | - | - |
|Installed Software|1.3.6.1.2.1.25.6.3.1.2||
|Open TCP Ports|1.3.6.1.2.1.6.13.1.3||
|Running Programs|1.3.6.1.2.1.25.4.2.1.2||
|Windows Users|1.3.6.1.4.1.77.1.2.25||
|Storage Units|1.3.6.1.2.1.25.2.3.1.4||
|System Processes|1.3.6.1.2.1.25.1.6.0||
|Processes Path|1.3.6.1.2.1.25.4.2.1.4||


# Findings