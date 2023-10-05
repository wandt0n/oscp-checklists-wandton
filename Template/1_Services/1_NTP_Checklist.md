#service 
<% tp.file.rename("1_NTP_U123")%>
☑️

```bash
ntpq -c "host" -c "hostname" -c "version" -c "ntpversion" -c "readlist" -c "readvar" -c "associations" -c "peers" -c "sysinfo" $hip
```

# Findings

# Loot
```bash
find / -name "ntp.conf"
```