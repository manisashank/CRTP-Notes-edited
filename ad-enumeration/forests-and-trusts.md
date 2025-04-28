# Forests and Trusts

Trust is a relationship between two domains or forests which allows trusted domain or forest to access resources in the other domain or forest.

Trust is automatically built or manually established.

## One-way and two-way trusts <a href="#one-way-and-two-way-trusts" id="one-way-and-two-way-trusts"></a>

### One Way

Trust relationships enable access to resources can be either one-way or two-way.\
A one-way trust is a **unidirectional** path between two domains.

For example In a one-way trust:\
&#xNAN;_&#x44;omain A_ <- _Domain B_

Users in _**Domain A**_ can access resources in _**Domain B**_. However, users in _Domain B_ can't access resources in _Domain A_.

### Two Way

In a two-way trust, _Domain A_ trusts _Domain B_ and _Domain B_ trusts _Domain A_.

Users in _**Domain A**_ can access resources in _**Domain B**_ and, users in _Domain B_ can access resources in _Domain A_.

## Transitive and non-transitive trusts <a href="#transitive-and-non-transitive-trusts" id="transitive-and-non-transitive-trusts"></a>

Transitivity determines whether a trust can be extended outside of the two domains with which it was formed.

* A transitive trust can be used to extend trust relationships with other domains.
* A non-transitive trust can be used to deny trust relationships with other domains.

## Defaults

**Parent-Child domains will be always two-way transitive.**

**Tree-Root will always be two way transitive.**

<figure><img src="../.gitbook/assets/trust-relationships.png" alt=""><figcaption><p>default trust relationship flows</p></figcaption></figure>

## External Trusts

Trust between two domains in different forests when forests do not have a trust relationship.\
Can be one-way or two-way but can't be transitive.

<img src="../.gitbook/assets/file.excalidraw (1) (1).svg" alt="" class="gitbook-drawing">

## Forest Trusts

Forest trusts are manually created between two root forests,.

{% hint style="warning" %}
**Important:** Forest trusts can only be created between two forests and can't be implicitly extended to a third forest.
{% endhint %}

<figure><img src="../.gitbook/assets/forest-trusts-diagram.png" alt=""><figcaption></figcaption></figure>

This example configuration provides the following access:

* Users in _Forest 2_ can access resources in any domain in either _Forest 1_ or _Forest 3_
* Users in _Forest 3_ can access resources in any domain in _Forest 2_
* Users in _Forest 1_ can access resources in any domain in _Forest 2_



## Enumeration

{% tabs %}
{% tab title="PowerView" %}
Get a list of all domain trusts for the current domain

```powershell
Get-DomainTrust
Get-DomainTrust -Domain us.dollarcorp.moneycorp.local

# External trusts
Get-DomainTrust | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```

{% hint style="danger" %}
<img src="https://remnote-user-data.s3.amazonaws.com/iWuiq1q1PF1eq0pWxM0EFydeJ2KMMs-kdCF1eL_x866PtWZ62c3hB_hh_vRLyvcJXw_6QWeKC6L7Hk8tUv0qswfKQ-gSwd6oDnQ2yhbdUNryQ-tChrQcNcEFLcuNlsM7.png" alt="" data-size="original">

If you get an error like this when we try to enumerate the trust relationships of any domain like in the above example, that means your current domain do not have trust relationship with the specified domain.
{% endhint %}

Get details about the current forest

{% code overflow="wrap" %}
```powershell
Get-Forest
Get-Forest -Forest eurocorp.local
```
{% endcode %}

Get all domains in the current forest

{% code overflow="wrap" %}
```powershell
Get-ForestDomain
Get-ForestDomain -Forest eurocorp.local
```
{% endcode %}

Get all global catalogs for the current forest

```powershell
Get-ForestGlobalCatalog
Get-ForestGlobalCatalog -Forest eurocorp.local
```

Map trusts of a forest

<pre class="language-powershell"><code class="lang-powershell"># External trusts in current forest
<strong>Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
</strong>
# All domain trusts for domains with specified forest
Get-ForestDomain -Forest &#x3C;external-trusting-forest> | %{Get-DomainTrust -Domain $_.Name}

Get-ForestTrust
Get-ForestTrust -Forest eurocorp.local
</code></pre>
{% endtab %}

{% tab title="AD Module" %}
Get a list of all domain trusts for the current domain

```powershell
Get-ADTrust
Get-ADTrust -Identity us.dollarcorp.moneycorp.local
```

Get details about the current forest

{% code overflow="wrap" %}
```powershell
Get-ADForest
Get-ADForest -Identity eurocorp.local
```
{% endcode %}

Get all domains in the current forest

{% code overflow="wrap" %}
```powershell
(Get-ADForest).Domains
```
{% endcode %}

Get all global catalogs for the current forest

```powershell
Get-ADForest | select -ExpandProperty GlobalCatalogs
```

Map trusts of a forest

```powershell
Get-ADTrust -Filter 'msDS-TrustForestTrustInfo -ne "$null"'
```
{% endtab %}
{% endtabs %}

## Examples

`Get-DomainTrust`

*

    <figure><img src="https://remnote-user-data.s3.amazonaws.com/QLeJHMS8pZRsCRpsYdrv_XXIH7aj__5sVHcPGmrwLF1aih3J1lCSY4ajAlOhZuQWAGvGxOEhxhkCUTjs2KKMrJu4WDodrWOwGVnhx2WhORfztxpjD61fUL9q1XqXFuW5.png" alt=""><figcaption></figcaption></figure>
* The first entry says that `dollarcorp.moneycorp.local` and `moneycorp.local` domains has `bidirectional trust within_forest`
* The third entry says that `dollarcorp.moneycorp.local` and `eurocorp.local` has an `external bidirectional trust b/w them`. **We know that this is an external trust by TrustAttributes value as FILTER\_SIDS**

## References

* [https://learn.microsoft.com/en-us/entra/identity/domain-services/concepts-forest-trust](https://learn.microsoft.com/en-us/entra/identity/domain-services/concepts-forest-trust)
