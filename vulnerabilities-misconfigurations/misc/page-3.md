# Dumping DPAPI

## Configuring

1. Login to `DC03` as the `Administrator` user with the password `Welcome01!`.
2. Open the "Active Directory Users and Computers" management tool and open the "Users" directory. Right click the "Users" directory and click "New User"
3. Create a user with the name `sa_backup` and the password `LS6RV5o8T9`. Make sure you deselect "User must change password at next logon" and select "Password never expires".

![](<../../.gitbook/assets/image (67).png>)

![](<../../.gitbook/assets/image (62).png>)

4\. Add the user to the "Account operator" and "Backup Operator" groups via the interface, memberof section or run the following command:

```
Add-ADGroupMember "Account Operators" -Members sa_backup
Add-ADGroupMember "Backup Operators" -Members sa_backup
```

5\. Login with the `sa_sql` user and the password `Iloveyou2` on `DATA01`.

6\. Click start and open the "Credential Manager".

![](<../../.gitbook/assets/image (64).png>)

7\. Click on the "Windows Credentials" tab and select "Add a Windows credential".

![](<../../.gitbook/assets/image (12).png>)

8\. Fill in the following information:

* Internet or network address: `secure.local`
* User Name: `sa_backup`
* Password: `LS6RV5o8T9`

![](<../../.gitbook/assets/image (21).png>)

## Attacking

### How it works



### Tools



### Executing the attack



## Defending

### Recommendations

* a

### Detection



## References

