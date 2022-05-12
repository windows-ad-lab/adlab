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



## Defending

### Recommendations



### Detection



## References

