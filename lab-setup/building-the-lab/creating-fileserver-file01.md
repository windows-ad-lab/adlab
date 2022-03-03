# Creating Fileserver - FILE01

## General machine info

* Machine Name: `FILE01`
* IP Adress: `10.0.0.4`
* Subnetmask: `255.255.255.0`
* Gateway: `10.0.0.1`
* DNS: `10.0.0.3`
* Role: Filesharing services
* Domain: `amsterdam.bank.local`

## Installation after sysprep

1. Startup the machine
2. When asked if you copied the Virtual Machine, select "I Copied It"

![](<../../.gitbook/assets/afbeelding (103) (2) (4).png>)

3\. Choose the correct settings for your lab, in our example we choose for the region "Netherlands", for app language we choose "English (United States)" and for keyboard layout "United States-International"

![](<../../.gitbook/assets/afbeelding (1) (1).png>)

4\. Accept the 'License terms'.

5\. When asked to "Customize Settings" and set a password for the `Administrator` user, set the same password as before. Which was `Welcome01!`.

![](<../../.gitbook/assets/afbeelding (104).png>)

6\.  Press CTRL + ALT + DEL and login with the user and password we just set.

## Renaming and setting a static IP

1\.  Open File Explorer --> right click "This PC" --> Properties.

![](<../../.gitbook/assets/afbeelding (29) (2) (2).png>)

2\. Click on "Rename this PC".

![](<../../.gitbook/assets/afbeelding (79).png>)

3\. Fill in `FILE01`and click Next.

![](<../../.gitbook/assets/afbeelding (47).png>)

4\. When asked to restart, click on "Restart Now".

5\. Login again and rightclick in the Taskbar on the Networking Icon and select "Open Network & Internet Settings".

![](<../../.gitbook/assets/afbeelding (109) (2).png>)

6\. Click on "Change adapter options".

![](<../../.gitbook/assets/afbeelding (20).png>)

7\. Right click the Ethernet adapter and select "Properties".

![](<../../.gitbook/assets/afbeelding (102) (2).png>)

8\. Select "Internet Protocol Version 4 (TCP/IPv4) and click "Properties".

![](<../../.gitbook/assets/afbeelding (112) (2).png>)

9\. Copy the following settings:

![](<../../.gitbook/assets/afbeelding (106).png>)

10\. Click on "OK" and close all the Windows.
