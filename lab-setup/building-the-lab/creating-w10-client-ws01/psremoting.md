# PSRemoting

PSRemoting allows you to run commands on remote computers just as if you were sitting in front of them. You could see it as the Windows SSH service.

## Enabling PSRemoting

1. Login to `WS01` as the `Administrator` user with password `Welcome01!`.
2. Start PowerShell as administrator and run the following command:

```
Enable-PSRemoting
```

![](<../../../.gitbook/assets/image (10) (1) (2).png>)

> The `Enable-PSRemoting` cmdlet performs the following operations:
>
> * Runs the [Set-WSManQuickConfig](https://docs.microsoft.com/en-us/powershell/module/microsoft.wsman.management/set-wsmanquickconfig?view=powershell-7.2) cmdlet, which performs the following tasks:
>   * Starts the WinRM service.
>   * Sets the startup type on the WinRM service to Automatic.
>   * Creates a listener to accept requests on any IP address.
>   * Enables a firewall exception for WS-Management communications.
>   * Creates the simple and long name session endpoint configurations if needed.
>   * Enables all session configurations.
>   * Changes the security descriptor of all session configurations to allow remote access.
> * Restarts the WinRM service to make the preceding changes effective.\
>   Source: [https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enable-psremoting?view=powershell-7.2](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enable-psremoting?view=powershell-7.2)

Users of the local `Administrators` or `Remote Management Users` groups can connect to the machine.

### Giving a normal user access to the service

Local admin acces is not required, it is possible as a normal user if its part of the `Remote Management Group`.

1. Add `John` to the `Remote Management Users` on `WS01` by executing the following command:

```
net localgroup "Remote Management Users" john /add
```

![](<../../../.gitbook/assets/image (8) (1) (1).png>)

## Testing

1. Login to `DC01` as the `Administrator` user with password `Welcome01!`
2. Start PowerShell and run the following command to connect to `WS01` as `Administrator`:

```
Enter-PSSession ws01
```

![](<../../../.gitbook/assets/image (39) (1) (1) (1) (1).png>)

3\. Create a PSCredential for the user `John` with the password `Welcome2022!` using the `Get-Credential` command.

```
$creds = Get-Credential
```

![](<../../../.gitbook/assets/image (23) (1) (1).png>)

4\. Run the following command to connect to `WS01` as `John`:

```
Enter-PSSession WS01 -Credential $creds
```

![](<../../../.gitbook/assets/image (11) (1) (2).png>)

Read more about PSRemoting and lateral movement:

{% content-ref url="../../../vulnerabilities-misconfigurations/misc/lateral-movement/psremoting.md" %}
[psremoting.md](../../../vulnerabilities-misconfigurations/misc/lateral-movement/psremoting.md)
{% endcontent-ref %}
