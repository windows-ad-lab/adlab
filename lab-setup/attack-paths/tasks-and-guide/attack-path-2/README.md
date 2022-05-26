# Attack path 2

## Tasks for the specified attack path

1. Enumerate valid domain users.
2. Check if any of the valid users are vulnerable to a kerberos vulnerability.
3. Check if the user can access any system or service.
4. Escalate privileges to sysadmin.
5. Get a shell.
6. Escalate privileges on the machine with a delegation attack.
7. Enumerate SQL Links to hop across trusts.
8. Perform a UNC Path injection and capture the hash of SQL Service user.
9. Crack the hash of the SQL service user.
10. Check for ACL's and get access to the system.
11. Look for saved credentials on the machine (DPAPI).
12. Become domain admin by abusing the "Backup Operators" group.
13. Check for interesting user attributes across the trust to `bank.local`.
14. Take over the `bank.local` forest.

{% hint style="danger" %}
Below here there WILL be big spoilers on what to do to exploit the misconfigurations.
{% endhint %}

## Attack path visualized

![](<../../../../.gitbook/assets/image (69).png>)
