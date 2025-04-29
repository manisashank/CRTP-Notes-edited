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

***

## What is a spooler service in windows

The **Spooler service** in Windows—officially known as the **Print Spooler**—is a system service responsible for managing all print jobs sent to a computer printer or print server. It allows Windows to queue and manage multiple print jobs efficiently.

To check if spooler service is running or not - **Locally**

{% tabs %}
{% tab title="cmd" %}
```
sc query spooler
```
{% endtab %}

{% tab title="Powershell" %}
```
Get-Service -Name spooler
```
{% endtab %}

{% tab title="GUI" %}
1. Press `Win + R`, type `services.msc`, and press **Enter**.
2. In the Services window, scroll down and find **Print Spooler**.
3. Look in the **Status** and **Startup Type** columns:
   * **Status** should be **Running** if it's active.
   * **Startup Type** tells you whether it starts automatically (`Automatic`), manually, or is disabled.
{% endtab %}
{% endtabs %}

To check if spooler service is running or not - **Remotely**

{% hint style="warning" %}
Be a **local administrator** on the remote machine, or

Have **delegated rights** (e.g., via Group Policy or RBAC).
{% endhint %}

{% tabs %}
{% tab title="Built-in Powershell" %}
{% code overflow="wrap" %}
```powershell
Get-WmiObject -Class Win32_Service -ComputerName <RemoteComputerName>
Get-CimInstance -ClassName Win32_Service -ComputerName <RemoteComputerName>

# Filter for specific service
Get-WmiObject -Class Win32_Service -ComputerName <RemoteComputerName> | Where-Object { $_.Name -eq 'Spooler' }
```
{% endcode %}
{% endtab %}

{% tab title="Built-in cmd" %}
```
sc \\<RemoteComputerName> query spooler
```
{% endtab %}
{% endtabs %}

1. Look for:
   * `STATE` → should say **RUNNING** if it's active.
   * `START_TYPE` (via `sc qc spooler`) → tells how it starts (Automatic, Manual, Disabled).

Check startup type

```
Get-WmiObject -Class Win32_Service -Filter "Name='Spooler'" | Select-Object Name, StartMode
```
