# Password Spraying

## Configuring

To implement the attack we need to set a weak password for one of the users we enumerated. The easiest way is by using the `net user` command in cmd. To change the password for `john` to `Welcome2022!` and `chris` to `Summer2022!` execute the following commands:

```
net user john Welcome2022! /domain
net user chris Summer2022! /domain
```

![](<../../../.gitbook/assets/image (3).png>)

## Attacking

### How it works

Password spraying is a type of brute force attack. An attacker will try a default password against a list of usernames. For example, an attacker uses the password `Welcome2022!` against many different accounts to avoid account lockout that would normally occur when brute-forcing a single account with many passwords.

**\<part about common used passwords>**

### Tools

* [Kerbrute](https://github.com/ropnop/kerbrute)
* [Spray](https://github.com/Greenwolf/Spray)
* [Crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec)
* [Domainpasswordspray](https://github.com/dafthack/DomainPasswordSpray)
* [Rubeus](https://github.com/GhostPack/Rubeus)

### Executing the attack

#### Spraying a single password

{% tabs %}
{% tab title="Kerbrute" %}
```
./kerbrute_linux_amd64 passwordspray -d amsterdam.bank.local --dc 10.0.0.3 ~/adlab/users.txt 'Welcome2022!'
```
{% endtab %}

{% tab title="Spray" %}

{% endtab %}

{% tab title="Crackmapexec" %}

{% endtab %}

{% tab title="Domainpasswordspray" %}

{% endtab %}
{% endtabs %}

#### Spraying a small list of passwords

{% tabs %}
{% tab title="Kerbrute" %}

{% endtab %}

{% tab title="Second Tab" %}

{% endtab %}
{% endtabs %}

## Defending

### Recommendations

*

### Detection



## References

{% embed url="https://github.com/ropnop/kerbrute" %}

{% embed url="https://github.com/Greenwolf/Spray" %}

{% embed url="https://github.com/byt3bl33d3r/CrackMapExec" %}

{% embed url="https://github.com/dafthack/DomainPasswordSpray" %}

