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



### Tools



### Executing the attack



## Defending

### Recommendations

* a

### Detection



## References

