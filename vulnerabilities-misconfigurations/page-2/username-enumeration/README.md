---
description: >-
  It is possible to enumerate valid usernames without authentication by sending
  TGT requests with no pre-authentication.
---

# Username Enumeration

## Configuring

To implement the attack we need to create a couple users with easy guessable usernames. To do this we can choose some usernames from a populair [list of usernames](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/xato-net-10-million-usernames.txt) from the [SecLists](https://github.com/danielmiessler/SecLists) repository.

We will create the following users:

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

Previously we created users by using the GUI. Now we will create users using PowerShell. Create the file users.txt and place the usernames in there.

```
cd C:\
notepad users.txt
$password = ConvertTo-SecureString 'ReallySecurePassword123!' -AsPlainText -Force
$files = Get-Content -Path C:\users.txt
ForEach ($name in $files) {
New-ADUser -Name "$name" -GivenName "$name" -SamAccountName "$name" -UserPrincipalName $name@amsterdam.bank.local -Path "OU=Employees,DC=amsterdan,DC=bank,DC=local" -AccountPassword $password -Enabled $true
}
```

![](<../../../.gitbook/assets/image (8).png>)

Run the command below to check the users are created:

```
Get-ADUser -Filter * | Select-Object Name
```

![](<../../../.gitbook/assets/image (15).png>)

## Attacking

### How it works

> _To enumerate usernames, send TGT requests with no pre-authentication. If the KDC responds with a `PRINCIPAL UNKNOWN` error, the username does not exist. However, if the KDC prompts for pre-authentication, we know the username exists._
>
> _Source:_ [_https://github.com/ropnop/kerbrute#user-enumeration_](https://github.com/ropnop/kerbrute#user-enumeration)__

### Kerbrute

To enumarate usernames we can use the tool [Kerbrute](https://github.com/ropnop/kerbrute). To install Kerbrute on Kali download the latest [release](https://github.com/ropnop/kerbrute/releases) from GitHub and save it somewhere. I would recommend `/opt`.

![](<../../../.gitbook/assets/afbeelding (31).png>)

For the list of usernames download the [xato-net-10-million-usernames.txt](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/xato-net-10-million-usernames.txt) list from [SecLists](https://github.com/danielmiessler/SecLists).

```
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/xato-net-10-million-usernames.txt
```

After downloading the tool and the username list run Kerbrute against the domain `amsterdam.bank.local`. Pipe the command to tee to save the output to a txt file.

```
./kerbrute_linux_amd64 userenum -d amsterdam.bank.local xato-net-10-million-usernames.txt | tee username_enum.txt
```

\<screenshot>

These valid users can be used for AS-REP roasting or Password Spraying Attacks. For now save these users to `usernames.txt`.

## Defending

### Recommendations



### Detection

The attack generates the Windows event ID [4768](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4768) "A Kerberos authentication ticket (TGT) was requested" if Kerberos logging is enabled.

