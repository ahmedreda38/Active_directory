# Hardening Active Directory
## Understand the environment
- For us to secure our Active directory environment, we really need to know it's structure and components first, which will help us construct an accurate Threat modelling, and rank the assets and their importance and impact of compromise for each component
- Map the organization resources and buildup an Assets inventory
#### Things To Document and Track
- `Naming conventions of OUs, computers, users, groups`
- `DNS, network, and DHCP configurations`
- `An intimate understanding of all GPOs and the objects that they are applied to`
- `Assignment of FSMO roles`
- `Full and current application inventory`
- `A list of all enterprise hosts and their location`
- `Any trust relationships we have with other domains or outside entities`
- `Users who have elevated permissions`
## People
1. Enforce Strong password policy and ban the use of common words
2. Clean up the privileged groups
3. Disallow local administrator access on user workstations unless a specific need
4. Disable the default `RID-500 local admin` account and create a new admin account for administration subject to LAPS password rotation.
5. implement mulit-tier administration for Admins
6. Use `Protected Users` Group when neeed
## Processes
- For the company to deal with incidents and Hold people accountable for what cause they were the reason for, there has to be a Firm and Well-defined policy and procedures
1. Proper policies and procedures for AD asset management
2. Access control policies (user account provisioning/de-provisioning), multi-factor authentication mechanisms.
3. Processes for provisioning and decommissioning hosts (i.e., baseline security hardening guideline, gold images)
4. AD cleanup policies
    - `Are accounts for former employees removed or just disabled?`
    - `What is the process for removing stale records from AD?`
    - Processes for decommissioning legacy operating systems/services (i.e., proper uninstallation of Exchange when migrating to 0365).
    - Schedule for User, groups, and hosts audit.

### Technology

Periodically review AD for legacy misconfigurations and new and emerging threats. As changes are made to AD, ensure that common misconfigurations are not introduced. Pay attention to any vulnerabilities introduced by AD and tools or applications utilized in the environment.

- Run tools such as BloodHound, PingCastle, and Grouper periodically to identify AD misconfigurations.
- Ensure that administrators are not storing passwords in the AD account description field.
- Review SYSVOL for scripts containing passwords and other sensitive data.
- Avoid the use of "normal" service accounts, utilizing Group Managed (gMSA) and Managed Service Accounts (MSA) where ever possible to mitigate the risk of Kerberoasting.
- Disable Unconstrained Delegation wherever possible.
- Prevent direct access to Domain Controllers through the use of hardened jump hosts.
- Consider setting the `ms-DS-MachineAccountQuota` attribute to `0`, which disallows users from adding machine accounts and can prevent several attacks such as the noPac attack and Resource-Based Constrained Delegation (RBCD)
- Disable the print spooler service wherever possible to prevent several attacks
- Disable NTLM authentication for Domain Controllers if possible
- Use Extended Protection for Authentication along with enabling Require SSL only to allow HTTPS connections for the Certificate Authority Web Enrollment and Certificate Enrollment Web Service services
- Enable SMB signing and LDAP signing
- Take steps to prevent enumeration with tools like BloodHound
- Ideally, perform quarterly penetration tests/AD security assessments, but if budget constraints exist, these should be performed annually at the very least.
- Test backups for validity and review/practice disaster recovery plans.
- Enable the restriction of anonymous access and prevent null session enumeration by setting the `RestrictNullSessAccess` registry key to `1` to restrict null session access to unauthenticated users.
