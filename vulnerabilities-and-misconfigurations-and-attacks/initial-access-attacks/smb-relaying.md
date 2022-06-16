# SMB Relaying

## Configuring

Login with an 'Administrator' account created for this attack.

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

[Responder](https://github.com/lgandx/Responder)

[Impacket-ntlmrelayx](https://raw.githubusercontent.com/SecureAuthCorp/impacket/master/examples/ntlmrelayx.py)

### Executing the attack

First we will run Responder, to look for traffic within our network. If we notice an user is active, we will try to relay the Net-NTLM-hash. The command we will run, to look passively:

```
sudo responder -I tun0 -A
```

{% hint style="info" %}
Our interface is tun0, this is because we're connected to a VPN.
{% endhint %}

After a few minutes you notice a NTLMv2-SSP hash

![](<../../.gitbook/assets/image (73).png>)

## Defending

### Recommendations



### Detection





## References

{% embed url="https://infosecwriteups.com/abusing-ntlm-relay-and-pass-the-hash-for-admin-d24d0f12bea0" %}
