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

![Our newly created folders](<../../.gitbook/assets/image (4).png>)

3\. Change the permissions of the folders, so only specific security groups can access those folders. We can do this by right clicking on the folder -> properties. A new window will pop-up, go to Security and click on Advanced.

![](<../../.gitbook/assets/image (1).png>)

4\. After clicking on advanced a new window will pop-up and we need to change some settings within this window.

4.1 Click on 'Disable inheritance' and 'Convert inherited permissions into explicit permissions on this object'.

4.2 Remove the permissions from Users.

4.3 Add the created group IT. In our example we gave the group IT Modify, Read & execute, List folder contents, Read and Write permissions.

When everything has been followed from step 4, you will get something like this:

![](../../.gitbook/assets/image.png)

Follow step 4 for all folders you have created. You can even add other security groups to spice things up.



## Attacking

### How it works

When testing an active directory environment, you will stumble on some credentials eventually. For example, you have access to a Windows-machine and got administrator privileges and you're able to dump hashes from the machine. Or during the test you will get credentials from your client, but you're unaware what you can do with those credentials.

Within this attack, we will show you how you can enumerate shares where you have access too. You can enumerate the shares with a password or with a NTLM-hash.

### Tools

Crackmapexec

smbmap

### Executing the attack

Within this attack, we're going to use the NTLM-hash from the user `noah`, which we got from dumping the hashes on WS01

{% content-ref url="different-methods-of-dumping-credentials/services.md" %}
[services.md](different-methods-of-dumping-credentials/services.md)
{% endcontent-ref %}

The hash we can use with a tool called CrackMapExec. CrackMapExec has the ability to list the shares, which we can see and where we have permissions on.

1. We will run CrackMapExec on our attacker machine. So start-up your attacker machine and open up a terminal.
2. We will be executing CrackMapExec with the user noah and the found NTLM-hash. The command we executed:

```
cme smb 10.0.0.0/24 -u noah -H 02671733e400aa35ad23828f5df3b676 --shares
```

You can also run CrackMapExec with the -p flag instead of the flag -H. With the flag -p you have to supply the password of the user, the password we gave the user noah is `haoNHasAStrongPassword321!@`.

In our case we run the command with the NTLM-hash, the result is:

![](<../../.gitbook/assets/image (5).png>)

As we can see, we have read permissions to multiple shares. We also notice the share `Data`, which has read and write permissions.

We can list the folders we have created with a tool called `smbmap`. We will do this again with the NTLM-hash of the user noah.

1. Hopefully you still have your attacker machine open, if not, open it up again.
2. We will execute smbmap and non-recursively show the content of the `Data` folder. This will be done through the -r flag. If you want to watch the content recursively you have to use the -R flag. The command we will execute:

{% code overflow="wrap" %}
```
smbmap -u noah -p 'aad3b435b51404eeaad3b435b51404ee:02671733e400aa35ad23828f5df3b676' -H 10.0.0.4 -d amsterdam.bank.local -r Data
```
{% endcode %}

The output of the command:

![](<../../.gitbook/assets/image (6).png>)

next we will look for interesting content. This can be found on the below page

{% content-ref url="page-3-1.md" %}
[page-3-1.md](page-3-1.md)
{% endcontent-ref %}

## Defending

### Recommendations

* a

### Detection



## References

