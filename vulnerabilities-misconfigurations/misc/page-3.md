# Dumping DPAPI

## Configuring

1. Login to `DC03` as the `Administrator` user with the password `Welcome01!`.
2. Open the "Active Directory Users and Computers" management tool and open the "Users" directory. Right click the "Users" directory and click "New User"
3. Create a user with the name `adm_jony` and the password `LS6RV5o8T9`. Make sure you deselect "User must change password at next logon" and select "Password never expires".

![](<../../.gitbook/assets/image (67).png>)

![](<../../.gitbook/assets/image (38).png>)

4\. Add the user to the Domain Admin group via the interface, memberof section or run the following command:

```
Add-ADGroupMember "Domain Admins" -Members adm_jony
```

5\. Login with this user on `DATA01`.

## Attacking

### How it works



### Tools



### Executing the attack



## Defending

### Recommendations

* a

### Detection



## References

