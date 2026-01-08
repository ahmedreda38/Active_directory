## Object
- Any Resource present inside AD environment such as OUs, Printers, Users, DCs, etc
## Attributes
- each Object has a set of attributes, Computer Object might have different set of attributes than User Object
## Schema
- it's like a blueprint for the structure of the AD, such as the class attributes and methods, when creating a computer object with the name of MSW01 we just created an instance of the Computer Class with its following attributes and methods
## Domain
- A domain is a logical group of objects such as computers, users, OUs, groups, etc, different domains could have trust realation between them if needed
## Forest
- A forest is a collection of Active Directory domains. It is the topmost container and contains all of the AD objects introduced below,
## Tree
- A tree is a collection of Active Directory domains that **begins at a single root domain**
## Container
- Container objects hold other objects and have a defined place in the directory subtree hierarchy.
## Leaf
- Leaf objects do not contain other objects and are found at the end of the subtree hierarchy.
## Global Unique Identifier - GUID
- unique 128-bit assigned to each domain object at its creation, and it is the most accurate and reliable attributes when searching for an object inside AD env, and it never changes for a created object.
## Security Identify - SID
- Unique Identifier for a security Group / Principal with certain permissions and policies such as the Administrator's SID ending with 500 -> `S-1-5-21domain-500`, Other [LDAPWiki: Well-known Security Identifiers](https://ldapwiki.com/wiki/Wiki.jsp?page=Well-known%20Security%20Identifiers)
## Distinguished Name - DN 
- it is the full path to an object, such as `cn=bjones, ou=IT, ou=Employees, dc=inlanefreight, dc=local`
## Relative Distinguished Name - RDN
- RDN is the bjones at the end, it has to be unique accross an Organizational Unit - OU, But can be different across domains.
## sAMAccountName
User's Logon name <= 20 chars
## userPrincipalName
- it an optional attribute that comes in this format `user@domail.local`
## Flexible Single Master Operation - FSMO roles
FSMO roles: 
1. Schema Master`
2. `Domain Naming Master` (one of each per forest)
3. `Relative ID (RID) Master` (one per domain)
4. `Primary Domain Controller (PDC) Emulator` (one per domain)
5. `Infrastructure Master` (one per domain). 
All five roles are assigned to the first DC in the forest root domain in a new AD forest. Each time a new domain is added to a forest, only the RID Master, PDC Emulator, and Infrastructure Master roles are assigned to the new domain.
## Global Catalog - GC
A DC that stores replica of all objects in AD forest, full copy for current domain and partial copy for other domains
- it performs authentication / authorization for all groups a user belong to
- it performs Object Search by providing a ingle attribute
## Read-Only Domain Controller (RODC)
From its name it is a read-only AD database.
- It does not cache users Account passwords except RODC comp account and RODC KRBTGT
## Replication
## Access Control List - ACL
- is the ordered collection of Access Control Entries (ACEs) that apply to an object.
## Access Control Entry - ACEs
- identifies a trustee (user account, group account, or logon session) and lists the access rights that are allowed, denied, or audited for the given trustee
## Discretionary Access Control List - DACL
 - List of ACEs that are checked when a process tries to access a securable object
	 - If object does not have DACL: system will give full control to everyone
	 - If Object have DACL with 0 ACEs: System will deny all access attempts 
## System Access Control List - SACL
- secure ACEs that triggers security even logging for access attemps
## Fully Qualified Domain Name - FQDN
Complete name for a computer inside a domain
## Tombstone
Stores deleted Objects for 90-180 days, but with trimming the attributes
## AD Recycle Bin
Stores Deleted Objects for 60 days with the full attributes for the latest state
## SYSVOL
stores copies of public files in the domain such as system policies, Group Policy settings, logon/logoff scripts
## adminCount
it's an attributes that defines whether or not the SDProp process protectsthis user or not, we look for users with `adminCount = 1`
## sIDHistory
## NTDS.DIT
The NTDS.DIT file can be considered the heart of Active Directory. It is stored on a Domain Controller at `C:\Windows\NTDS\` and is a database that stores AD data such as information about user and group objects, group membership, and, most important to attackers and penetration testers, the password hashes for all users in the domain.
