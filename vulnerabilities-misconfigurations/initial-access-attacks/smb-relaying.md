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

SET BaseRange is the ip-range where our kali machine is located.\
Set Min and Max, is the specific host within this ip-range. Our kali machine is located within the ip-range `192.168.248.1` through `192.168.248.25`

The for loop is going to loop through the ip-range and tries to 'copy' a file from a non-existing share. This will result in leaking the NTLM-hash.

We need to save this file to a location accessible for the user who's going to run the bat-file. In our example the user who's going to run the scheduled task is pukcab. We have put the file inside the documents folder.

#### Creating a scheduled task

1\. Open up 'Task Scheduler'

![](<../../.gitbook/assets/afbeelding (29).png>)

2\. Select 'Task Scheduler Library' and create a new task

![](<../../.gitbook/assets/afbeelding (26).png>)

3\. In the 'General' tab, select "Run whetever user is logged on or not". Also we need to change the user, this can be done by clicking on "Change User..." and login with the user we have created for this attack, in our case `pukcab:Bangbang123`&#x20;

![](<../../.gitbook/assets/afbeelding (9).png>)

4\. At the 'Triggers' tab, click on "New...". Within the new tab, set 'Begin the task:' on "At startup". Check "Repeat task every:" and set the value "5 minutes" and set 'for a duration of:' to "indefinitely".

![](<../../.gitbook/assets/afbeelding (4).png>)

5\. In the 'Actions' tab click on 'New...' and browse to your .bat file and click on 'OK'. You should have about the same as us:

![](<../../.gitbook/assets/afbeelding (6).png>)

6\. Conditions doesn't need any change, so we can go to the last tab called 'Settings'. Within 'Settings' the only thing we need to change is 'Do not start a new instance' to "Run a new instance in parralel".

![](../../.gitbook/assets/afbeelding.png)

7\. Click on "OK" and fill in the credentials of the user. After we restart our workstation, the scheduled task will run every 5 minutes.

## Attacking

### How it works







### Executing the attack







## Defending

### Recommendations



### Detection





## References
