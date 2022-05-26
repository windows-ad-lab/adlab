# Attack path 2

## Attack path visualized

![](<../../../.gitbook/assets/attack\_path2.drawio (2).png>)

## Tasks for the specified attack path

1. Enumerate valid usernames.
2. Check if any of the valid users has interesting attributes.
3. Look for anything interesting the new user can access.
4. Escalate privileges to sysadmin.
5. Get a shell on the server.
6. Escalate privileges on the machine with a delegation attack.
7. Enumerate SQL Links to hop accross trusts.
8. Perform UNC Path injection and capture the hash of SQL Service user.
9. Crack the hash.
10. Check for ACL's and get access to the system.
11. Look for saved credentials on the machine (DPAPI).
12. Become domain admin by abusing the "Backup Operators" group.
13. Check for interesting user attributes accross the trust to `bank.local`.
14. Take over the `bank.local` forest.

## Steps for the specified attack path

{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/privilege-escalation/enumerate-logins/" %}
[enumerate-logins](../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/privilege-escalation/enumerate-logins/)
{% endcontent-ref %}

{% content-ref url="../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/as-rep-roasting.md" %}
[as-rep-roasting.md](../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/as-rep-roasting.md)
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



{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/database-links.md" %}
[database-links.md](../../../vulnerabilities-misconfigurations/active-directory-attacks/sql-server-attacks/database-links.md)
{% endcontent-ref %}



##

