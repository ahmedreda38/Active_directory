# Privileged Access
- To move around windows domain we have three options
	1. RDP
	2. Powershell Remoting
	3. MSSQL Server

## Enumerating RDP group members
- To test it on a remote host we can use **Get-NetLocalGroupMember** from powersploits
```powershell
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"
```
## Enumerating SQLAdmins in the domains
- Using Bloodhounds' Cypher Query Language:
```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```
### Enumerating MSSQL instances with PowerUpSQL
```powershell 
Import-Module .\PowerUpSQL.ps1 
Get-SQLInstanceDomain
```
### To login remotely 
#### Windows
```powershell
Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'
```
#### Linux
```bash
Impacket-mssqlclient target.local/myuser:'passowrd'@<ip> -windows-auth
```


## Enumerating canPSRemote users
- using powerview
```powershell
import-module ./PowerView.ps1
Get-DomainGroupMember -Identity "Remote Management Users" | Select-Object MemberName, MemberSID
```
- Enumerating the hosts that a users can PSRemote into
```powershell
Get-netComputer | Get-NetLocalGroupMember -GroupName "Remote Management Users"
```
- for specific user:
```powershell
# Get all computers and check their local Remote Management group for bdavis
Get-DomainComputer | ForEach-Object {
    $computer = $_.Name
    try {
        Get-NetLocalGroupMember -ComputerName $computer -GroupName "Remote Management Users" -ErrorAction Stop | 
        Where-Object { $_.MemberName -like "*bdavis*" } | 
        Select-Object @{N='HostFound';E={$computer}}, MemberName
    } catch {
        # Skip hosts that are offline or unreachable
    }
}
```
### Using Enter-PSSession cmdlet -> PSRemoting
```powershell
$password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force
$cred = new-object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password)
Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $cred

# To get out
Exit-PSSession
```

# Double Hop problem
## what is Double Hop and when does it happen?
- It happen when we authenticate to a windows Host using **evil-winrm** with set of credentails, and `evil-winrm` uses kerberos authentication, so we get a ticket that verifies us to use the target HOST, but our credentials aren't cached, so when we try to access another Host we get `Permission denied`! even if we are authorized and have privilege, **TGT** isn't sent to the Session, Only **TGS**!, And that's why **HOST-A** can prove your identity to **HOST-B** because your credentials/TGT is not cached/stored.
- If **unconstrained delegation** is enabled on a server, it is likely we **won't** face the "Double Hop" problem. In this scenario, when a user sends their TGS ticket to access the target server, their **TGT** ticket will be sent along with the request.
## Workaround!
### 1. Using PSCredentials (from `evil-winrm` - Linux)
```powershell
$pass = ConvertTo-SecureString 'mysecurePa$$1' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('TARGETdomain\myuser', $pass)

# And we can use the $creds object with powerview -Credentials $creds
get-domainuser -spn -credential $Cred | select samaccountname
```
### 2. PSSession Configuration (from domain-joined Host - GUI required) 
1. Register PSSession configurations:
```powershell
Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm
```
2. Restart WinRM Session
```powershell
Restart-Service WinRM
```
3. Re-login using the PSSession Configuration
```powershell
Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm -ConfigurationName backupadmsess
```
