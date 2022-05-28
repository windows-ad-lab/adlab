# AS-REP Roasting

## Configuring

### Prerequisite&#x20;

{% content-ref url="../../vulnerabilities-misconfigurations/active-directory-attacks/password-spraying.md" %}
[password-spraying.md](../../vulnerabilities-misconfigurations/active-directory-attacks/password-spraying.md)
{% endcontent-ref %}

### Configuring

1. Login on `DC02` with the username `Administrator` and password `Welcome01!`.
2. Open the "Active Directory Users and Computers" administration tool on `DC02`.

![](<../../.gitbook/assets/image (34) (1).png>)

3\. Click on "View" and enable "Advanced Features".

![](<../../.gitbook/assets/image (13).png>)

4\. Open "Users", right click the user `banktest` and click on "Properties"

5\. Open "Account" and scroll to the bottom in "Account options", then select "Do not require kerberos preauthentication".

![](<../../.gitbook/assets/image (45) (1).png>)

6\. Click on "Apply" and "OK".

## Attacking

### How it works

When pre-authentication is not required, an attack can directly send a request for authentication. The KDC will return an encrypted TGT and the attacker can extract the hash and brute-force it offline.

### Tools

* [Crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec)

### Executing the attack

AS-REP roasting was already covered in the Initial Access Attacks section.&#x20;

{% content-ref url="../initial-access-attacks/username-enumeration/as-rep-roasting.md" %}
[as-rep-roasting.md](../initial-access-attacks/username-enumeration/as-rep-roasting.md)
{% endcontent-ref %}

But since we have a set of valid credentials of the domain now, we could request a list of all usernames and check for AS-REP roastable users. During the Initial Access AS-REP Roasting we used a example script from Impacket. But there are more tools that can accomplish the same thing, such as Crackmapexec.

1. Use the discovered credentials `john` and password `Welcome2022!` with crackmapexec to authenticate over ldap and AS-REP roast all roastable users.

```
crackmapexec ldap 10.0.0.3 -u john -p Welcome2022! --asreproast asreproast.txt
```

![](<../../.gitbook/assets/image (60).png>)

2\. We retrieved two hashes, one new one from `bankuser`. Lets crack it with hashcat and use the passwordlist we created earlier during the passwordspray. The hashcat parameters are:

* Crackingmode: `-a 0` for using a wordlist
* Hashmode: `-m 18200` for Kerberos 5, etype 23, AS-REP
* List with hashes: `asreproasting.txt`
* Passwords list: `passwords.txt`

```
hashcat -a 0 -m 18200 asreproast.txt passwords.txt
```

![](<../../.gitbook/assets/image (55) (1).png>)

We successfully cracked the password of the user `bankuser`, the password is `Bank2022!`.

## Defending

### Recommendations

* Periodically check for users that don't require pre-authentication and remove the attribute.

Check for users with the attribute:

```
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} | Select-Object samAccountName
```

Remove the attribute for a single user:

```
Set-ADAccountControl -DoesNotRequirePreAuth $false -Identity <USER>
```

Check for users with the attribute and remove it:

```
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} | Set-ADAccountControl -DoesNotRequirePreAuth $false
```

### Detection



## References

{% embed url="https://github.com/byt3bl33d3r/CrackMapExec" %}

