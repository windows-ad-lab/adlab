# Account Operators

## Configuring

### Prerequisite&#x20;

The membership of the "Account Operators" group is configured in the Dumping DPAPI page.

{% content-ref url="../../misc/page-3.md" %}
[page-3.md](../../misc/page-3.md)
{% endcontent-ref %}

![](<../../../.gitbook/assets/image (54).png>)

## Attacking

### How it works

The Account Operators group is a built in group in AD. By default it has no members.

The group grants limited account creation privileges to a user. Members of this group can create and modify most types of accounts, including those of users, local groups, and global groups, and members can log in locally to domain controllers.

By default it has no direct path to Domain Admin, but these groups might be able to add members to other groups which have other ACL's etc. In this lab (as far as I know) you cant become DA with these privileges.

### Executing the attack

* Login to the DC locally (Not through RDP but only locally).

![](<../../../.gitbook/assets/image (28).png>)

* Create users.

![](<../../../.gitbook/assets/image (65).png>)

* Add users to low privileged groups. This can be achieved through the cmdlet `Add-DomainGroupMember` from PowerView. Since we didn't create any groups in the domain we will use the "Domain Guests" group.

> Members of the Account Operators group cannot manage the Administrator user account, the user accounts of administrators, or the [Administrators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-admins), [Server Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-serveroperators), [Account Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-accountoperators), [Backup Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-backupoperators), or [Print Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-printoperators) groups. Members of this group cannot modify user rights.



![](<../../../.gitbook/assets/image (62) (1).png>)

## Defending

### Recommendations

* Never use any of the "Operator" groups. Create specialised groups and minimal permissions for the tasks the different IT departments/roles need. Use the least privilege principal.

### Detection



## References

{% embed url="https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#account-operators" %}
