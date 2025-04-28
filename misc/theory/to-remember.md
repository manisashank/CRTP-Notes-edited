# To Remember

* "Enterprise Admins" Group will only be present on the forest root. So to query Enterprise Admins group use `Get-DomainGroup -Name "Enterprise Admins" -Domain <forest_root_domain>`
* `Invoke-CheckLocalAdminAccess` will leave **4624 and 4634** security event ID's on the system and will leave 4672 on successful admin login. If we run this on all the local systems it would be too noisy and the SIEM's can detect this. - So execute this only with a limited count and then complete the full count. (maybe use 100 machines at a time in a total of 1000 machines). Also use jitter.
* `whoami` commands are prone to detection
* Adding our compromised user into "Domain Admins" group will raise alarms, so instead we can add an ACL giving our compromised user all the access on "Domain Admins" group. So then we'll have full control over Domain Admins group even though if we aren't a member of the group. But the problem here is there is a sanitization process i.e, "Security Description Propagator or SD Prop" which means every 1 hr (Frequency can be customized) all the Protected groups ACL's are overwritten with "AdminSDHolder" ACL's.
* Adding users/editing the local admins group on a system is very noisy.
* When a domain/forest has external trust then we can perform all the authenticated enums/actions like an authenticated user in the external forest/domain does.
* The External Trust isn't transitive. So if there is a `Domain A` in `Forest A` and `Domain B, C in Forest B` with a `two way trust b/w Domain B,C` and there is an `external trust b/w Domain A of Forest A and Domain B of Forest B` then `Domain A of Forest A doesn't have access to Domain C of Forest B`&#x20;
  *

      <figure><img src="https://remnote-user-data.s3.amazonaws.com/AF6aH6CtZUHWQFcXgECNptKjA94Xhc7-wNQKPyGtqLOTQe8I7kTdPDJnln36HIbTVXfohIm58vkIfUN3zIMEbxsf4gTEb56KIu0B_tbhiqjxqxI2MjaT85mDNUVL051O.png" alt=""><figcaption></figcaption></figure>
* While running Bloodhound Ingestors if we specify collection method as all then it enumerates/lists the sessions on the DC which the MDI triggers as malicious and triggers alarms, so it is recommended to specify `-ExcludeDC` flag to reduce the chances of triggering alarms ‒ **Since the sessions on DC aren't enumerated it can lead to losing few results on "Shortest Path to DC" in Bloodhound**
* **Extracting creds from LSASS should be the last vector to try cause it is more prone to detection**
* While using pass the hash or any attacks always use `aes256` and `not rc4 (if possible)`. Cause the encryption downgrade from `aes256 to rc4 will be flagged by the MDI`
