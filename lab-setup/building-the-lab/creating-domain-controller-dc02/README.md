# Creating Domain Controller - DC02

## General machine info

* Machine Name: `DC02`
* IP Adress: `10.0.0.3`
* Subnetmask: `255.255.255.0`
* Gateway: `10.0.0.1`
* DNS: `10.0.0.2`
* Role: Domain Services, DHCP, DNS
* Domain: `amsterdam.bank.local`

## Installation after sysprep

1. Startup the machine.
2. When asked if you copied the Virtual Machine, select "I Copied It".

![](<../../../.gitbook/assets/afbeelding (103) (1) (2).png>)

3\. Choose the correct settings for your lab, in our example we choose for the region "Netherlands", for app language we choose "English (United States)" and for keyboard layout "United States-International"

![](<../../../.gitbook/assets/afbeelding (1) (1) (1) (2) (1).png>)

4\. Accept the 'License terms'.

5\. When asked to "Customize Settings" and set a password for the `Administrator` user, set the same password as before. Which was `Welcome01!`.

![](<../../../.gitbook/assets/afbeelding (104).png>)

6\. Press CTRL + ALT + DEL and login with the user and password we just set.

## Renaming and setting a static IP

1\. Open File Explorer --> right click "This PC" --> Properties.

![](<../../../.gitbook/assets/afbeelding (17) (1) (2) (9).png>)

2\. Click on "Rename this PC".

![](<../../../.gitbook/assets/afbeelding (28).png>)

3\. Fill in `DC02` and click Next.

![](<../../../.gitbook/assets/afbeelding (42).png>)

4\. When asked to restart, click on "Restart Now".

5\. Login again and rightclick in the Taskbar on the Networking Icon and select "Open Network & Internet Settings".

![](<../../../.gitbook/assets/afbeelding (109) (1) (2).png>)

6\. Click on "Change adapter options".

![](<../../../.gitbook/assets/afbeelding (20) (1) (1) (3).png>)

7\. Right click the Ethernet adapter and select "Properties".

![](<../../../.gitbook/assets/afbeelding (102) (1) (2) (1).png>)

8\. Select "Internet Protocol Version 4 (TCP/IPv4) and click "Properties".

![](<../../../.gitbook/assets/afbeelding (112) (1) (1) (1).png>)

9\. Copy the following settings:

![](<../../../.gitbook/assets/afbeelding (13).png>)

10\. Click on "OK" and close all the Windows.

## Creating Child Domain

### Installing Domain Services

1\. Click on start and open the "Server Manager".

![](<../../../.gitbook/assets/image (3) (1) (1) (2).png>)

2\. On the right top click on "Manage" and "Add Roles and Features".

![](<../../../.gitbook/assets/afbeelding (81) (1) (1) (1).png>)

