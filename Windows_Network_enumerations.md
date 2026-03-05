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
-Identify Gateways
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
