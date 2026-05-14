- Tool built totally in `C`.
- precompiled binaries, [https://github.com/gentilkiwi/mimikatz/releases](https://github.com/gentilkiwi/mimikatz/releases)
### What has `mimikatz` done to microsoft?
- prob lots of damage, but this was because microsoft responds that this tool need an administrator, and it is the software design that can't be fixed, and after lots of big attacks and large money losses, microsoft decided to put some security controls to try protecting against some of what mimikatz does, such as:
	1. Credential Guard
	2. Protected Process light - PLL
	3. Windows Defender Application Control - WDAC
	4. Enhanced logging
### Where can we find it?
- We can find it on most of the C2 frameworks, that help inject it into memory 
- precompiled binary at [https://github.com/gentilkiwi/mimikatz/releases](https://github.com/gentilkiwi/mimikatz/releases)
- within metasploit-framework's meterpreter shell -> `load_kiwi`
- Powershell Module at [Invoke-Mimikatz/Invoke-Mimikatz.ps1 at master · g4uss47/Invoke-Mimikatz](https://github.com/g4uss47/Invoke-Mimikatz/blob/master/Invoke-Mimikatz.ps1)


## Foundational Background

It uses the `SeDebugPrivilege` to open any process for debugging purpose and in our case, the target process is `LSASS`

---
#### WoW64
`Windows on Windows 64` , Is a compatibility subsystem for 32bit programs to run on 64bit Windows arch, and it works by emulating a 32bit environment for the program, and it maps the `dll` calls to the right directory:

When a 32-bit program asks for:
```
C:\Windows\System32\kernel32.dll
```
WoW64 silently redirects it to the 32-bit version:
```
C:\Windows\SysWOW64\kernel32.dll
```
Same idea with registry:
```
HKLM\SOFTWARE
```
from a 32-bit process may transparently become:
```
HKLM\SOFTWARE\Wow6432Node
```

```
FLIPPED :(
C:\Windows\System32   -> 64-bit system binaries
C:\Windows\SysWOW64  -> 32-bit system binaries
```


- so on windows 64bit versions, mimikatz needs to run with a OS architecture that meets the main OS arch on the windows to access the windows system executables/processes

---
#### LSASS.exe
- it is the authentication brain of the windows, it helps in windows authentication and authorization and it provides session tokens for having a state and not prompting users to login every minute/request

|LSASS Responsibility|Credential Impact|
|---|---|
|User authentication|Stores plaintext passwords (pre-Win8.1), NT hashes|
|Security token creation|Contains logon session information|
|Password change processing|Temporarily stores old/new credentials|
|Kerberos ticket management|Caches TGTs and service tickets|
|NTLM authentication|Stores NT hashes for SSO operations|
|Credential caching|Maintains domain cached credentials|
#### Windows Security Token Architecture
```
┌─────────────────────────────────────────┐
│           SECURITY TOKEN                │
├─────────────────────────────────────────┤
│  User SID          │ S-1-5-21-...-1001  │
│  Group SIDs        │ Administrators,    │
│                    │ Users, etc.        │
│  Privileges        │ SeDebugPrivilege,  │
│                    │ SeBackupPrivilege  │
│  Logon Session ID  │ 0x00000000:003E7   │
│  Token Type        │ Primary/Imperson.  │
│  Integrity Level   │ High/Medium/Low    │
└─────────────────────────────────────────┘
```
- it is a kernel object that holds the security context of a process or a thread
- it says the followings:
	1. who is the user
	2. what groups does this user belong to
	3. what special privileges he has
	4. session ID
	5. Is is the primary logon session or impersonated from another thread/prcess 
	6. Process integrity Level which defines the access limitation for the process 

##### Token Types and Impersonation Levels[¶](https://darkoperator.github.io/mimikatz-missing-manual/02-Basics/#token-types-and-impersonation-levels "Permanent link")

|Token Type|Description|Mimikatz Relevance|
|---|---|---|
|Primary Token|Assigned to processes at creation|Determines base privilege level|
|Impersonation Token|Used for client impersonation|Can be stolen with `token::elevate`|

|Impersonation Level|Network Access|Local Access|
|---|---|---|
|Anonymous|No|No|
|Identification|No|Limited|
|Impersonation|No|Yes|
|Delegation|Yes|Yes|
```
Impersonation = local access as the user
Delegation   = network + local access as the user (abused in active directory delegation attacks)
```

