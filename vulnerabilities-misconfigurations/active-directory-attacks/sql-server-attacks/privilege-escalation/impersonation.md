---
description: >-
  SQL Server has a special permission, named impersonate, this enables one user
  to operate with the permissions of another user as well as their own
  permissions.
---

# Impersonation

## Configuring

#### Prerequisite

{% content-ref url="../initial-access/normal-domain-user-access.md" %}
[normal-domain-user-access.md](../initial-access/normal-domain-user-access.md)
{% endcontent-ref %}

#### Configuring

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Open "Microsoft SQL Server Management Studio"

![](<../../../../.gitbook/assets/image (34).png>)

3\. Login with the `Administrator` user using Windows Authentication.

![](<../../../../.gitbook/assets/image (7).png>)

4\. Click “New Query” button and use the SQL query below to create two new users:

![](<../../../../.gitbook/assets/image (29).png>)

```
CREATE LOGIN Developer WITH PASSWORD = 'MyPassword!';
CREATE LOGIN DB_Owner WITH PASSWORD = 'MyPassword!';
```

![](<../../../../.gitbook/assets/image (6).png>)

5\. Rune the following Query to allow impersonation:

```
USE Production;
GRANT IMPERSONATE ON LOGIN::Developer to [AMSTERDAM\Richard];
GRANT IMPERSONATE ON LOGIN::DB_Owner to [Developer];
```

![](<../../../../.gitbook/assets/image (19).png>)

## Attacking

### How it works

SQL Server has a special permission, named impersonate, this enables one user to operate with the permissions of another user as well as their own permissions. For example: user A can impersonate user B which can impersonate user C which can impersonate sa. This can be used to escalate privileges.

### Tools

* [HeidiSQL](https://www.heidisql.com)

### Executing the attack

1. Login to `WS01` as Richard with the password `Sample123`.
2. Download and start heidiSQL.
3. Click on "New" on the left bottom and configure the following settings:

* Network Type: `Microsoft SQL Server (TCP/IP)`
* Library: `SQLOLEDB`
* Hostname / IP: `WEB01.amsterdam.bank.local`
* Select: "Use Windows Authentication"
* Port: `1433`

![](<../../../../.gitbook/assets/image (33).png>)

4\. Click "OK" on the security Issue warning.

5\. Click on the "Query" tab and enter the following Query to check which users can be impersonated by the current user:

```
-- Find users that can be impersonated
SELECT distinct b.name
FROM sys.server_permissions a
INNER JOIN sys.server_principals b
ON a.grantor_principal_id = b.principal_id
WHERE a.permission_name = 'IMPERSONATE'
```

We can impersonate the Developer user.

![](<../../../../.gitbook/assets/image (62).png>)

6\. Impersonate the Developer user with the following Query:

```
EXECUTE AS LOGIN = 'developer'
```

![](<../../../../.gitbook/assets/image (42).png>)

{% hint style="info" %}
Make sure the Master database is selected since the developer user doesn't have access to the production database.
{% endhint %}

7\. Execute the who can be impersonated query again:

![](<../../../../.gitbook/assets/image (56).png>)

8\. Impersonate DB\_Owner:

```
EXECUTE AS LOGIN = 'DB_Owner'
```

We successfully impersonated Developer and then DB\_Owner.

## Defending

### Recommendations

* Minimalise the use of impersonation and audit the users who can impersonate other users.

### Detection



## References

{% embed url="https://www.netspi.com/blog/technical/network-penetration-testing/hacking-sql-server-stored-procedures-part-2-user-impersonation" %}
