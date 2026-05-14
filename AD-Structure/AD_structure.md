# What is active directory?
- it is a directory service by microsoft, which manages authentication, authorization, services, groups and policies
- it stores users, computers credentials and handles the management of groups servers and workstations

## Core components
1. **Domain**: logical grouping of objects under a unique name
   - (Workstation01 + Workstation02 + Printer + MSSQL) Grouped logically together
2. **Domain Controller**: A SERVER that controls (responds to) authentication requests and service access requests
3. **Forest**: Top-level container that holds one or more domains. (Can hold Trees within it's structure)
4. **Tree**: collection of domains that share a contiguous namespace (they share the smae top-level domain -> *.minyawy.local)
<img width="877" height="335" alt="Different between a tree and a forest in AD" src="https://github.com/user-attachments/assets/9424e458-15ae-474b-85fe-9b7170d31d69" />

## Organizational units (OU)
OUs are used to organize AD objects into logical administrative containers. Organizational Units are similar to folders in the file system, in which other AD objects can be stored.

### Purpose and Functions of OUs - Organizational Units are used for:
1. **Logical** and hierarchical structuring of AD objects.
2. Allows to delegate administrative permissions to non-admin users and groups for all objects in the OU without granting them the domain administrator privileges;
3. Can be used to assign Group Policy Objects (GPOs) to users and computers in an OU.

## Trusts
enable secure authentication and resource sharing between domains or forests. A trust defines how authentication requests flow and whether access is allowed across boundaries.

### Types of Trusts:
1. **One-way**: Only one domain trusts the other. For example, in a one-way trust from Domain A to Domain B, users in Domain A can access Domain B’s resources, but not vice versa.
2. **Two-way**: Both domains trust each other, allowing mutual authentication.
3. **Transitive**: Extends trust relationships beyond the two domains directly involved, enabling authentication across multiple domains in a forest.
4. **Non-transitive**: Limited to the two domains that established it.

## Replication
it is the process by which the changes made to one domain controller (DC) are **synchronized** with other DCs in the network. This ensures that all DCs have consistent and up-to-date information about the directory data, such as user accounts, passwords, and group memberships. (KCC, Connection Objects, Sites and Site Links,Global Catalog Server, Universal Group Membership Caching )
