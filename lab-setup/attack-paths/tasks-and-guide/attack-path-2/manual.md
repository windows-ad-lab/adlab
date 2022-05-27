# Manual

## Attack path visualized

![](<../../../../.gitbook/assets/image (68).png>)

### 1. Enumerate Users

Task: Enumerate valid domain users.



### 2. AS-REP roast

Task: Check if any of the valid users are vulnerable to a kerberos vulnerability.



### 3. Access SQL Server

Task: Check if the user can access any system or service.



### 4. Become sysadmin

Task: Escalate privileges to sysadmin.



### 5. Execute commands

Task: Get a shell.



### 6. Privesc RBCD

Task: Escalate privileges on the machine with a delegation attack.



### 7. Abuse SQL Link

Task: Enumerate SQL Links to hop across trusts.



### 8. UNC Path injection

Task: Perform a UNC Path injection and capture the hash of SQL Service user.



### 9. Crack SQL account hash

Task: Crack the hash of the SQL service user.



### 10. ACL - Write Owner & RCBCD

Task: Check for ACL's and get access to the system.



### 11. Dumping DPAPI

Task: Look for saved credentials on the machine (DPAPI).



### 12. Privileged groups

Task: Become domain admin by abusing the "Backup Operators" group.



### 13. Kerberoast

Task: Check for interesting user attributes across the trust to `bank.local`.



### 14. Take over Foresat

Task: Take over the `bank.local` forest.

