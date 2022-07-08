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

![](<../../.gitbook/assets/image (78).png>)

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



### Tools



### Executing the attack



## Defending

### Recommendations

* a

### Detection



## References

