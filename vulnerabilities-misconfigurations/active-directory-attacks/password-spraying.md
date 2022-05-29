---
description: >-
  People don't always choose strong passwords, neither do IT people for
  temporary accounts. Spraying passwords against all user accounts is effective
  for moving laterally and escalating privileges.
---

# Password spraying

## Configuring

1. To create two new users with weak passwords execute the following commands on `DC02` after logging in with the `Administrator` user.

```
net user bankuser Bank2022! /add /domain
net user banktest Bank2022! /add /domain
```

![](<../../.gitbook/assets/image (56) (1) (1).png>)

## Attacking

### Tools

* [Kerbrute](https://github.com/ropnop/kerbrute)
* [Spray](https://github.com/Greenwolf/Spray)
* [Crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec)
* [Domainpasswordspray](https://github.com/dafthack/DomainPasswordSpray)
* [Rubeus](https://github.com/GhostPack/Rubeus)

### Executing the attack

Spraying passwords was already covered in the Initial Access Attacks section.

{% content-ref url="../../vulnerabilities-and-misconfigurations-and-attacks/initial-access-attacks/username-enumeration/password-spraying.md" %}
[password-spraying.md](../../vulnerabilities-and-misconfigurations-and-attacks/initial-access-attacks/username-enumeration/password-spraying.md)
{% endcontent-ref %}

But since we have a set of valid credentials of the domain now, we could request a list of all usernames and passwordspray again. We will do just that in this section.

1. Use the discovered credentials `john` and password `Welcome2022!` with crackmapexec to authenticate over ldap and retrieve a list of all the users.

```
crackmapexec ldap 10.0.0.3 -u john -p Welcome2022! --users
```

![](<../../.gitbook/assets/image (62) (1) (1).png>)

2\. We discovered a couple extra users such as `admin_amsterdam`, `IT-support01`, `pukcab`, `IT-support01`, `bankuser` and `banktest`.

3\. We could spray passwords using the tool spray, just like before. But lets use another tool now, like Crackmapexec. We just need to give it a list of usernames and passwords and add the `--continue-on-success` parameter otherwise it stops as the first succesfull login.

```
crackmapexec smb 10.0.0.3 -u users.txt -p passwords.txt --continue-on-success
```

![](<../../.gitbook/assets/image (71) (1) (1) (1) (1).png>)

We discovered two extra set of credentials. `Bankuser` and `banktest`.

## Defending

### Recommendations

* Implement a strong password policy:

{% content-ref url="../../defence/hardening/strong-password-policy.md" %}
[strong-password-policy.md](../../defence/hardening/strong-password-policy.md)
{% endcontent-ref %}

### Detection



## References

{% embed url="https://github.com/ropnop/kerbrute" %}

{% embed url="https://github.com/Greenwolf/Spray" %}

{% embed url="https://github.com/byt3bl33d3r/CrackMapExec" %}

{% embed url="https://github.com/dafthack/DomainPasswordSpray" %}
