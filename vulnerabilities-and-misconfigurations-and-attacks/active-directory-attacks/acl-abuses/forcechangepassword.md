# ForceChangePassword

## Configuring

### Prerequisite&#x20;

{% content-ref url="../../misc/page-3-1.md" %}
[page-3-1.md](../../misc/page-3-1.md)
{% endcontent-ref %}

### Configuring

1. Login on `DC02` with the username `Administrator` and password `Welcome01!`.
2. Open the "Active Directory Users and Computers" administration tool on `DC02`.

![](<../../../.gitbook/assets/image (11) (1).png>)

3\. Open the "Users" OU and then right click it, select "New" and "User".

4\. Fill in the name `sa_transfer` and set the password to `2i^t#fFpL`.

![](<../../../.gitbook/assets/image (34) (1).png>)

5\. Make sure "User must change password at next logon" is NOT selected and select "Password never expires".

![](<../../../.gitbook/assets/image (31) (1).png>)

6\. Right click on the `sa_transfer` user and select "Properties", open the "Security" tab and click on "Advanced".

![](<../../../.gitbook/assets/image (74).png>)

7\. Click on "Add" and then "Select a principal" and fill in the name testreset and click "Check Names" .

![](<../../../.gitbook/assets/image (6) (1).png>)

8\. Click on "OK" and select the privilege "Reset Password".

![](<../../../.gitbook/assets/image (59).png>)

9\. Click on "Ok", "Apply" and again on "OK".

## Attacking

### How it works



### Tools



### Executing the attack

## Defending

### Recommendations



### Detection



## References

