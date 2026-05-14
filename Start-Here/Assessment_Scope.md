## Assessment Scope

The following `IPs`, `hosts`, and `domains` defined below make up the scope of the assessment.
#### In Scope For Assessment

| **Range/Domain**                | **Description**                                                                           |
| ------------------------------- | ----------------------------------------------------------------------------------------- |
| `INLANEFREIGHT.LOCAL`           | Customer domain to include AD and web services.                                           |
| `LOGISTICS.INLANEFREIGHT.LOCAL` | Customer subdomain                                                                        |
| `FREIGHTLOGISTICS.LOCAL`        | Subsidiary company owned by Inlanefreight. External forest trust with INLANEFREIGHT.LOCAL |
| `172.16.5.0/23`                 | In-scope internal subnet.                                                                 |

#### Out Of Scope
- `Any other subdomains of INLANEFREIGHT.LOCAL`
- `Any subdomains of FREIGHTLOGISTICS.LOCAL`
- `Any phishing or social engineering attacks`
- `Any other IPS/domains/subdomains not explicitly mentioned`
- `Any types of attacks against the real-world inlanefreight.com website outside of passive enumeration shown in this module`
### External & Internal Perimeters
- External : 
	- `https://www.inlanefreight.com`
- Internal :
	- Internal Active Directory environment
