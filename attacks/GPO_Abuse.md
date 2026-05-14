## Group Policy Object (GPO) Abuse
### What can we do with abusing GPO
1. Add malicious scheduled tasks
2. Grant our controlled users Additional rights
3. Adding a local admin user to one or more hosts

### Enumerating GPS names `PowerView`
```powershell
Get-DomainGPO |select displayname
```
### Enumerating GPO names - built-ins in Group Policy Management tools
```pwoershell
Get-GPO -All | Select DisplayName
```
### Check if any domain user can any rights over GPO
```powershell
$sid = Convert-NameToSid "Domain Users"
Get-DomainGPO | Get-ObejctAcl | ?{$_.SecurityIdentifier -eq $sid}
```
### Converting GPO GUID to Name
```powershell
Get-GPO -Guid 7CA9C789-14CE-46E3-A722-83F4097AF532
```
