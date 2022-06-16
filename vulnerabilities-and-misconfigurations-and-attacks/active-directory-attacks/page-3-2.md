---
description: >-
  A old habit from IT people was to write down the password for shared user
  accounts in the description field, which every user with a bit of knowledge
  can read from all users!
---

# Password in description

## Configuring

1. Login to `DC02` as `Administrator` with the password `Welcome01!`.
2. Open the "Active Directory Users and Computers" administration tool on `DC02`.

![](<../../.gitbook/assets/image (29).png>)

3\. Open the "Emplyees" OU and right click on it, select "New" and then "User". Name the user `helpdesk` and set the password `SuperSecretField1!`. Make sure to deselect "User must change password at next logon" and select "Password never expires".

4\. Right click the user helpdesk and select "Properties". In the "General" tab fill in the password SuperSecretField1! in the description field. Then click on "Apply" and "OK".

![](<../../.gitbook/assets/image (58) (1).png>)

## Attacking

### How it works

Every domain user is able to retrieve the non-protected attributes of all objects. One of these attributes is the description field. IT people used to save passwords in these fields, because its easy for them to see it in the Active Directory Users and Computers tool. But as I said, every user can see these attributes. So requesting all users with a description field, might give you access to other accounts.

### Tools

* [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

### Executing the attack

1. Login with the username `John` and password `Welcome2022!` on WS01.
2. Start PowerShell and download and execute an amsi and PowerView in memory:

![](<../../.gitbook/assets/image (31) (1).png>)

3\. The following command request all domain users with a value in the description attribute and then only selects the samaccountname and description attributes:

```
Get-DomainUser | Where-Object -Property description | Select-Object samaccountname, description
```

![](<../../.gitbook/assets/image (39).png>)

The description from helpdesk looks like a password.

4\. Run PowerShell as another user (Shift rightlick) and fill in the username `helpdesk` and password `SuperSecretField1!`.

![](<../../.gitbook/assets/image (61).png>)



![](<../../.gitbook/assets/image (16).png>)

If a PowerShell session opens it worked:

![](<../../.gitbook/assets/image (5).png>)

{% hint style="info" %}
It might be interesting to also check the descriptions of groups and computers. Never found password but sometimes some usefull information about the groups and systems!
{% endhint %}

## Defending

### Recommendations

* Periodically check for passwords in the description attribute and remove any passwords found.

Check for users with the attribute:

```
Get-ADUser -Filter {description -like '*'} -Properties samaccountname, description | Select-Object samaccountname, description
```

Remove the attribute:

```
Set-ADUser <USER> -Description $null
```

### Detection



## References

{% embed url="https://hackdefense.com/publications/wachtwoorden-in-het-omschrijvingen-veld/" %}

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1" %}
