# ForceChangePassword

## Configuring

### Prerequisite&#x20;

{% content-ref url="../../misc/page-3-1.md" %}
[page-3-1.md](../../misc/page-3-1.md)
{% endcontent-ref %}

### Configuring

1. Login on `DC02` with the username `Administrator` and password `Welcome01!`.
2. Open the "Active Directory Users and Computers" administration tool on `DC02`.

![](<../../../.gitbook/assets/image (11) (1) (1).png>)

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

* [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

### Executing the attack

We know the password of the user `testreset`, this is `Testing123Testing!`.  It's possible to either login with the account, or open up a PowerShell session. We will go with a PowerShell session.

1. Start PowerShell and within the taskbar right click on PowerShell and then shift+ right click on Windows PowerShell. If we do this correctly it's possible to select 'Run as different user'.\


![](../../../.gitbook/assets/image.png)

2\. Fill in the login details of the `testreset` user and click on 'OK'. Now a PowerShell window will open and we can confirm it's running under the testreset user, by typing `whoami`.

![](<../../../.gitbook/assets/image (9).png>)

3\. Within the script where we found the testreset user, we also noticed the account `sa_transfer_test` account.  If we run `net user /domain` command within PowerShell, we see the `sa_transfer` account. It might be that we have the same permissions on this account with our testreset user.

![](<../../../.gitbook/assets/image (4).png>)

We can confirm this by loading in PowerView and check the ACL's on the sa\_transfer account. We will run the following command to check this out.

{% code overflow="wrap" %}
```
 Get-ObjectAcl -SamAccountName sa_transfer -ResolveGUIDs | ? {$_.ObjectAceType -eq "User-force-Change-Password"}
```
{% endcode %}

The above command will filter out everything but reset password permissions. The output will be as follows:

![](<../../../.gitbook/assets/image (8).png>)

If we convert the SecurityIdentifier, we notice it's the user testreset.

![](<../../../.gitbook/assets/image (5).png>)

4\. We now know that we have permissions to reset the password of the `sa_transfer` user. To reset the password of `SA_transfer`, we will run the following command:

{% code overflow="wrap" %}
```
Set-DomainUserPassword -Identity sa_transfer -AccountPassword (ConvertTo-SecureString 'WeCanResetThisPassword123!' -AsPlainText -Force) -Verbose
```
{% endcode %}

![](<../../../.gitbook/assets/image (2).png>)

## Defending

### Recommendations



### Detection



## References

{% embed url="https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces#forcechangepassword" %}

{% embed url="https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#forcechangepassword" %}
