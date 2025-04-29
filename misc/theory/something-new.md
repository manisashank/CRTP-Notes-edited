# Something New

* Powershell is NOT powershell.exe, It is the System.Management.Automation.dll
* Powershell Execution policy is not a security layer. It can be easily bypassed with the BYPASS parameter. Microsoft itself told officially that the execution policy is set so that the users don't "ACCIDENTALLY" run the scripts.
* Does enumerating the environment create any alerts? Mostly no, but in some cases recently the MDI (Microsoft Defender) is creating alerts if more information is reconned about the pre-authenticated accounts or accounts that don't require any password
* As a remediation if you change your default "Administrator" account name to something else it won't be enough as attackers/we can still identify it from the SID, because the default administrator's SID ends with 500, i.e, the Relative identifier (RID)
* Once you have the DC access, you can add an entry in "Domain Admins" group ACL with our compromised user and full access, so this wont' mention our username in the members section but also provides us with all the Domain Admins privs. NOTE: But this isn't a valid persistence mechanism because the "AdminSDHolder" container has an ACL which tells it to sanitize/override any changes that are made to protected groups or their members. So add an full access ACL for the compromised user in the AdminSDHolder itself which will propagate the changes in a while.&#x20;
* set-mppref for disabling Windows defender creates a lot of noise.
* SID Filtering:
  * Once we get Domain Admin on one forest, with the external forest trust i.e, if Domain A of Forest A has a trust relation ship with Domain B of Forest B, then even with SID Injection on Domain A (with SID of Domain B) resources of Domain B in Forest B can't be accessed. This is because of SID Filtering.
  * That is for high priv SID's i.e, from 500 to 1000 are filtered when injected and passed onto the external forest DC.&#x20;

{% hint style="warning" %}
NOTE: By default the resources won't be accessible with injection, but if Domain B from Forest B has shared any resources explicitly with Domain A of Forest A, those can be accessed (with the SID Injection)
{% endhint %}
