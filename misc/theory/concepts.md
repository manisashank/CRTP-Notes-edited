# Concepts

## [PowerShell Remoting](concepts.md#powershell-remoting)

* PSRemoting is like psexec but on steroids but more silent and super fast
* Uses WinRM (Windows Remote Management) which listens by default on 5985 (HTTP) and 5986 (HTTPS). WinRM is Microsoft's implementation of WS-Management
* **Enabled by default on Server 2012 onwards with a firewall exception.**
* **It is recommended way to manage windows Core servers.**
* You may need to enable remoting (`Enable-PSRemoting`) on a Desktop Windows machine, **Admin privs are required to do that.**
* **The remoting process runs as a high integrity process. That is, you get an elevated shell.**
* When a One-to-One PSSession is created:
  * It is an Interactive session
  * When the shell is spawned, it is `wsmprovhost` is executed and not powershell.exe
    * We can confirm this by executing `Get-PSHostProcessInfo`



## [Kerberos](concepts.md#kerberos)

* Clients (Programs on behalf of the user) need to obtain tickets from Key Distribution Centre (KDC) which is a service running on the domain controller. These tickets represent the client's credentials.
*

    <figure><img src="https://remnote-user-data.s3.amazonaws.com/7N1zarZJkxbXxyYguVoO9QNw4Ee81e3adpAVNplxAJ3GjkHgNYOcnxRzchQ3T8XMWiVL4ugANK8dnEIRmNJq2ooTkujRMnlqzGMF8udUAurHVnEGq5tCgMoYti6v_yVb.png" alt=""><figcaption></figcaption></figure>
* Step 1: The Client on behalf of the user sends an authentication request (AS-REQ) to the KDC, This is an timestamp encrypted with the user's NTLM/AES hash. Then the DC/KDC checks for clock-skew (default is 5mins skew is allowed) and other details, if everything looks good.
* Step 2: The KDC then responds (AS-REP) to this with a TGT (Ticket Granting Ticket), that is encrypted/signed with krbtgt secrets (NTLM/AES hash).
* Step 3: The client then presents this TGT to the DC and requests (TGS-REQ) for a TGS (Ticket Granting Service)
* Step 4: The DC responds with the TGS (TGS-REP)
* Step 5: The user then presents this TGS to the Service (Application Server in this case) and gets access to it.
