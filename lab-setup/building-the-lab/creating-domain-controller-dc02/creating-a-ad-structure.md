# Creating a AD structure

1. Open the "Server Manager", click on "Tools" and then "Active Directory Users and Computers".

![](<../../../.gitbook/assets/afbeelding (52) (1) (1) (1).png>)

### Creating a Organizational Unit (OU)

2\. Extend the directories and right click on "amsterdam.bank.local", select "New" and "Organizational Unit". Give it the name `Employees` and click on "OK"

![](<../../../.gitbook/assets/image (36) (1) (1) (1) (1) (1).png>)

### Creating Groups

1. Right click on the newly created OU and select "New" and then "Group"

![](<../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1) (1).png>)

2\. Fill in "Finance" and click on "OK"

![](<../../../.gitbook/assets/image (38) (1) (1) (1) (1).png>)

3\. Repeat and create the following groups:

* Finance
* HR
* Employees
* IT

![](<../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### Creating Users

For attack path \<url to attackpath> we need to create an user account in the IT-group, to create an account we can use `net user /add <username> <password> /domain` and to add it inside the IT-group we can use `net group IT <username> /add /domain`. In this example we're using the following commands:

```
net user /add IT-support01 Sup3rCompl1c4t3dP4ssw0rd2022 /domain
net group IT IT-Support01 /add /domain
```

With the command `net user <username> /domain` it's possible to check someone's group memberships. In this example we're using the following command:

```
net user IT-Support01 /domain
```

![](<../../../.gitbook/assets/afbeelding (9) (1) (1).png>)

For attack path \<url to attackpath> we need to create an user account in the Employees-group, to create an account we can use `net user /add <username> <password> /domain` and to add it inside the Employees-group we can use `net group Employees <username> /add /domain`. In this example we're using the following commands:

```
net user /add pukcab Bangbang123 /domain
net group Employees pukcab /add /domain
```

The password `Bangbang123` is a commonly used password from 2020, see refferences for the passwordlist.

With the command `net user <username> /domain` it's possible to check someone's group memberships. In this example we're using the following command:

```
net user pukcab /domain
```

![](<../../../.gitbook/assets/afbeelding (25) (1).png>)

## References

{% embed url="https://github.com/danielmiessler/SecLists/blob/master/Passwords/2020-200_most_used_passwords.txt" %}
