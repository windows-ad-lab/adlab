# Attack path 2

## Attack path visualized

![](<../../../.gitbook/assets/attack\_path2.drawio (1).png>)

## Tasks for the specified attack path

1. Enumerate valid usernames.
2. Check if any of the valid users has the 'DONT\_REQ\_PREAUTH' set.
3. Access the SQL instance.
4. Escalate privileges to sysadmin.
5. Get a shell on the SQL server.
6. Escalate privileges with Resource Based Constrained Delegation.
7. Enumerate SQL Links.
8. Perform UNC Path injection and capture the hash of the user the SQL server is running as.
9. Crack the hash retrieved through UNC PATH injection
10. Abuse the write Owner acl.
11. Check the description for passwords.
12. Check for SPN's across trusts.
13. Take over the `bank.local` forest.

## Steps for the specified attack path

{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/privilege-escalation/enumerate-logins/" %}
[enumerate-logins](../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/privilege-escalation/enumerate-logins/)
{% endcontent-ref %}

{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/password-spraying.md" %}
[password-spraying.md](../../../vulnerabilities-misconfigurations/active-directory-attacks/password-spraying.md)
{% endcontent-ref %}

{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/initial-access/normal-domain-user-access.md" %}
[normal-domain-user-access.md](../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/initial-access/normal-domain-user-access.md)
{% endcontent-ref %}

{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/privilege-escalation/impersonation.md" %}
[impersonation.md](../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/privilege-escalation/impersonation.md)
{% endcontent-ref %}

{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/privilege-escalation/db-owner.md" %}
[db-owner.md](../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/privilege-escalation/db-owner.md)
{% endcontent-ref %}

{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/executing-commands.md" %}
[executing-commands.md](../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/executing-commands.md)
{% endcontent-ref %}

