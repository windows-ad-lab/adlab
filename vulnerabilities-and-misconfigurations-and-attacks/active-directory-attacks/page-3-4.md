# Foreign user

## Configuring

1. Login to `DC01` with the `Administrator` user and the password `Welcome01!`.
2. Open the "Active Directory Users and Computers" administration tool.

![](<../../.gitbook/assets/image (17).png>)

3\. Open the "Users" directory and right click it, then select "New" and then "Users".

4\. Create a new user with the name `secure_admin` and the password `rFKbUJrDu$sz*36ffKr6`.

![](<../../.gitbook/assets/image (22).png>)

5\. Make sure "Password never expires" is checked and "User must change password at next logon" is unchecked when creating the user.

6\. to `DC03` with the `Administrator` user and the password `Welcome01!`.

7\. Open the "Active Directory Users and Computers" administration tool.

8\. Open the "Users" directory and right click it, then select "New" and then "Group".

9\. Make a new group with the name `Local admin data` and select group scope "Domain Local".

![](<../../.gitbook/assets/image (77).png>)

10\. Right click the group and select "Properties". Click on the "Members" tab and click on "Add"

11\. Click on "Locations" and select bank.local.

![](<../../.gitbook/assets/image (59).png>)

12\. Enter the username secure\_admin and click "Check Names" and then click "OK".

![](<../../.gitbook/assets/image (65).png>)

13\. Click "Apply" and then "OK".

14\. To make the group local admin to the DATA01 server there are two options

* Login to the machine and use the net command to add the group to the local administrators group.
* Deploy a GPO to make the group local admin on the machine.

15\. We will go with the first step since its just easier for the lab setup, but normally IT would deploy it with GPO. So login with the username Administrator and password Welcome01! to DATA01 and execute the following command:

```
net localgroup administrators "local admin data" /add
```

![](<../../.gitbook/assets/image (64).png>)

## Attacking

### How it works

If there is a trust between two forests, it is possible to add a user of domain A to a group in domain B. If for example this group gives local admin privileges to a system in forest B then after taking over that user it is possible to gain access to domain B using that account. Depending on the trust flow configured between the domains(one way or two way trust), they can both add other users or only one domain can add users.

Meaning that if you take over domain A you can easily access domain B by enumerating the groups which have a foreign user from domain A. I see this often used for having one admin account to manage both domains for example.

### Tools

* [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

### Executing the attack

This attack is executing from the perspective of already gaining Enterprise Admin privileges within the `bank.local` admin.

1. To easily execute the attack login on `DC01` with the user `Administrator` and the password `Welcome01!`.
2. Start PowerShell and load an amsi and PowerView into memory:

![](<../../.gitbook/assets/image (57).png>)

3\. With the PowerView cmdlet `Get-DomainForeigGroupMember` we can query the target domain for users in groups that aren't from their domain.

```
Get-DomainForeignGroupMember -Domain secure.local
```

![](<../../.gitbook/assets/image (44).png>)

4\. The output shows us that there is a foreign user in the `Local admin data` group and that the member is `S-1-5-21-320929719-844265543-1524670925-1602`.  To get the user name we can use the `ConvertFrom-SID` cmdlet from PowerView:

```
ConvertFrom-SID S-1-5-21-320929719-844265543-1524670925-1602
```

![](<../../.gitbook/assets/image (78).png>)

5\. The user `bank\secure_admin` from `bank.local` is member of `local admin data` in `secure.local`. We can also see this in bloodhound after collecting the data for both domains and loading it into the BloodHound GUI.

```
Invoke-BloodHound -Collectionmethod all -Domain secure.local
Invoke-BloodHound -Collectionmethod all -Domain bank.local
```

After clicking on "Users with Foreign Domain Group Membership" and selecting `Secure.local` under the "Dangerous Rights" section in the "Analysis" tab we see:

![](<../../.gitbook/assets/image (75).png>)

6\.&#x20;

























## Defending

### Recommendations

* a

### Detection



## References

