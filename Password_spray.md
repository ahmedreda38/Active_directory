## Enumerating Password Policy inside the domain
- using `nxc smb`
```shell
nxc smb <domain-name> -u myuser -p hispasswd --pass-pol
```

## check if anyone using his username as his password?
```shell
nxc smb domain.name -u users.txt -p users.txt --continue-on-success # optional to grep on the success [grep '+']
```
