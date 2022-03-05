# Empty Password

## Configuring

1. Open the "Active Directory Users and Computers" administration tool.

![](<../../../.gitbook/assets/image (5).png>)

2\. Click on "View" en enable "Avanced Features"

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



## Attacking

### How it works



### Tools



### Executing the atttack



## Defending

### Recommendations



### Detections



## References

{% embed url="https://specopssoft.com/blog/find-ad-accounts-using-password-not-required-blank-password" %}
