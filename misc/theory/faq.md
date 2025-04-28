# FAQ

### Pass the Hash vs Over-Pass the Hash

| Feature               | Pass-the-Hash           | Over-Pass-the-Hash                         |
| --------------------- | ----------------------- | ------------------------------------------ |
| Auth protocol used    | NTLM                    | Kerberos                                   |
| Authentication method | Uses NTLM hash directly | Uses NTLM hash to get a TGT                |
| Primary goal          | Lateral movement        | Domain escalation or high-level access     |
| Tool commonly used    | PsExec, WMI, etc.       | Mimikatz                                   |
| Account type targeted | Any AD user             | Typically domain accounts with high rights |

1. Pass the Hash
   1. An attacker captures the NTLM hash of a user's password and then uses it to authenticate to systems **without knowing the actual password**.
   2. Using tools like Mimikatz or PsExec to authenticate as a user on another machine using the hash.
2. Over-Pass the hash
   1. Instead of using the hash directly in NTLM, the attacker uses the NTLM hash to **request a Kerberos TGT** from the Key Distribution Center (KDC).

***

### What is Logon type 9 (runas /netonly)

* This tells Windows to **use the given credentials for network access only**, while the local user context remains the same.
* Logon Type 9 = NewCredentials
* This logon type appears in Windows Event Logs (especially Event ID 4624) and represents a **logon using explicit credentials**, often done programmatically without impersonating the logged-on user.
