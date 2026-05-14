# Active Directory Pentesting Notes

Personal Active Directory pentesting notes, cheatsheets, and command references for authorized labs and assessments.

> Simple Notes I take while studying..

## Quick Start

If you are new to the repo, follow this path:

1. Build the base concepts from [AD Structure](#ad-structure).
2. Work through discovery in [Enumeration](#enumeration).
3. Validate weaknesses from [Attacks](#attacks).
4. Continue with [Movement and Compromise](#movement-and-compromise).
5. Use [Tools](#tools) and [Defense](#defense) as supporting references.


## AD Structure

| Note | Purpose |
| --- | --- |
| [AD structure](./AD-Structure/AD_structure.md) | Core AD components, OUs, trusts, and replication. |
| [Active Directory Objects](./AD-Structure/Active_Directory_Objects.md) | Object types, security principals, and pentest relevance. |
| [Important Terminologies](./AD-Structure/Important_Terminologies.md) | AD vocabulary such as SID, DN, GUID, ACL, DACL, SYSVOL, and NTDS.dit. |
| [Important Protocols](./AD-Structure/Important-Protocols.md) | Kerberos, DNS, LDAP, MSRPC, SAMR, and DRSUAPI basics. |
| [NTLM Authentication](./AD-Structure/NTLM-auth.md) | LM, NT hash, NTLMv1, NTLMv2, and cached credentials. |
| [FSMO and Trusts](./AD-Structure/FSMO-&-Trust.md) | FSMO roles, trust types, and transitive versus non-transitive trusts. |
| [Users, Computers, and Groups](./AD-Structure/Users-Computers-Groups.md) | Group types, scopes, nested membership, and useful attributes. |

## Enumeration

| Note | Purpose |
| --- | --- |
| [General Enumeration](./Enumeration/Enumerations.md) | Workflow-based enumeration note with a table of contents. |
| [Windows Network Enumerations](./Enumeration/Windows_Network_enumerations.md) | DNS, DC discovery, firewall enumeration, and LOLBINS. |
| [BloodHound](./Enumeration/Bloodhound.md) | BloodHound collection and PowerView-based manual enumeration. |
| [PowerView](./Enumeration/PowerView.md) | PowerView loading and quick usage notes. |

## Attacks

| Note | Purpose |
| --- | --- |
| [Password Spraying](./attacks/Password_spray.md) | Password policy checks, target user lists, and password spraying commands. |
| [Credential Harvesting](./attacks/Credentials_Harvesting.md) | Local credential search, Responder, SMB signing, and NTLM relay setup. |
| [AS-REP Roasting](./attacks/AsRepRoasting.md) | Kerberos pre-authentication weakness, extraction, and cracking workflow. |
| [Kerberoasting](./attacks/Kerberoasting.md) | SPN enumeration, TGS extraction, and offline cracking workflow. |
| [NTLM Relay](./attacks/NTLM_Relay.md) | NTLM relay concepts, requirements, attack flow, and LLMNR/NBT-NS poisoning. |
| [ADCS Attacks](./attacks/ADCS_ATTACKS.md) | AD Certificate Services ESC1, ESC2, ESC3, ESC4, and ESC8 examples. |
| [DACL Abuses](./attacks/DACL-Abuses.md) | GenericAll, GenericWrite, WriteOwner, WriteDacl, ForceChangePassword, AddSelf, RBCD, and related abuses. |
| [GPO Abuse](./attacks/GPO_Abuse.md) | Group Policy Object abuse notes from the source vault. |
| [Common Active Directory Vulnerabilities](./attacks/Common_Active-Directory_Vulnerabilities.md) | ZeroLogon, noPac, PrintNightmare, PetitPotam, and version checking notes. |
| [Misc Misconfigs](./attacks/Misc_Misconfigs.md) | Exchange, printer, DNS, GPP, and password-related misconfiguration notes. |

## Movement and Compromise

| Note | Purpose |
| --- | --- |
| [Local Privilege Escalation](./Movement-and-Compromise/Local%20Privilege%20Escalation.md) | PowerUp, SeImpersonatePrivilege, SeDebugPrivilege, and related local privilege escalation notes. |
| [Privileged Access](./Movement-and-Compromise/Privileged_Access.md) | RDP, PowerShell Remoting, MSSQL, and Double Hop notes from the source vault. |
| [DCSync](./Movement-and-Compromise/DCSync.md) | Replication-based domain credential dumping and Golden Ticket notes. |
| [Domain Trusts](./Movement-and-Compromise/Domain_Trusts.md) | Domain trust, ExtraSids, and cross-forest notes from the source vault. |
| [File Upload](./Movement-and-Compromise/File%20upload.md) | File transfer notes copied from the source vault. |
| [Pivoting with Ligolo-ng](./Movement-and-Compromise/Pivoting-Ligolo-ng.md) | Placeholder for Ligolo-ng pivoting notes. |

## Tools

| Note | Purpose |
| --- | --- |
| [Mimikatz](./Tools/Mimikatz.md) | Mimikatz notes copied from the source vault. |
| [Metasploit](./Tools/Metasploit.md) | Metasploit notes copied from the source vault. |
| [SliverC2](./Tools/SliverC2.md) | SliverC2 notes copied from the source vault. |

## Defense

| Note | Purpose |
| --- | --- |
| [AD Hardening](./Defense/AD_Hardening.md) | Hardening Active Directory notes from the source vault. |



