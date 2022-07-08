---
description: >-
  It is possible that accounts have an empty password if the useraccountcontrol
  attribute contains the value PASSWD_NOT_REQ.
---

# Empty password

## Configuring

1. Login on `DC02` with the username `Administrator` and password `Welcome01!`.
2. Open the "Active Directory Users and Computers" administration tool on `DC02`

![](<../../.gitbook/assets/image (69) (1).png>)

3\. Click on "View" and enable "Advanced Features".

![](<../../.gitbook/assets/image (63).png>)

4\. Right click the "Users" section and select "New" and then "User". Create a new user named `bank_dev` with the password `Password01!`. Make sure to deselect "User must change password at next logon" and select "Password never expires".

![](<../../.gitbook/assets/image (47).png>)

4\. Right click the user and select "Properties". Open the tab "Attribute Editor", search for "Useraccountcontrol" and click "Edit".

![](<../../.gitbook/assets/image (57) (1).png>)

5\. Set the value to `544` and cick "OK".

![](<../../.gitbook/assets/image (46) (1).png>)

6\. Click "Apply" and "OK".

7\. Right click on `bank_dev` and select "Reset Password". Uncheck "User must change password at next logon" and make sure the Password fields are empty. Click on "OK"

![](<../../.gitbook/assets/image (22) (1) (1).png>)

## Attacking

### How it works

It is **possible** that accounts have an empty password if the useraccountcontrol attribute contains the value `PASSWD_NOT_REQ`. With access to a normal domain user we could request all users with this attribute set.

### Tools

* [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)

### Executing the attack

1. Use the discovered credentials `john` and password `Welcome2022!` with CrackMapExec to authenticate over ldap and request all users with the value `PASSWD_NOT_REQ` set.

```
crackmapexec ldap 10.0.0.3 -u john -p Welcome2022! --password-not-required
```

![](<../../.gitbook/assets/image (6) (1) (1).png>)

2\. We already knew the user `steve` had a empty password from our initial access attacks. The `Guest` password is empty by default, but this account is also disabled by default. We can check if `bank_dev` user has a empty password just like we did earlier.

```
crackmapexec smb 10.0.0.3 -u bank_dev -p ''
```

![](<../../.gitbook/assets/image (31) (1) (1).png>)

The password is indeed empty.

## Defending

### Recommendations

* Periodically check for users with the `PASSWD_NOT_REQ` attribute and remove it.

Check for users with the attribute:

```
Get-ADUser -Filter {PasswordNotRequired -eq $true} | Select-Object samAccountName
```

Remove the attribute:

```
Set-ADAccountControl -PasswordNotRequired $false -Identity <USER>
```

Check for users with the attribute and remove the attribute:

```
Get-ADUser -Filter {PasswordNotRequired -eq $true} | Set-ADAccountControl -PasswordNotRequired $false
```

### Detection



## References

{% embed url="https://specopssoft.com/blog/find-ad-accounts-using-password-not-required-blank-password/" %}

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1" %}

{% embed url="https://github.com/byt3bl33d3r/CrackMapExec" %}
