# Get-Changes

## Configuring

### Prerequisite&#x20;

{% content-ref url="writedacl.md" %}
[writedacl.md](writedacl.md)
{% endcontent-ref %}

### Configuring

Nothing need to be configured to abuse this since we set the ACL's of the object during the attack in the "WriteDacl" section.

## Attacking

### How it works

With the privileges DS-Replication-Get-Changes and DS-Replication-Get-Changes-All rights it is possible to execute the DCSync attack. Which will sync all(or a specifichash) of the users.

### Tools

* [MimiKatz](https://github.com/samratashok/nishang/blob/master/Gather/Invoke-Mimikatz.ps1)

### Executing the attack

1. Login to `DATA01` with the username `bank\secure_admin` and the password `rFKbUJrDu$sz*36ffKr6`.
2. In the page DACL-Abuses I showed you how to check for ACL's using BloodHound. In this attack we will abuse the ACL `secure_admin` has on the domain object:

![](<../../../.gitbook/assets/image (25).png>)

3\. Open PowerShell as an Administrator and download an amsi and MimiKatz into memory.

![](<../../../.gitbook/assets/image (23).png>)

4\. Execute the DCSync attack with MimiKatz and retrieve the `Administrator` hash with the following command:

```
Invoke-Mimikatz -Command '"lsadump::dcsync /domain:secure.local /user:Administrator"'
```

![](<../../../.gitbook/assets/image (46).png>)

### Cleanup

1. Login to `DC03` as `Administrator` with the password `Welcome01!`.
2. Open "Active Directory Users and Computers" and open the "Computers" section and open the "Properties" for the domainobject `secure.local`. Make sure the "Advanced Features" are enabled.
3. Open the "Security" tab and select the `secure_admin` user. Then click "Remove"

![](<../../../.gitbook/assets/image (71).png>)

4\. Then click "Apply" and "OK".&#x20;

## Defending

### Recommendations



### Detection



## References

{% embed url="https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#id34" %}
