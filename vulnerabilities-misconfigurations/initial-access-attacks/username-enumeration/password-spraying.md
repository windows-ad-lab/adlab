---
description: >-
  People don't always choose strong passwords, neither do IT people for
  temporary accounts. Spraying passwords against all found user accounts is
  effective for getting access to the domain.
---

# Password Spraying

## Configuring

### Prerequisite&#x20;

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

### Configuring

To implement the attack we need to set a weak password for some of the users we created and enumerated in the previous step. The easiest way is by using the `net user` command in the terminal. To change the password for `john` to `Welcome2022!` and `chris` to `Summer2022!` execute the following commands on `DC02`:

```
net user john Welcome2022! /domain
net user chris Summer2022! /domain
```

![](<../../../.gitbook/assets/image (3) (1) (2).png>)

## Attacking

### How it works

Password spraying is a type of brute force attack. An attacker will try a default password against a list of usernames. For example, an attacker uses the password `Welcome2022!` against many different accounts to avoid account lockout that would normally occur when brute-forcing a single account with many passwords.

A old habit what a lot of companies enforce is the requirement to change passwords every 30, 60 or 90 days. Which results in people changing their password often, forgetting it and eventually choosing a easy guessable password. Another habbit is adding a increment to the same password or using months/seasons. Which means we can easily create a password list of common passwords and things people would choose using following formats:

* season+year+! (Summer2022!)
* month+year+! (March2022!)
* companyname+year+! (Amsterdambank2022!)
* city+! (Amsterdam!)
* etc.

### Tools

* [Kerbrute](https://github.com/ropnop/kerbrute)
* [Spray](https://github.com/Greenwolf/Spray)
* [Crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec)
* [Domainpasswordspray](https://github.com/dafthack/DomainPasswordSpray)
* [Rubeus](https://github.com/GhostPack/Rubeus)

### Executing the attack

#### Spraying a single password

To spray a single password from our kali machine we will use kerbrute again. But this time we will use the passwordspray functionality. We will spray the password `Welcome2022!` against all the users we enumerated earlier.

```
./kerbrute_linux_amd64 passwordspray -d amsterdam.bank.local --dc 10.0.0.3 ~/adlab/users.txt 'Welcome2022!'
```

![](<../../../.gitbook/assets/image (11) (1) (2) (1).png>)

{% hint style="warning" %}
It is recommended to not spray more then one password every 30 minutes to prevent locking out accounts. We don't have access to the domain yet and can't retrieve the current passwordpolicy.
{% endhint %}

#### Spraying a small list of passwords

To spray a list of passwords we can use Spray. First create a `passwords.txt` file and copy the following passwords:

```
Spring2022!
Winter2022!
Autumn2022!
Summer2022!
Welcome2022!
Bank2022!
AmsterdamBank2022!
```

Since we don't want to lock any accounts we will try one password every 31 minutes by using the following command:

```
bash spray.sh smb 10.0.0.3 ~/adlab/users.txt ~/adlab/passwords.txt 1 31 passwordspray.txt
```

![](<../../../.gitbook/assets/image (23) (1) (1) (1).png>)

{% hint style="info" %}
For demonstration purposed we used 1 minute.
{% endhint %}

## Defending

### Recommendations

* Implement a strong password policy:

{% content-ref url="../../../defence/hardening/strong-password-policy.md" %}
[strong-password-policy.md](../../../defence/hardening/strong-password-policy.md)
{% endcontent-ref %}

### Detection



## References

{% embed url="https://github.com/ropnop/kerbrute" %}

{% embed url="https://github.com/Greenwolf/Spray" %}

{% embed url="https://github.com/byt3bl33d3r/CrackMapExec" %}

{% embed url="https://github.com/dafthack/DomainPasswordSpray" %}

