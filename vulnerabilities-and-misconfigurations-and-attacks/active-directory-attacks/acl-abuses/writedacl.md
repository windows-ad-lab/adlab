# WriteDacl

## Configuring

1. Login to `DC03` with the Administrator user and the password `Welcome01!`.
2. Open the "Active Directory Users and Computers" administration tool.

![](<../../../.gitbook/assets/image (24).png>)

3\. Click on "View" and enable "Advanced Features.

![](<../../../.gitbook/assets/image (67).png>)

4\. Right click the "Secure.local" domain object and select "Properties". Then open the "Security" tab and click "Add".

![](<../../../.gitbook/assets/image (77) (1).png>)

5\. Click on "Object Types" and select "Computers", then select "OK".

![](<../../../.gitbook/assets/image (73).png>)

6\. Fill in `Data01` and click "Check Names", then click "Advanced" and "OK".

![](<../../../.gitbook/assets/image (49).png>)

7\. Click on "DATA01" and then on "Advanced". Scroll through the list and search for the `DATA01` entry. Then click on "Edit" and select "Modify permissions".

![](<../../../.gitbook/assets/image (12).png>)

8\. Click on "OK", "Apply" and "OK". Then on "Apply" and "OK" again to close and apply all the permissions screens.

## Attacking

### How it works

With the ability to modify the DACL on the target object, you can grant yourself almost any privilege against the object you wish. Basically giving yourself Genericall over the object.

{% hint style="info" %}
In the lab we will only give a user DCSync privileges to the domain object. But it is possible to give almost all privileges to the object you have the rights too and then abuse these privileges like described in the GenericAll page.
{% endhint %}

### Tools

* [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

### Executing the attack

The attack is executed from the perspective of already having high privileged access to the `DATA01` server in the `secure.local` domain and having control of an user.

1. Login to `DATA01` with the username `bank\secure_admin` and the password `rFKbUJrDu$sz*36ffKr6`.
2. In the page DACL-Abuses I showed you how to check for ACL's using BloodHound. In this attack we will abuse the ACL Data01 has on the domain object:

![](<../../../.gitbook/assets/image (62).png>)

3\. Open PowerShell as an Administrator and download an amsi, MimiKatz and PowerView into memory.

![](<../../../.gitbook/assets/image (77).png>)

4\. We are currently running in the context of the secure\_admin user, but we need to run in the context of the `DATA01` computeraccount, we can do this by getting system. Execute the following MimiKatz command to do just that:

```
Invoke-Mimikatz -Command '"privilege::debug" "token::elevate"'
```

![](<../../../.gitbook/assets/image (50) (1).png>)

5\. Now we can give `secure_admin` DCSync privileges on the `secure.local` domain object using the Add-DomainObjectAcl cmdlet from PowerView:&#x20;

```
Add-DomainObjectAcl -TargetIdentity 'DC=secure,DC=local' -PrincipalIdentity 'secure_admin' -PrincipalDomain bank.local -Rights DCSync -Verbose
```

![](<../../../.gitbook/assets/image (61).png>)

7\. We can quickly run BloodHound to check if the correct permissions are applied to the `secure_admin` user:

![](<../../../.gitbook/assets/image (19).png>)

![](<../../../.gitbook/assets/image (79).png>)

8\. The next steps of the attack to execute DCSync is described on the following page:

{% content-ref url="get-changes.md" %}
[get-changes.md](get-changes.md)
{% endcontent-ref %}

## Defending

### Recommendations



### Detection



## References

{% embed url="https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#writedacl" %}
