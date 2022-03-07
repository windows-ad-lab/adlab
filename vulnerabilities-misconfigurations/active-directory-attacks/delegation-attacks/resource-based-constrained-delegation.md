# Resource Based Constrained Delegation

## Configuring

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Open PowerShell and execute the following command.

```
Install-WindowsFeature WebDAV-Redirector –Restart
```

2\. After the server restarts, Open the "Windows Defender Firewall".

![](<../../../.gitbook/assets/image (38).png>)

3\. Click on "Allow an app or feature through Windows Defender Firewall"

![](<../../../.gitbook/assets/image (2).png>)

4\. Select "File and Printer sharing" and click on "OK"

![](<../../../.gitbook/assets/image (23).png>)

## Attacking

### How it works

[https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/](https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/)

### Tools

* [Powermad](https://github.com/Kevin-Robertson/Powermad)
* [Invoke-DNSUpdate](https://github.com/Kevin-Robertson/Powermad/blob/master/Invoke-DNSUpdate.ps1)
* [Change-LockScreen](https://github.com/nccgroup/Change-Lockscreen)

### Executing the attack

#### Prerequisite

* Low priv shell on a machine
* An account with a SPN associated (or able to add new machines accounts (default value this quota is 10))
* WebDAV Redirector feature must be installed on the victim machine. (W10 has it by default, but manually installed on server 2016 and later)
* A DNS record pointing to the attacker’s machine (By default authenticated users can do this)

#### Executing the attack



### Cleanup

1. Login to `DC02` as `Administrator` with the password `Welcome01!`.
2. Execute the following command to remove the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute from `WEB01`.
3. Open "DNS Manager" and expand DC02 --> Forward Lookup Zones --> and click on `Amsterdam.bank.local.`Remove the DNS-record created for webdav:

![](<../../../.gitbook/assets/image (33).png>)

## Defending

### Recommendations



### Detection



## References

{% embed url="https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation" %}

{% embed url="https://github.com/Kevin-Robertson/Powermad" %}

{% embed url="https://github.com/Kevin-Robertson/Powermad/blob/master/Invoke-DNSUpdate.ps1" %}
