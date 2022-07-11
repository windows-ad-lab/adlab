# Password on shares

## Configuring

### Prerequisite&#x20;

{% content-ref url="page-3.md" %}
[page-3.md](page-3.md)
{% endcontent-ref %}

### Configuring

1. Login on `DC02` with the username `Administrator` and password `Welcome01!`.
2. Open the "Active Directory Users and Computers" administration tool on `DC02`.

![](<../../.gitbook/assets/image (50).png>)

3\. Right click on the "Users" directory and select "New" and then "User". Create a user with the name `testreset`.

![](<../../.gitbook/assets/image (29).png>)

4\. Fill in the password `Testing123Testing!` and select "Password never expires" and make sure "User must change password at next logon" is not selected.

5\. Login on `FILE01` with the username `Administrator` and password `Welcome01!`.

6\. Open "Windows Explorer" and browse to `C:\Share\IT`.

![](<../../.gitbook/assets/image (80).png>)

7\. Create a directory "Scripts" and enter the Directory. Then create a new textdocument with the name `Password-reset-transferuser-testscript.ps1` and place the following content inside it:

```
$password = 'Testing123Testing!'
$credentials = New-Object System.Management.Automation.PsCredential("amsterdam\testreset",$password)

Import-module ActiveDirectory
#Generate a 15-character random password.
$Password = -join ((33..126) | Get-Random -Count 15 | ForEach-Object { [char]$ })
$NewPwd = ConvertTo-SecureString $Password -AsPlainText -Force
$user = "sa_transfer_test"
Set-ADAccountPassword $user -NewPassword $NewPwd -Reset -Credential $credentials
#Display userid and new password on the console.
Write-Host $user, $Password
```

![](<../../.gitbook/assets/image (78) (1).png>)

8\. Download a couple bogus scripts to the server and place them in the same folder such as:

```
wget https://raw.githubusercontent.com/ruudmens/LazyAdmin/master/Windows%2010/CreateLocalAdminAcc.ps1
wget https://raw.githubusercontent.com/ruudmens/LazyAdmin/master/ActiveDirectory/Add-UsersToGroup.ps1
wget https://raw.githubusercontent.com/ruudmens/LazyAdmin/master/ActiveDirectory/CleanupDisabledUsers.ps1
wget https://raw.githubusercontent.com/ruudmens/LazyAdmin/master/ActiveDirectory/Get-ADUsers.ps1
wget https://raw.githubusercontent.com/ruudmens/LazyAdmin/master/ActiveDirectory/Get-ADComputers.ps1
```

## Attacking

### How it works

During our pentests I often find shares that are accessible for all users and often there are some credentials to be discovered in either configuration files, scripts or Microsoft documents(`.doc`, `.xls`) made to store passwords. Sometimes there even is a NAS that allows null session and read/write on all shares.

### Tools

*

### Executing the attack

The attack is executed from the perspective of already having discovered the IT Share on `FILE01` and having access to it.

{% content-ref url="page-3.md" %}
[page-3.md](page-3.md)
{% endcontent-ref %}

1. For having easy access to look at the share login as `noah` with the password `haoNHasAStrongPassword321!@` on `WS01`.
2. Open File Explorer and browse to `\\file01\Data`.

![](<../../.gitbook/assets/image (22).png>)

3\. Open the IT directory and then the Script directory.

![](<../../.gitbook/assets/image (56).png>)

4\. While looking through the scripts we see that the script `Password-reset-transferuser-testscript` has login credentials saved in the script:

![](<../../.gitbook/assets/image (18).png>)

5\. The password for the user `testreset` probably is `Testing123Testing!`. After reading through the code it seems that its a script to reset the credentials of the user `sa_transfer_test` with the credentials of `testreset`.

We can easily test this by starting a new PowerShell session as this user by opening powershell, then in the taskbar right click on it and shift right clicking on Window PowerShell and then selecting "Runas as different user".

![](<../../.gitbook/assets/image (27).png>)

6\. Copy and paste the credentials and click on "OK". A PowerShell window opens, type `whoami` to check if we are running as the `testreset` user:

![](<../../.gitbook/assets/image (19).png>)

On the following page the ACL's this account has will be abused by resetting a password from another user:

{% content-ref url="../active-directory-attacks/acl-abuses/forcechangepassword.md" %}
[forcechangepassword.md](../active-directory-attacks/acl-abuses/forcechangepassword.md)
{% endcontent-ref %}

## Defending

### Recommendations

* Don't save credentials in readable files.
