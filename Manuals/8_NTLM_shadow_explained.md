
| | | | | 
|-|-|-|-|
| $Administrator:$|$500:$|$NO PASSWORD*********************:$|$BE134K40129560B46534340292AF4E72:::$|
|USER     |UID|            LM Hash             |     NT Hash (also called NTLM)|


CrackStation -> RockYou.txt (john)

If there IS an LM Hash present:
1. Copy whole lines into hashfile (user, uid, LMHash and NTHash)!
2. Run john with --format=LM (he will identify 2 hashes per lines, as LMHashes can be cracked partly)
3. Run john without attack type (e.g. wordlist) but with --show -> If there are ???? left, this part of the LM wasn't cracked yet. But it needs to.
   ( I've created a script to check spelling against the NTML Hash. It needs to be tested in a valid case though )
4. Create wordlist that only contains the found password canidate. It's in uppercase.
5. Run john with --rules, --format=NT and --wordlist=(the created wordlist) to get the password correctly spelled.



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

