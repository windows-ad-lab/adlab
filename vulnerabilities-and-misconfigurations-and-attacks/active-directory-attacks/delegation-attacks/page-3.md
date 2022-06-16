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



### Tools



### Executing the attack



## Defending

### Recommendations

* a

### Detection



## References

