#service 
<% tp.file.rename("1_SMTP_")%>
☑️

##### Enumerate users
```bash
nmap --script=smtp-enum-users $hip -oN nmap_smtpusers.txt
```
- VRFY username (verifies if username exists)
- EXPN username (verifies if username is valid)
##### Mail Spoof Test
    - HELO anything MAIL FROM: spoofed_address RCPT TO:valid_mail_account DATA . QUIT
##### Weird configurations
Source address: Identical with recipient, <user@unknown_domain>, <user@localhost>, user, "", IP address of target server <user@IP_Address>
Recipient address: Double quotes <"user@recipent-domain">, <nobody@recipient_domain@[IP Address]>
Formatting: mail from: <user@[IP Address]> rcpt to: <@domain:nobody@recipient-domain>, mail from: <user@[IP Address]> rcpt to: <recipient_domain!nobody@[IP Address]>
# Findings

# Loot
```bash
find / -name "sendmail.cf" -o -name "submit.cf"
```