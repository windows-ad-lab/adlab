---
description: >-
  By default the SA user is NOT enabled. Administrators might enable it during
  the installation and choose a weak password.
---

# SQL Server default login

## Configuring

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.

2\. Open "Microsoft SQL Server Management Studio"

![](<../../.gitbook/assets/image (32).png>)

3\. Login with the `Administrator` user using Windows Authentication.

4\. Expand the "Security" and "Logins" folders. Right click "sa" and select "Properties". Make sure "Enforce password policy" is unchecked and fill in the password `sa` twice.

![](<../../.gitbook/assets/image (5).png>)

## Attacking

### How it works

One of the default users (not enabled by default) for SQL Server is the SA user. Administrators might enable it during the installation and choose a weak password.

### Tools



### Executing the attack



## Defending

### Recommendations



### Detection



## References

