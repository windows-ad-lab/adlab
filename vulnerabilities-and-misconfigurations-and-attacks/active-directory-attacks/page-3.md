# AS-REP Roasting

## Configuring

### Prerequisite&#x20;

{% content-ref url="../../vulnerabilities-misconfigurations/active-directory-attacks/password-spraying.md" %}
[password-spraying.md](../../vulnerabilities-misconfigurations/active-directory-attacks/password-spraying.md)
{% endcontent-ref %}

### Configuring

1. Login on `DC02` with the username `Administrator` and password `Welcome01!`.
2. Open the "Active Directory Users and Computers" administration tool on `DC02`.

![](<../../.gitbook/assets/image (34).png>)

3\. Click on "View" and enable "Advanced Features".

![](<../../.gitbook/assets/image (13).png>)

4\. Open "Users", right click the user `banktest` and click on "Properties"

5\. Open "Account" and scroll to the bottom in "Account options", then select "Do not require kerberos preauthentication".

![](<../../.gitbook/assets/image (45).png>)

## Attacking

### How it works

When pre-authentication is not required, an attack can directly send a request for authentication. The KDC will return an encrypted TGT and the attacker can extract the hash and brute-force it offline.

### Tools

* Crackmapexec

### Executing the attack

AS-REP roasting was already covered in the Initial Access Attacks section.&#x20;

{% content-ref url="../initial-access-attacks/username-enumeration/as-rep-roasting.md" %}
[as-rep-roasting.md](../initial-access-attacks/username-enumeration/as-rep-roasting.md)
{% endcontent-ref %}

But since we have a set of valid credentials of the domain now, we could request a list of all usernames and check for AS-REP roastable users.



## Defending

### Recommendations

* a

### Detection



## References

