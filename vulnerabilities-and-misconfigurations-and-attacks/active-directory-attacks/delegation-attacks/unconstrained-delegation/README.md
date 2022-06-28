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

If a system has unconstrained delegation configured it saves kerberos tickets of users in the lsass so it can be used to authenticate to other services. An attacker could extract these saved tickets and use them against any other service.

Example: We have a webserver and webapplication that authenticates to a database too change some entries on behalf of the user. To do this kerberos unconstrained delegation is configured on the webserver.

### Tools

* Rubeus

### Executing the attack



## Defending

### Recommendations



### Detection



## References

