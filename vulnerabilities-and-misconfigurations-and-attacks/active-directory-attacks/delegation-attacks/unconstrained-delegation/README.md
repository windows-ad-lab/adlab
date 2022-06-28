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

![](<../../../../.gitbook/assets/image (71).png>)

3\. Open the "Computers" directory and right click the `FILE01` server and select "Properties".

![](<../../../../.gitbook/assets/image (69).png>)

4\. Open the "Delegation" tab and select "Trust this computer for delegation to any service (Kerberos only)".

![](<../../../../.gitbook/assets/image (17).png>)

5\. Click on "Apply" and "OK".

## Attacking

### How it works



### Tools



### Executing the attack



## Defending

### Recommendations



### Detection



## References

