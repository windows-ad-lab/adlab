# Creating File/SQL Server - DATA01

## General machine info

* Machine Name: `DATA01`
* IP Adress: `10.0.0.101`
* Subnetmask: `255.255.255.0`
* Gateway: `10.0.0.1`
* DNS: `10.0.0.100`
* Role: Domain Services
* Domain: `secure.local`

## Installation after sysprep

1. Startup the machine.
2. When asked if you copied the Virtual Machine, select "I Copied It".

![](<../../../../.gitbook/assets/afbeelding (38) (1) (1).png>)

3\. Choose the correct settings for your lab, in our example we choose for the region "Netherlands", for app language we choose "English (United States)" and for keyboard layout "United States-International".

4\. Accept the 'License terms'.

5\. When asked to "Customize Settings" and set a password for the `Administrator` user, set the same password as before. Which was `Welcome01!`.

6\. Press CTRL + ALT + DEL and login with the user and password we just set.

## Renaming and setting a static IP

1. &#x20;Open File Explorer --> right click "This PC" --> Properties.

![](<../../../../.gitbook/assets/afbeelding (2).png>)

2\.  Click on "Rename this PC".

![](<../../../../.gitbook/assets/afbeelding (6).png>)

3\. Fill in `DATA01` and click "Next".

4\. When asked to restart, click on "Restart Now".

5\. Login again and rightclick in the Taskbar on the Networking Icon and select "Open Network & Internet Settings".

6\. Click on "Change adapter options".

![](<../../../../.gitbook/assets/afbeelding (19) (1).png>)

7\. Right click the Ethernet adapter and select "Properties".

8\. Select "Internet Protocol Version 4 (TCP/IPv4) and click "Properties".

![](<../../../../.gitbook/assets/afbeelding (46) (1).png>)

9\. Copy the following settings:

![](<../../../../.gitbook/assets/afbeelding (44) (1) (3).png>)

10\. Click on "OK" and close all the Windows.

## Joining the domain

1\. Open File Explorer --> right click "This PC" --> Properties

![](<../../../../.gitbook/assets/image (37) (1) (1) (1).png>)

2\. In the top right corner click on "Rename this PC (advanced)"

![](<../../../../.gitbook/assets/image (63) (1) (1) (1) (1) (1).png>)

3\. A new window will pop-up called 'System Properties', within this window click on "Change..."

4\. Another window will pop-up called 'Computer Name/Domain Changes', within this window change the 'Domain' to `secure.local` and click on "OK"

![](<../../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1).png>)

5\. You will be asked to supply credentials, we will use the `administrator` account from `secure.local` and click on "OK". Password is `Welcome01!`.

6\. We're welcomed to the domain, click on "OK". After this message it will asks us to reboot the machine, click on "OK". Close the 'System Properties' window and restart the machine now.