3\. Check the "Skip this page by default" box and click "Next" three times (For the steps "Before You Begin", "Installation Type" and "Server Selection".

4\. At the step "Server Roles" select "Active Directory Domain Services".

![](<../../../.gitbook/assets/afbeelding (22).png>)

5\. After we have selected "Active Directory Domain Services" a new window will pop-up. Within this window we click on "Add Features".

![](<../../../.gitbook/assets/afbeelding (101).png>)

6\. After "Active Directory Domain Services" is selected we click on "Next". We also click "Next" at the "Features" and "AD DS" steps.

7\. At the "Confirmation" step click "Install". This can take some minutes.

![](<../../../.gitbook/assets/afbeelding (39).png>)

8\. When the installation finishes close the window.

### Promoting to Child Domain Controller

1\. In the server manager click on the flag and click on "Promote this server to a domain controller"

![](<../../../.gitbook/assets/afbeelding (73).png>)

2\. Select "Add a new domain to an existing forest". After this we choose the domain type "Child Domain". In this example our domain is called 'bank.local', so we fill this in at parent domain name. Our new domain name is going to be "amsterdam.bank.local", so fill in "amsterdam". At the last step we fill in the credentials of our parent domain. Click on "Next"

![](<../../../.gitbook/assets/afbeelding (68).png>)

![The "Change..." button window](<../../../.gitbook/assets/afbeelding (96).png>)

3\. At the step "Domain Controller Options" set a DSRM Password. For this lab we will choose `AmsterdamBankRecoveryKey2022` as password.

![](<../../../.gitbook/assets/afbeelding (84).png>)

4\. For the steps "DNS Options", "Additional Options", "Paths" and "Review Options" click Next.

5\. At the step "Prerequisites Check" click "Install".

![](<../../../.gitbook/assets/afbeelding (53).png>)

With the help of PowerShell we can confirm that our child domain is created and that we have a trust to `DC01` (`bank.local`).

`Get-ADDomain`

![](<../../../.gitbook/assets/image (12) (1) (1) (1).png>)

`Get-ADTrust -Identity "bank.local"`

![Get-ADTrust -identity "bank.local" - from DC02](<../../../.gitbook/assets/afbeelding (61).png>)

### Creating extra Domain Admins

### Creating a user

1. Open the "Server Manager", click on "Tools" and then "Active Directory Users and Computers".

![](<../../../.gitbook/assets/afbeelding (52) (1) (1) (3).png>)

2\. Extend the directories and click on the folder "Users". All the default users and groups are shown in this folder.

![](<../../../.gitbook/assets/image (13) (1) (1) (1) (1).png>)

3\. Right click the "Users" directory, go to "New" and click "User"

![](<../../../.gitbook/assets/image (41) (1).png>)

4\. Fill in the following information and click on "Next".

* First name: `Amsterdam`
* Last name: `admin`
* User logon name: `admin_amsterdam`

5\. For this user we will set a password we can remember `TheBestSecureBank2022`. Make sure you save it somewhere, like in a password manager. And uncheck the box "user must change password at next logon"

![](<../../../.gitbook/assets/image (45).png>)

6\. Click "Next" and "Finish"

### Adding the user to the group

1. Right click the user and click on "Add to a group..."

![](<../../../.gitbook/assets/image (33) (1) (1).png>)

2\. Add the user to the "Domain Admins" group by typing the name into the textbox and click on "OK

![](<../../../.gitbook/assets/image (35).png>)

3\. With the following simple PowerShell command we can check if `Amsterdam admin` is part of the `Domain Admins` group.

`Get-ADGroupMember "Domain Admins"`

![](<../../../.gitbook/assets/image (55).png>)

## Installing and configuring the DHCP service

### Installing DHCP

Since we disabled DHCP in our VMWare, we need a DHCP server to lease IP-adresses to our machines which doesn't have a static IP, such as Workstations.

1. Click on start and open the "Server Manager".

![](<../../../.gitbook/assets/image (3) (1) (1) (1) (1).png>)

2\. On the right top click on "Manage" and "Add Roles and Features".

![](<../../../.gitbook/assets/afbeelding (81) (1) (1) (3).png>)

3\. Click "Next" two times (For the steps"Installation Type" and "Server Selection".

4\. At the step "Server Roles" select "DHCP Server", click "Add Features" and "Next" three times.

![](<../../../.gitbook/assets/image (4) (1) (1) (1) (1).png>)

5\. At the step "Confirmation" click "Install". Once the installation finishes click close.

6\. In the server manager click on the flag and click on "Complete DHCP configuration"

![](<../../../.gitbook/assets/image (16) (1).png>)

7\. Click "Next" and then select "User alternate credentials" and fill in the credentials for `admin_bank`.

Username: `bank\admin_bank`\
Password: `jr8Q3o97@s37AF`

![](<../../../.gitbook/assets/image (6) (1) (1).png>)

8\. Click on "Commit" and "close".

### Configuring DHCP

1. Click on "Tools" in the "Server Manager" and select "DHCP".

![](<../../../.gitbook/assets/image (59) (1).png>)

2\. Unfold the directories, right click on "IPv4" and select "New Scope"

![](<../../../.gitbook/assets/image (65) (1) (1) (1) (1).png>)

3\. Click "Next", fill in the name "DHCP Clients" and click "Next".

4\. At the step "IP Adress Range" fill in the following:

* Start IP Adress: `10.0.0.128`
* End IP Adress: `10.0.0.250`
* Length: `24`
* Subnet mask: `255.255.255.0`

![](<../../../.gitbook/assets/image (5) (1) (1) (1).png>)

5\. At the step "Add Exclusion and Delay" and "Lease Duration" click "Next". We will leave this default.

6\. At the step "Configure DHCP Options" make sure "Yes" is selected and click "Next"

7\. Fill in `10.0.0.1` for the default gateway and click on "Add", it should be in the list and then click on "Next"

![](<../../../.gitbook/assets/image (1) (1) (1) (1) (1).png>)

8\. At the step "Domain Name and DNS Server" make sure `10.0.0.2` and `10.0.0.3` are listed as DNS servers.

![](<../../../.gitbook/assets/image (14) (1) (1).png>)

9\. At the step "WINS Servers" and "Activate Scope" click "Next".

10\. Click "Finish".

11\. We should now see a Scope for IPV4.

![](<../../../.gitbook/assets/image (54).png>)
