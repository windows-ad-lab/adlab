# Normal domain user access

## Configuring

### Prerequisite&#x20;

{% content-ref url="../../../initial-access-attacks/username-enumeration/" %}
[username-enumeration](../../../initial-access-attacks/username-enumeration/)
{% endcontent-ref %}

### Configuring

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Open "Microsoft SQL Server Management Studio"

![](<../../../../.gitbook/assets/image (65) (1).png>)

3\. Login with the `Administrator` user using Windows Authentication.

![](<../../../../.gitbook/assets/image (63).png>)

4\. Expand the "Security" and "Logins" folders. Right click on "Logins" and click "New Login".

5\. Click on "Search", click "Locations" and expand the directories and click on "Amsterdam.bank.local".

![](<../../../../.gitbook/assets/image (60).png>)

6\. Fill in "Richard" and click "Check Names".

![](<../../../../.gitbook/assets/image (64) (1) (1).png>)

7\. At "Default Database" select "Production".

![](<../../../../.gitbook/assets/image (47) (1).png>)

8\. Click on "User Mapping" and select "Production".

![](<../../../../.gitbook/assets/image (13).png>)

9\. Click "OK".

![](<../../../../.gitbook/assets/image (32) (1).png>)

## Attacking

### How it works

Check for MSSQL servers inside the domain and try to login using the credentials from the current user or from another user. Sometimes all Domain Users have access to the database and its even possible that everyone is sysadmin on the database.

### Tools

* [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL)
* [HeidiSQL](https://www.heidisql.com)

### Executing the attack

#### PowerUpSQL

1. Download PowerUpSQL on the kali machine and host it on a webserver:

```
wget https://raw.githubusercontent.com/NetSPI/PowerUpSQL/master/PowerUpSQL.ps1
python3 -m http.server 8090
```

2\. Login to `WS01` as Richard with the password `Sample123`.

3\. Start PowerShell and download and execute an amsi and PowerUpSQL in memory:

![](<../../../../.gitbook/assets/image (49).png>)

4\. Get the SQL instances from the domain:

```
Get-SQLInstanceDomain
```

![](<../../../../.gitbook/assets/image (51).png>)

The output shows one SQL Instance.

5\. Get the SQL instances from the domain and check access:

```
Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded
```

![](<../../../../.gitbook/assets/image (19) (1).png>)

The output shows that we can access the MSSQL instance `WEB01.amsterdam.bank.local`.

If ran from a user that doesn't have access (for example John), it says `not accessible`.

![](<../../../../.gitbook/assets/image (52).png>)

6\. To get mor info about the session on the SQL server run:

```
Get-SQLServerInfo -Instance WEB01.amsterdam.bank.local
```

![](<../../../../.gitbook/assets/image (67).png>)

It shows that we aren't sysadmin. Which means we can't execute commands on the database, but our user has access to the database. So we can look into the database for sensitive information or we might be able to escalate our privileges to sysadmin.

#### Connecting with HeidiSQL

1. Download [HeidiSQL](https://www.heidisql.com/download.php?download=portable-64) on `WS01`.

```
wget https://www.heidisql.com/download.php?download=portable-64
```

2\. To execute SQL queries and look into the database start heidiSQL.

3\. Click on "New" on the left bottom and configure the following settings:

* Network Type: `Microsoft SQL Server (TCP/IP)`
* Library: `SQLOLEDB`
* Hostname / IP: `WEB01.amsterdam.bank.local`
* Select: "Use Windows Authentication"
* Port: `1433`

![](<../../../../.gitbook/assets/image (47).png>)

4\. Click "OK" on the security Issue warning.

5\. Click on the databases on the left and see if we got access to any:

![](<../../../../.gitbook/assets/image (66).png>)

We are able to access the Production database, but not the Development one.

![](<../../../../.gitbook/assets/image (14).png>)

## Defending

### Recommendations

* Periodically audit who has access to which SQL servers / databases etc.

### Detection



## References

{% embed url="https://github.com/NetSPI/PowerUpSQL" %}

{% embed url="https://www.heidisql.com" %}
