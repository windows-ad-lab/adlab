# Child to parent

## Attacking

### How it works

Once you gain Domain Admin privileges within the child domain its possible to get Enterprise Admin in the parent domain by generating a golden ticket in the child domain with the SID of the Enterprise Admin group.

### Tools

* [MimiKatz](https://github.com/samratashok/nishang/blob/master/Gather/Invoke-Mimikatz.ps1)

### Executing the attack

The attack is executed from the perspective of already gaining domain admin privileges. This is done in the unconstrained delegation section:

{% content-ref url="delegation-attacks/unconstrained-delegation/" %}
[unconstrained-delegation](delegation-attacks/unconstrained-delegation/)
{% endcontent-ref %}

The attack can be performed in two ways, abusing the [trust key](https://github.com/0xJs/RedTeaming\_CheatSheet/blob/main/windows-ad/Domain-Privilege-Escalation.md#trust-key) or the [krbtgt hash](https://github.com/0xJs/RedTeaming\_CheatSheet/blob/main/windows-ad/Domain-Privilege-Escalation.md#krbtgt-hash). I prefer the krbtgt hash way so that is what we would do in the attack below.

1. For easy execution, login on `WS01` as the `Administrator` user with the password `Welcome01!`.

## Defending

### Recommendations

* a

### Detection



## References

{% embed url="https://github.com/samratashok/nishang/blob/master/Gather/Invoke-Mimikatz.ps1" %}
