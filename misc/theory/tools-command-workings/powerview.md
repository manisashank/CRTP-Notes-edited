---
description: What happens when a particular command is run
noIndex: true
---

# PowerView



## [Find-LocalAdminAccess](powerview.md#find-localadminaccess)

Find all machines on the current domain where the current user has local admin access - Uses SCM

This function queries the DC of the current or provided domain for a list of computers (`Get-NetComputer`) and then user multi-threaded `Invoke-CheckLocalAdminAccess` on each machine.

{% hint style="warning" %}
NOTE: This method is very noisy so can be detected by a SIEM.
{% endhint %}

Can also use `Find-WMILocalAdminAccess.ps1` - Uses WMI and `Find-PSRemotingLocalAdminAccess.ps1` - Uses WinRM

<figure><img src="https://remnote-user-data.s3.amazonaws.com/CBQTsybW_Qy-cIyFJBVxw5h2Crlehzdh4VlSVslT2WNqK89raVVhNQv9y3MPa09Uu9OWSOg3xTAsnua0EfYjTMNodlxJ3mPzL2Ai9BKes-l9N_rwMDdosEgRnw95cVDq.png" alt=""><figcaption></figcaption></figure>



## [Find-DomainUserLocation](powerview.md#find-domainuserlocation)

Find computers where a domain admin (or specified user/group) has sessions

```powershell
Find-DomainUserLocation -Verbose
Find-DomainUserLocation -UserGroupIdentity "RDPUsers"
```

* This function queries the DC of the current or provided domain for members of the given group (Domain Admins by default) using `Get-DomainGroupMember`, gets a list of computers `Get-DomainComputer` and list sessions and logged on users `Get-NetSession/Get-NetLoggedon` from each machine.
* The idea here is if we are able to compromise any computer that has Domain Admin user's session then we can extract their creds.

{% hint style="danger" %}
**NOTE**: Upto server 2016 any domain user can list sessions on remote machines and we need admin access on remote machine to list logged-on users but after server 2016 i.e, 2019 and 2022 we can't list sessions on remote machines by default and we need admin access on the remote for this to be successful.
{% endhint %}

* ![](https://remnote-user-data.s3.amazonaws.com/1NsAy3qoUXpuc1mOhA8bxGSGJ-57cw2fw1c8a_H_FThaTuHSotjbldoFqmMNKAEEbIPjdOeeXa8RPWC3IkcGULF8Y-dwOTynYi7B2029YMvkJCWezPz_mf1sZmR8et2V.png)

