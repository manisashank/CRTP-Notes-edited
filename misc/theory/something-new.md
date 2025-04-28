# Something New

* Powershell is NOT powershell.exe, It is the System.Management.Automation.dll
* Powershell Execution policy is not a security layer. It can be easily bypassed with the BYPASS parameter. Microsoft itself told officially that the execution policy is set so that the users don't "ACCIDENTALLY" run the scripts.
* Does enumerating the environment create any alerts? Mostly no, but in some cases recently the MDI (Microsoft Defender) is creating alerts if more information is reconned about the pre-authenticated accounts or accounts that don't require any password
* As a remediation if you change your default "Administrator" account name to something else it won't be enough as attackers/we can still identify it from the SID, because the default administrator's SID ends with 500, i.e, the Relative identifier (RID)
* Once you have the DC access, you can add an entry in "Domain Admins" group ACL with our compromised user and full access, so this wont' mention our username in the members section but also provides us with all the Domain Admins privs. NOTE: But this isn't a valid persistence mechanism because the "AdminSDHolder" container has an ACL which tells it to sanitize/override any changes that are made to protected groups or their members. So add an full access ACL for the compromised user in the AdminSDHolder itself which will propagate the changes in a while.&#x20;
* set-mppref for disabling Windows defender creates a lot of noise.
