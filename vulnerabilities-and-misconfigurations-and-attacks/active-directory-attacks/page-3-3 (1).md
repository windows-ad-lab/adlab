# Kerberoasting

## Configuring



1. Login to `DC01` with the `Administrator` user and the password `Welcome01!`.
2. Open PowerShell as administrator and execute the following commands to create a user with the name `sa_admin` and set a SPN for it:

```
net user sa_admin Welcome123456! /add
setspn -A MSSQLsvc/DC03:1433 sa_admin
```

3\. Execute the following command to make the user `Enterprise Admin` and `Domain Admin`:

```
Add-ADGroupMember -Identity "Enterprise Admins" -Members sa_admin
Add-ADGroupMember -Identity "Domain Admins" -Members sa_admin
```

## Attacking

### How it works

> Service principal names (SPNs) are used to uniquely identify each instance of a Windows service. To enable authentication, Kerberos requires that SPNs be associated with at least one service logon account (an account specifically tasked with running a service.
>
> Adversaries possessing a valid Kerberos ticket-granting ticket (TGT) may request one or more Kerberos ticket-granting service (TGS) service tickets for any SPN from a domain controller (DC). Portions of these tickets may be encrypted with the RC4 algorithm, meaning the Kerberos 5 TGS-REP etype 23 hash of the service account associated with the SPN is used as the private key and is thus vulnerable to offline Brute Force attacks that may expose plaintext credentials
>
> Source: [https://attack.mitre.org/techniques/T1558/003/](https://attack.mitre.org/techniques/T1558/003/)

### Tools

* [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py)

### Executing the attack

1. Use the discovered credentials `john` and password `Welcome2022!` with [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) from Impacket to Kerberoast all roastable users.

```
python3 /opt/impacket/examples/GetUserSPNs.py 'amsterdam.bank.local/john:Welcome2022!' -dc-ip 10.0.0.2 -target-domain bank.local -outputfile kerberoast.txt
cat kerberoast.txt
```

![](<../../.gitbook/assets/image (14).png>)

2\. We retrieved one hash. Lets crack it with Hashcat. The parameters are:

* Crackingmode: `-a 0` for using a wordlist
* Hashmode: `-m 18200` for Kerberos 5, etype 23, AS-REP
* List with hashes: `kerberoasting.txt`
* Passwords list: `passwords.txt`
* Rule list: `-r rules.rule`

```
.\hashcat.exe -a 0 -m 13100 .\kerberoast.txt .\wordlists\rockyou.txt -r .\rules\dive.rule
```

![](<../../.gitbook/assets/image (34).png>)

We successfully cracked the password of the user `sa_admin`, the password is `Welcome123456!`

## Defending

### Recommendations

* Use [Group Managed Service Accounts or Managed Service Accounts.](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
* Limit the privileges of Service Accounts.
* Use strong passwords (at least 32 characters) on service accounts.

### Detection



## References

{% embed url="https://attack.mitre.org/techniques/T1558/003/" %}

{% embed url="https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview" %}
