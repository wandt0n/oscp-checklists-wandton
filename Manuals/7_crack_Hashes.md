# Identify Hashes
```
hash-identifier
```

```
hashid $hash
```


# Hashcat
NT Hash
```powershell
hashcat -a 3 -m 900 --encoding-to utf16le NTHashes.txt
```

# John
```
unshadow passwd shadow > unshadowed.txt
```

```
john $file --wordlist=/usr/share/wordlists/rockyou.txt
```

# For unsalted hashes
https://crackstation.net/
