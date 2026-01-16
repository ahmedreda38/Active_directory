# Groups in AD
- Groups are used for mass assignment of permissions and right to access services and resources
- there are Two classifications for Groups
  - type: `security` || `distribution`
  - scope: `Local domain` || `Global` || `Unviversal`

### Type
1. `Security`: type groups are the ones that are considering permissions and rights
2. `Distribution`: type groups are considering some mailing stuff such as the Defauls recepients in  `TO: `
### Scope
1. `Local Domain`: These Groups **can not** be used in other domains, but **CAN** contain users from other domains,they can have nested local groups 
2. `Global`: **Can be used** to obtain access to resources from other domain, but **can ONLY** contain users from the domain it was created in
3. `Universal`: The universal group scope can be used to manage resources distributed across multiple domains and can be given permissions to any object within the same **forest**

#### Inspect Groups and their Scope
```powershell
Get-ADGroup  -Filter * |select samaccountname,groupscope
```

### Nested Group membership and pemission inheritance 
- When user `obad` is a member of Group `devteam` and this Group is member of another Parent Group that has Higher privilege, Now `obad` easily inherits the Permissions of the Parent Group.


## Group Attributes
* member
* memberOf
* ObjectGUID
* ObjectSID
* CN
* sAMAccountName
* GroupType
  - for more info about the attributes [Microsoft's Documentation](https://learn.microsoft.com/en-us/windows/win32/ad/group-objects)