# Creating Domain Controller - DC03

## General machine info

* Machine Name: `DC03`
* IP Adress: `10.0.0.100`
* Subnetmask: `255.255.255.0`
* Gateway: `10.0.0.1`
* DNS: `10.0.0.100`
* Role: Domain Services
* Domain: `secure.local`

## Installation after sysprep

1. Startup the machine.
2. When asked if you copied the Virtual Machine, select "I Copied It".

![](<../../../.gitbook/assets/afbeelding (17) (1).png>)

3\. Choose the correct settings for your lab, in our example we choose for the region "Netherlands", for app language we choose "English (United States)" and for keyboard layout "United States-International".

4\. Accept the 'License terms'.

5\. When asked to "Customize Settings" and set a password for the `Administrator` user, set the same password as before. Which was `Welcome01!`.

6\. Press CTRL + ALT + DEL and login with the user and password we just set.

## Renaming and setting a static IP

1. &#x20;Open File Explorer --> right click "This PC" --> Properties.

![](<../../../.gitbook/assets/afbeelding (19) (1).png>)

2\.  Click on "Rename this PC".

![](<../../../.gitbook/assets/afbeelding (26).png>)

3\. Fill in `DC03` and click "Next".

4\. When asked to restart, click on "Restart Now".

5\. Login again and rightclick in the Taskbar on the Networking Icon and select "Open Network & Internet Settings".

![](<../../../.gitbook/assets/afbeelding (25).png>)

6\. Click on "Change adapter options".

![](<../../../.gitbook/assets/afbeelding (36).png>)

7\. Right click the Ethernet adapter and select "Properties".

![](<../../../.gitbook/assets/afbeelding (38) (1) (1).png>)

8\. Select "Internet Protocol Version 4 (TCP/IPv4) and click "Properties".

![](<../../../.gitbook/assets/afbeelding (9).png>)

9\. Copy the following settings:

![](<../../../.gitbook/assets/afbeelding (17).png>)

10\. Click on "OK" and close all the Windows.

## Creating a new forest

### Installing Domain Services

1\. Click on start and open the "Server Manager".

