# GenericAll

## Configuring

### Prerequisite&#x20;



### Configuring



## Attacking

### How it works

GenericAll ACL is basicly all permissions. There are different ways of attacking different objects which are:

#### Groups

With these permission on a group you can add anyone to the group.

{% content-ref url="add-user-to-group.md" %}
[add-user-to-group.md](add-user-to-group.md)
{% endcontent-ref %}

#### Users

With these permission on a user you can do two things:

* Targeted kerberoast (set spn to user and kerberoast it)
* Change their password (This will deny access to the user and may raise flags)

{% content-ref url="targeted-kerberoast.md" %}
[targeted-kerberoast.md](targeted-kerberoast.md)
{% endcontent-ref %}

{% content-ref url="forcechangepassword.md" %}
[forcechangepassword.md](forcechangepassword.md)
{% endcontent-ref %}

#### Computers

With these permissions on a computerobject you can do two things:

* Read its LAPS password.
* Get access to the machine using Resource Based Constrained Delegation

{% content-ref url="../laps.md" %}
[laps.md](../laps.md)
{% endcontent-ref %}

{% content-ref url="../delegation-attacks/resource-based-constrained-delegation/page-4.md" %}
[page-4.md](../delegation-attacks/resource-based-constrained-delegation/page-4.md)
{% endcontent-ref %}

## References

{% embed url="https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#genericall" %}
