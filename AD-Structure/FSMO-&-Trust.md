## FSMO Roles - (Flexible Single Master operation) roles
- We have 5 main FSMO roles inside an activ directory environment
  1. **`Schema Master` - Forest Level Role** : Manages the Read/Write copy of the active directory scheme, which defines the attributes of all objects inside an AD
  2. **`Domain Naming Master` - Forest Level Role** : Manage domain names and ensures no duplicate names inside the same forest
  3. **`Relative ID (RID) Master`- Domain Level Role**:  Gives RID blocks to each DC in a Domain, and it is reached out when the domains run out of their RID pool
  4. **`Primary Domain Controller (PDC) emulator` - Domain Level Role** : this Role is Given for a DC in a domain which will be later responsible for auth requests, password changes and Group Policy Objects Management within a domain
  5. **`Infrastructure Master` - Domain Level Role**: 

## Trust in AD environment
- Mainly there are 5 Types of Trust
1. `Parent-child`
     - Domains within the same forest. The child domain has a two-way transitive trust with the parent domain.
2. `Cross-link`
     - a trust between child domains to speed up authentication.
3. `External`
     - A non-transitive trust between two separate domains in separate forests which are not already joined by a forest trust. This type of trust utilizes SID filtering.
4. `Tree-root`
     - a two-way transitive trust between a forest root domain and a new tree root domain. They are created by design when you set up a new tree root domain within a forest.
5. `Forest`
     -  a transitive trust between two forest root domains.
  
### Differnce between Transitive and non-Transitive
- Transitive:
    - Trust is inherited by the child domains and the trust is spread across child domains
      ```
      [ Domain A ] ───trust───> [ Domain B ] ───trust───> [ Domain C ]

      Result:
      ✔ Domain A trusts Domain C
      ✔ Credentials / Kerberos tickets may flow across
      ✔ Cross-domain attack paths exist
      ```
- Non-Transitive:
    - Trust is only in the scope of the two domains involved in the trust relationship
      ```
      [ Domain A ] ───trust───> [ Domain B ] ───trust───> [ Domain C ]
      
      Result:
      ✘ Domain A does NOT trust Domain C
      ✘ No implicit authentication path
      ✘ Trust boundary enforced
      ```
