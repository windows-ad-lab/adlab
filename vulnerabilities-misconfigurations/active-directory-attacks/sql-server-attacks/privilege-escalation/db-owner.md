# DB-Owner

## Configuring

### Prerequisite

{% content-ref url="impersonation.md" %}
[impersonation.md](impersonation.md)
{% endcontent-ref %}

### Configuring

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Open "Microsoft SQL Server Management Studio"

![](<../../../../.gitbook/assets/image (3).png>)

3\. Login with the `sa` user using the password `sa` or `Password1!` (Depending if you changed it for another vulnerability)

![](<../../../../.gitbook/assets/image (30).png>)

4\. Click “New Query” button and use the SQL query below to make `DB_Owner`  database owner of the `production` database.

![](<../../../../.gitbook/assets/image (10).png>)

```
Use Production;
ALTER LOGIN [Dbo] with default_database = [Production];
CREATE USER [Dbo] FROM LOGIN [Dbo];
EXEC sp_addrolemember [db_owner], [Dbo];
```

![](<../../../../.gitbook/assets/image (18).png>)

![](<../../../../.gitbook/assets/image (32).png>)























## Attacking

### How it works



### Tools



### Executing attack



## Defending

### Recommendations



### Detection



## References
