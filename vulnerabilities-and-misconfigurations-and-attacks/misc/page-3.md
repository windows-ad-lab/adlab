# Discovering Shares

## Configuring

### Prerequisite&#x20;

* The user _noah_ which has been made on the below page

{% content-ref url="different-methods-of-dumping-credentials/services.md" %}
[services.md](different-methods-of-dumping-credentials/services.md)
{% endcontent-ref %}

* Server with File and Storage services installed. And created a share, which can be used for the configuration part.

{% content-ref url="../../lab-setup/building-the-lab/creating-fileserver-file01/file-services.md" %}
[file-services.md](../../lab-setup/building-the-lab/creating-fileserver-file01/file-services.md)
{% endcontent-ref %}

### Configuring

**Creating security groups**

1. Login with the `Administrator` user on `DC02` and the password `Welcome01!`.&#x20;
2. Start-up PowerShell. With PowerShell we're going to make a new OU, called `IT-department`_._ Within the new OU, we're going to create a security group called `IT`_._ To create the OU, we're going to execute the following command:

{% hint style="info" %}
If you made changes to your domain-name, please change it within the commands
{% endhint %}

```
New-ADOrganizationalUnit -Name "IT-department" -Path "OU=Employees,DC=amsterdam,DC=bank,DC=local"
```

3\. Within the same PowerShell Window, we're going to make a security group. This can be done with the following command:

```
New-ADGroup -Name "IT" -DisplayName "IT" -GroupScope Universal -GroupCategory Security  -Path "OU=IT-department,OU=Employees,DC=amsterdam,DC=bank,DC=local"
```

4\. Next we will add the user `noah` to the IT group, this can be done with the following command

```
net group IT Noah /add
```

**Creating shares**

In this example we will use the already created SMB share called `Share`.

1. Login with an administrative user on a server with File and Storage services installed. In our example we will login with `Administrator` on `FILE01`.
2. Open file explorer and proceed to the share which you have created, in our example is the share called `Share`. Within the share, make some new folders. For example `Backup`, `Employee` and `IT`.

![Our newly created folders](<../../.gitbook/assets/image (53).png>)

3\. Change the permissions of the folders, so only specific security groups can access those folders. We can do this by right clicking on the folder -> properties. A new window will pop-up, go to Security and click on Advanced.

![](<../../.gitbook/assets/image (2).png>)

4\. After clicking on advanced a new window will pop-up and we need to change some settings within this window.

4.1 Click on 'Disable inheritance' and 'Convert inherited permissions into explicit permissions on this object'.

4.2 Remove the permissions from Users.

4.3 Add the created group IT. In our example we gave the group IT Modify, Read & execute, List folder contents, Read and Write permissions.

When everything has been followed from step 4, you will get something like this:

![](<../../.gitbook/assets/image (42).png>)

Follow step 4 for all folders you have created. You can even add other security groups to spice things up.



## Attacking

### How it works



### Tools



### Executing the attack



## Defending

### Recommendations

* a

### Detection



## References

