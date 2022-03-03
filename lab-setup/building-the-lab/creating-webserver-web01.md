# Creating Webserver - WEB01

## General machine info

* Machine Name: `WEB01`
* IP Adress: `10.0.0.5`
* Subnetmask: `255.255.255.0`
* Gateway: `10.0.0.1`
* DNS: `10.0.0.3`
* Role: `Web Server (IIS)`
* Domain: `amsterdam.bank.local`

## Installation after sysprep

1. Startup the machine
2. When asked if you copied the Virtual Machine, select "I Copied It"

![](<../../.gitbook/assets/afbeelding (103) (1).png>)

3\. Choose the correct settings for your lab, in our example we choose for the region "Netherlands", for app language we choose "English (United States)" and for keyboard layout "United States-International"

![](<../../.gitbook/assets/afbeelding (1) (2) (2).png>)

4\. Accept the 'License terms'.

5\. During the initial startup we have to set the `Administrator` password again. We wil use `Welcome01!` for now.

![](<../../.gitbook/assets/afbeelding (113).png>)

5\. Press CTRL + ALT + DEL and login with the user and password we just set.

## Renaming and setting a static IP

1\.  Open File Explorer --> right click "This PC" --> Properties.

![](<../../.gitbook/assets/afbeelding (29) (2).png>)

2\. Click on "Change settings".

![](<../../.gitbook/assets/afbeelding (99).png>)

3\. A window called "System Properties" pops-up, within this window we need to be in the tab "Computer Name" and click on "Change..."

![](<../../.gitbook/assets/afbeelding (55).png>)

3\. A window called "Computer Name/Domain Changes" pops-up, fill in `WEB01` and click "OK".

![](<../../.gitbook/assets/afbeelding (114).png>)

4\. When asked to restart, click on "OK". Close the current Windows and a new window will pop-up and asks you to restart your computer, click on "Restart Now".

5\. Login again and right click in the Taskbar on the Networking Icon and select "Open Network & Internet Settings"

![](<../../.gitbook/assets/afbeelding (109) (1).png>)

6\. Click on "Change adapter options".

![](<../../.gitbook/assets/afbeelding (20) (1) (1).png>)

7\. Right click the Ethernet adapter and select "Properties".

![](<../../.gitbook/assets/afbeelding (102) (1).png>)

8\. Select "Internet Protocol Version 4 (TCP/IPv4) and click "Properties".

![](<../../.gitbook/assets/afbeelding (112).png>)

9\. Copy the following settings:

![](<../../.gitbook/assets/afbeelding (12).png>)

10\. Click on "OK" and close all the Windows.

## Installing Web Services
