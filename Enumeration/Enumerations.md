# Enumerations
## Network scanning
### Host scanning
in the case when we have a subnet to work with, we can run nmap scan with that subnet to seethe live ip's, and once we got some ips, we can do more enuerations to see what services are running on each host and which one is the Domain controller.
```bash
nmap -sn 192.168.12.0/24 --exclude <our-ip>
```
```shell
rustscan -u 5000 -a 192.162.12.145 -- -Pn -sC
```
```bash
nmap -p- -T5 -vv -oA DC-Nmap -Pn -sC -sV 192.168.12.145 | tee -a DC-Nmap.txt
```
```bash
cat DC-Nmap.txt | grep Discovered | cut -d ' ' -f 4 | cut -d '/' -f 1 | sort -u | tr '\n' ',' | sed 's/,$//'  > open_ports
```
```bash
nmap -p $(cat open_ports) -sV 10.129.234.71 
```
_we can use nxc smb to get the domain name and NETBIOS name_
```shell
nxc smb 192.168.12.145 #Single target
nxc smb 192.168.12.9/24
nxc smb ips.txt
ldapsearch -H ldap://target.ip -x -b "DC=simply,DC=cyber" '(objectclass=person)'
smbclient -L \\\\target.ip\\
smbclient \\\\target.ip\\sharename

#enumrating smb and ldap for users
nxc smb 10.10.10.128 -u '' -p '' --shares
nxc smb 10.10.10.128 -u anonymous -p '' --shares
nxc ldap 10.10.10.128 -u '' -p '' --users
nxc ldap 10.10.10.128 -u anonymous -p '' --users
nxc smb 10.10.10.128 -u 'anonymous' -p '' --rid-brute | grep "SidTypeUser"
```
### All in on version to get and store the users in users.txt
```shell
nxc smb 10.10.10.128 -u 'anonymous' -p '' --rid-brute | grep "SidTypeUser" | cut -d '\' -f 2 | cut -d ' ' -f 1 > users.txt
```
- using _ldapsearch_
  ```shell
  ldapsearch -H ldap://10.10.10.128 -x -b "DC=simply,DC=cyber" '(objectclass=person)' | grep sAMAccountName | cut -d ':' -f 2 | tr -d ' '
  ```
### Enumerating Users with found credentials
- using ldapdomaindump
- ```shell
  mkdir ldapdump && cd ldapdump
  # will dump many information about the domain in amny forms
  # NOTE: if the domain_users.html file is 905 size this means it failed.
  ldapdomaindump 'ldap://target.ip:389' -u '<domain-name>\<username>' -p  'hisSup3rscurPA$$'
  ```
- using Bloodhound
then we can use this to injest in bloodhound and find the mapping and relations and etc..
```shell
bloodhound-python -d <domain-name> -u <username> -p <password> -ns <dc-ip> -c All --zip
```
## Using bloodhound
using the `bloodhound-cli` binary, we need to follow these steps
1. if now installed we build up the container
```shell
./bloodhound-cli install #[uninstall]
```
2. spawn it
```shell
./bloodhound-cli up
# to stop we use [down]
```
3. get the default password if missed
```shell
./bloodhound-cli config get default_password
```
4. reset default password
```shell
./bloodhound-cli resetpwd                   
```
5. for more debugging during the runtime 
```shell
docker logs bloodhound-bloodhound-1 # <-- container name
```
