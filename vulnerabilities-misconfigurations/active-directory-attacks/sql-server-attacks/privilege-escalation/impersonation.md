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

* mssql-cli

### Executing the attack



## Defending

### Recommendations



### Detection



## References

{% embed url="https://www.netspi.com/blog/technical/network-penetration-testing/hacking-sql-server-stored-procedures-part-2-user-impersonation" %}
