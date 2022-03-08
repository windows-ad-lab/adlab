# Creating Domain Controller - DC01

## General machine info

* Machine Name: `DC01`
* IP Adress: `10.0.0.2`
* Subnetmask: `255.255.255.0`
* Gateway: `10.0.0.1`
* DNS: `10.0.0.2`
* Role: Domain Services
* Domain: `bank.local`

## Installation after sysprep

1. Startup the machine.
2. When asked if you copied the Virtual Machine, select "I Copied It".

![](<../../../.gitbook/assets/afbeelding (103) (1) (2) (3).png>)

3\. Choose the correct settings for your lab, in our example we choose for the region "Netherlands", for app language we choose "English (United States)" and for keyboard layout "United States-International".

![](<../../../.gitbook/assets/afbeelding (1) (1) (1) (2) (2).png>)

4\. Accept the 'License terms'.

5\. During the initial startup we have to set the `Administrator` password again. We wil use `Welcome01!` for now.

![](<../../../.gitbook/assets/afbeelding (10).png>)

6\. Press CTRL + ALT + DEL and login with the user and password we just set.

## Renaming and setting a static IP

1\. Open File Explorer --> right click "This PC" --> Properties.

![](<../../../.gitbook/assets/afbeelding (17) (1) (2) (7).png>)

2\. Click on "Change settings"

![](<../../../.gitbook/assets/afbeelding (56).png>)

3\. Click "Change" and fill in `DC01` and click OK.

![](<../../../.gitbook/assets/afbeelding (3) (1).png>)

4\. When asked to restart, click on "Restart Now"

5\. Login again and rightclick in the Taskbar on the Networking Icon and select "Open Network & Internet Settings"

![](<../../../.gitbook/assets/afbeelding (109) (1).png>)

6\. Click on "Change adapter options"

![](<../../../.gitbook/assets/afbeelding (20) (1) (1) (2).png>)

7\. Right click the Ethernet adapter and select "Properties".

![](<../../../.gitbook/assets/afbeelding (102) (1) (2) (2).png>)

8\. Select "Internet Protocol Version 4 (TCP/IPv4) and click "Properties"

![](<../../../.gitbook/assets/afbeelding (15).png>)

9\. Copy the following settings:

![](<../../../.gitbook/assets/afbeelding (2) (1).png>)

10\. Click on "OK" and close all the Windows.

## Creating the Domain

### Installing Domain Services

1. Click on start and open the "Server Manager".

![](<../../../.gitbook/assets/afbeelding (44) (1) (2) (3).png>)

2\. On the right top click on "Manage" and "Add Roles and Features".

![](<../../../.gitbook/assets/afbeelding (81) (1) (1) (2).png>)

3\. Check the "Skip this page by default" box and click "Next" three times (For the steps "Before You Begin", "Installation Type" and "Server Selection".

4\. At the step "Server Roles" select "Active Directory Domain Services" and click "Next".

![](<../../../.gitbook/assets/afbeelding (58).png>)

5\. At the steps "Features" and "AD DS" click Next and click Install.

![](<../../../.gitbook/assets/afbeelding (71).png>)

6\. When the installation finishes close the window.

### Promoting to Domain Controller

1. In the server manager click on the flag and click on "Promote this server to a domain controller"

![](<../../../.gitbook/assets/afbeelding (75).png>)

2\. Select "Add a new forest" and choose a domain name.

![](<../../../.gitbook/assets/afbeelding (43).png>)

3\. At the step "Domain Controller Options" set a DSRM Password. For this lab we will choose `BankRecoveryKey2022` as password.

![](<../../../.gitbook/assets/afbeelding (41).png>)

4\. For the steps "DNS Options", "Additional Options", "Paths" and "Review Options" click Next.

5\. At the step "Prerequisites Check" click "Install".

![](<../../../.gitbook/assets/afbeelding (23).png>)

6\. The machine should automatically restart, if not manually restart the machine.

7\. Login with the `Administrator:BankRecoveryKey2022` credentials set during the installation.

### Creating a Enterprise Admin

### Creating a user

1. Open the "Server Manager", click on "Tools" and then "Active Directory Users and Computers".

![](<../../../.gitbook/assets/afbeelding (52) (1) (1) (2).png>)

2\. Extend the directories and click on the folder "Users". All the default users and groups are shown in this folder.

![](<../../../.gitbook/assets/afbeelding (14).png>)

3\. Right click the "Users" directory, go to "New" and click "User"

![](<../../../.gitbook/assets/afbeelding (50).png>)

4\. Fill in the following information and click on "Next".

* First name: `bank`
* Last name: `admin`
* User logon name: `admin_bank`

![](<../../../.gitbook/assets/afbeelding (35).png>)

5\. For this user we will set a random password or use `jr8Q3o97@s37AF`. Make sure you save it somewhere, like in a password manager. And uncheck the box "user must change password at next logon"

![](<../../../.gitbook/assets/afbeelding (80).png>)

6\. Click "Next" and "Finish"

### Adding the user to the group

1. Right click the user and click on "Add to a group..."

![](<../../../.gitbook/assets/afbeelding (86).png>)

2\. Add the user to the "Enterprise Admins" group by typing the name into the textbox and click on "OK"

![](<../../../.gitbook/assets/afbeelding (74).png>)

3\. Right click on the user, click "Properties" and go to the "Member Of" tab. The Enterprise Admins groups should be shown there:

![](<../../../.gitbook/assets/afbeelding (69).png>)
