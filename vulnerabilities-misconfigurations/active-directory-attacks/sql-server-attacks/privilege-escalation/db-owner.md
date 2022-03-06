# DB-Owner

## Configuring

### Prerequisite

{% content-ref url="../initial-access/normal-domain-user-access.md" %}
[normal-domain-user-access.md](../initial-access/normal-domain-user-access.md)
{% endcontent-ref %}

### Configuring

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Open "Microsoft SQL Server Management Studio"

![](<../../../../.gitbook/assets/image (3).png>)

3\. Login with the `sa` user using the password `sa` or `Password1!` (Depending if you changed it for another vulnerability)

![](<../../../../.gitbook/assets/image (30).png>)

4\. Click “New Query” button and use the SQL query below to make `Amsterdam\Richard` database owner of the `production` database.

![](<../../../../.gitbook/assets/image (10).png>)

```
Use Production;
EXEC sp_addrolemember [db_owner], [AMSTERDAM\Richard];
```

5\. Change the Owner of the database to the SA account. Right click on "Production", click "Properties" and open the "Files" tab. Click on the "..." and fill in "sa" and click on "OK"

![](<../../../../.gitbook/assets/image (2).png>)

6\. Execute the following query to make sure `Amsterdam\Richard` is Database owner and the real Owner of the database is `sa`:

```
select rp.name as database_role, mp.name as database_user
from sys.database_role_members drm
join sys.database_principals rp on (drm.role_principal_id = rp.principal_id)
join sys.database_principals mp on (drm.member_principal_id = mp.principal_id)

SELECT suser_sname(owner_sid) FROM sys.databases WHERE name = 'Production'
```

![](<../../../../.gitbook/assets/image (68).png>)

![](<../../../../.gitbook/assets/image (53).png>)

## Attacking

### How it works



### Tools



### Executing attack



## Defending

### Recommendations



### Detection



## References
