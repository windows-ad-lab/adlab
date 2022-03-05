---
description: >-
  It is possible that accounts have an empty password if the useraccountcontrol
  attribute contains the value PASSWD_NOT_REQ.
---

# Empty Password

## Configuring

1. Open the "Active Directory Users and Computers" administration tool on `DC02`.

![](<../../../.gitbook/assets/image (5).png>)

2\. Click on "View" and enable "Avanced Features"

![](<../../../.gitbook/assets/image (22).png>)

3\. Open "Employees", right click the user `steve` and click on "Properties"

![](<../../../.gitbook/assets/image (15).png>)

4\. Open "Attribute Editor", search for "Useraccountcontrol" and click "Edit".

![](<../../../.gitbook/assets/image (16).png>)

5\. Set the value to `544` and cick "OK".

![](<../../../.gitbook/assets/image (14).png>)

6\. Click "Apply" and "OK".

7\. Right click on Steve and select "Reset Password". Uncheck "User must change password at next logon" and make sure the Password fields are empty. Click on "OK"

![](<../../../.gitbook/assets/image (64).png>)

![](<../../../.gitbook/assets/image (65).png>)

## Attacking

### How it works

It is **possible** that accounts have an empty password if the useraccountcontrol attribute contains the value `PASSWD_NOT_REQ`.

### Tools

* [Crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec)

### Executing the attack

1. Check if any on the users have a empty password by spraying an empty password for all the enumerated users against the DC.

```
crackmapexec smb 10.0.0.3 -u users.txt -p '' -d amsterdam.bank.local
```

![](<../../../.gitbook/assets/image (62).png>)

{% hint style="warning" %}
Spraying an empty password counts as a invalid login. So it is adviced to not do this while also passwordspraying as it might cause account lockouts.
{% endhint %}

2\. The user `Steve` has an empty password.

## Defending

### Recommendations

* Periodically check for users with the `PASSWD_NOT_REQ` attribute and remove it.

```
Get-ADUser -Filter {PasswordNotRequired -eq $true}
```

### Detection





## References

{% embed url="https://specopssoft.com/blog/find-ad-accounts-using-password-not-required-blank-password" %}

{% embed url="https://github.com/byt3bl33d3r/CrackMapExec" %}
