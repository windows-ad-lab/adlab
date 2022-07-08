# GenericAll

## Attacking

### How it works

GenericAll ACL is basicly all permissions. There are different ways of attacking different objects which are:

#### Groups

With these permission on a group you can add anyone to the group.

{% content-ref url="add-user-to-group.md" %}
[add-user-to-group.md](add-user-to-group.md)
{% endcontent-ref %}

#### Users

With GenericAll permission on a user you can do two things:

* Targeted kerberoast (set spn to user and kerberoast it)
* Change their password (This will deny access to the user and may raise red flags)

{% content-ref url="targeted-kerberoast.md" %}
[targeted-kerberoast.md](targeted-kerberoast.md)
{% endcontent-ref %}

{% content-ref url="forcechangepassword.md" %}
[forcechangepassword.md](forcechangepassword.md)
{% endcontent-ref %}

#### Computers

With GenericAll permissions on a computerobject you can do two things:

* Read its LAPS password.
* Get access to the machine using Resource Based Constrained Delegation

{% content-ref url="../laps.md" %}
[laps.md](../laps.md)
{% endcontent-ref %}

{% content-ref url="../delegation-attacks/resource-based-constrained-delegation/computeraccount-takeover.md" %}
[computeraccount-takeover.md](../delegation-attacks/resource-based-constrained-delegation/computeraccount-takeover.md)
{% endcontent-ref %}

#### Domain object

In these GenericAll permissions the permissions DS-Replication-Get-Changes and Replication-Get-Changes-All rights are included. Giving you the ability to execute a DCSync attack.

{% content-ref url="get-changes.md" %}
[get-changes.md](get-changes.md)
{% endcontent-ref %}

## References

{% embed url="https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#genericall" %}
