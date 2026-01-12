## FSMO Roles - (Flexible Single Master operation) roles
- We have 5 main FSMO roles inside an activ directory environment
  1.**`Schema Master` - Forest Level Role** : Manages the Read/Write copy of the active directory scheme, which defines the attributes of all objects inside an AD
  2. **`Domain Naming Master` - Forest Level Role** : Manage domain names and ensures no duplicate names inside the same forest - 
  3. **`Relative ID (RID) Master`- Domain Level Role**:  Gives RID blocks to each DC in a Domain, and it is reached out when the domains run out of their RID pool
  4. **`Primary Domain Controller (PDC) emulator` - Domain Level Role** : this Role is Given for a DC in a domain which will be later responsible for auth requests, password changes and Group Policy Objects Management within a domain
  5. **`Infrastructure Master` - Domain Level Role**:
