# Kerberos
- kerberos is the main authentication & authorization Protocol used in microsoft's Active directory, it is stateless protocol, and it operates via Ticket Granting Tickets (TGT) And Ticket Granting Service (TGS)
- it uses Port 88 (TCP & UDP)
- the Domain controller is the one responsible for hosting the Key Distribution Center (KDC)
- ![Kerberos Flow](../images/kerberos.png)
- Mainly the when enumerating AD environment, we look for the ip address that has the KDC on it (Port 88 opened) then we can identify it as the Domain Controlled

# DNS
- In ADDS, DNS is used to hold the mapping between Hostnames and Ip addresses, And it hold a database that conatians SCV -> `Service Records`, which is the entry for each service and their location within the AD environment
- It uses Port 53 (UDP & TCP), Default is UDP, it falls back to TCP when the request size exceeds 512 bytes
- We can utilize `nslookup` and analyze the network traffic to see how the DNS query system works

# LDAP
- Lightweight Directory Access Protocol is the Backbone of AD-DS, It is an open-source and cross-platform protocol used for authentication against various directory services (Such as active directory)
- it uses Port **389**, and LDAP over SSL (LDAPS) communicates over port **636**.
- LDAP uses 2 authentication mechanisms
  1. Simple authentication: Using Username and password to create a Bind authentication Request
  2. SASL authentication: Uses other methods for authentication, such as kerberos
- LDAP auth messages are sent in cleartext, so it's better to safegaurd it with TLS
  
# MSRPC
- It is Microsoft's Implementation of Remote Procedural Calls (RPC)
- MSRPC has 4 key interfaces
   1. **lsarpc**: A set of RPC calls to the Local Security Authority (LSA) system which manages the local security policy on a computer, controls the audit policy
   2. **netlogon**: Netlogon is a Windows process used to authenticate users and other services in the domain environment. It is a service that continuously runs in the background.
   3. **samr**: Remote SAM (samr) provides management functionality for the domain account database, storing information about users and groups. we can use the samr protocol to perform reconnaissance about the internal domain using tools such as `BloodHound`,  Organizations can protect against this type of reconnaissance by changing a **Windows registry key** to only allow administrators to perform remote SAM queries since, by default, all authenticated domain users can make these queries to gather a considerable amount of information about the AD domain.
   4. **drsuapi**: drsuapi is the Microsoft API that implements the **Directory Replication Service** (DRS) Remote Protocol which is used to perform replication-related tasks across Domain Controllers in a multi-DC environment. Attackers can utilize drsuapi to create a copy of the Active Directory domain database (NTDS.dit) file to retrieve password hashes for all accounts in the domain