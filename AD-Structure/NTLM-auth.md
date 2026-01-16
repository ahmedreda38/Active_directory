# LM Hashes
- LM stands for LAN MANAGER, which is a legacy password storage mechanism, first used by microsoft OS in 1987, And it has been turned off by default starting from Windows Vista 2008.
- password can not exceed 14 chars
- password is uppercased before encryption (Means we have less char pool for brute forcing!)
- char keyspace limited to a total of 69 characters Only - easier to crack
- Stored in Local `SAM database`, And in `NTDS.dit`
### How it works
1. Password is split into two halfs `7-bytes` & `7-bytes` And padded with null if Needed
2. A Null bit is inserted at the end of each 7-bits to make it a 64 bits key for DES encryption
3. Now that we have 2 DES keys we encrypt this Constant 8-bytes string `KGS!@#$%` with each DES key
4. lastly we have two (encrypted 8-bytes for `KGS!@#$%`)
5. the two 8-bytes are concatenated to form a 16 bytes LM HASH
#### Cryptographic Notice
- If the password is <= 7-bytes , this means the second splitted half with give us the exact same encrypted 8 bytes for `KGS!@#$%`
---
# NT HASH
- widows NT Hash is simply the `MD4(UTF-16-Little-ENDIAN(password))` and it is the second hals of the full NTLM hash
- `Minya:500:aad3c435b514a4eeaad3b935b51304fe:e46b9e548fa0d122de7f59fb6d48eaa2:::` -> `e46b9e548fa0d122de7f59fb6d48eaa2` NT HASH
- NT hashes are typically found in:
    1. SAM Database (C:\Windows\System32\config\SAM)
    2. NTDS.dit (Active Directory environments)
    3. Memory (using tools like Mimikatz)
    4. LSASS process (in memory during login sessions)
- NT Hashes are not salted, Two users with same password = Same NT Hash
- Can be used for `Pass-The-Hash`, we can authenticate with it in a system without knowing the plaintext password


# NTLMv1 (Net-NTLMv1)
- NTLMv1 is a challenge–response authentication protocol, not a hash stored at rest.
- It is used when a client authenticates to a remote service (SMB, HTTP, LDAP, etc.).
- Deprecated and insecure, but may still be encountered in:
    1. Legacy systems
    2. Misconfigured environments
    3. Downgrade attacks
### How NTLMv1 works (High-level)
1. Server sends an 8-byte random challenge to the client.
2. Client takes the NT Hash of the password.
3. NT Hash is padded to 21 bytes and split into three 7-byte chunks.
4. Each chunk becomes a DES key.
5. Each DES key encrypts the same server challenge.
6. The 3 encrypted results (8 bytes each) are concatenated → 24-byte NTLMv1 response.
### Key Security Weaknesses
1. Uses DES → cryptographically weak.
2. Vulnerable to offline cracking.
3. If LM Hash is enabled, NTLMv1 may expose LM responses → very fast cracking.
4. Susceptible to NTLM downgrade attacks.

## Captured via:
1. Responder
2. NTLM relay tools
3. SMB/HTTP interception

# NTLMv2 (Net-NTLMv2)

* NTLMv2 is the **modern and default** NTLM challenge-response protocol.
* Much stronger than NTLMv1, but still vulnerable to **relay attacks**.
* Commonly captured during:

  * SMB authentication
  * HTTP authentication
  * LDAP binds

### How NTLMv2 works (Simplified)

1. Server sends an **8-byte challenge**.
2. Client generates a **client challenge** and metadata (timestamp, domain, etc.).
3. Client computes:

   ```
   NTLMv2 Hash = HMAC-MD5(NT Hash, Username + Domain)
   ```
4. Response is:

   ```
   HMAC-MD5(NTLMv2 Hash, ServerChallenge + Blob)
   ```
5. Final output includes:

   * Username
   * Domain
   * Server challenge
   * Client challenge
   * NTLMv2 response

### Security Characteristics

* Uses **HMAC-MD5** instead of DES.
* Includes:

  * Timestamp
  * Client nonce
  * Target information
* **Resistant to replay**
* **Offline cracking is expensive** but still possible with weak passwords.

* Frequently captured using:
  * Responder
  * Inveigh
  * MITM6
* Cannot be used for **Pass-The-Hash**.
* Can be used for:
  * **Offline password cracking**
  * **NTLM relay attacks** (if SMB signing is disabled)

* NTLMv2 ≠ stored hash
* It is a **network authentication response**, not reusable directly.

# Domain Cached Credentials (MSCache2)
- DCC holds the last 10 Hashes Locally to be able to communicate during any potential Communication problem with the DC, and these are bit hard to crack
- they are saved into `HKEY_LOCAL_MACHINE\SECURITY\Cache`
* Extractable with:

  * Mimikatz
  * Secretsdump

### How MSCache2 Works

```
DCC2 = PBKDF2-HMAC-SHA1(
    NT Hash,
    Username (lowercase),
    Iterations = 10240
)
```

---

## Quick Comparison 

| Type     | Stored / Network | Reusable | Crackable | Common Attack |
| -------- | ---------------- | -------- | --------- | ------------- |
| NT Hash  | Stored           | Yes      | Yes       | Pass-The-Hash |
| NTLMv1   | Network          | No       | Very easy | Offline crack |
| NTLMv2   | Network          | No       | Harder    | Relay / Crack |
| MSCache2 | Stored           | No       | Moderate  | Offline crack |

---
