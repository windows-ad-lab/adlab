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

![](<../../../.gitbook/assets/afbeelding (17) (1) (1).png>)

3\. Choose the correct settings for your lab, in our example we choose for the region "Netherlands", for app language we choose "English (United States)" and for keyboard layout "United States-International".

4\. Accept the 'License terms'.

5\. When asked to "Customize Settings" and set a password for the `Administrator` user, set the same password as before. Which was `Welcome01!`.

6\. Press CTRL + ALT + DEL and login with the user and password we just set.

## Renaming and setting a static IP

1. &#x20;Open File Explorer --> right click "This PC" --> Properties.

![](<../../../.gitbook/assets/afbeelding (19) (1) (1).png>)

2\.  Click on "Rename this PC".

![](<../../../.gitbook/assets/afbeelding (26) (1).png>)

3\. Fill in `DC03` and click "Next".

4\. When asked to restart, click on "Restart Now".

5\. Login again and rightclick in the Taskbar on the Networking Icon and select "Open Network & Internet Settings".

![](<../../../.gitbook/assets/afbeelding (25).png>)

6\. Click on "Change adapter options".

![](<../../../.gitbook/assets/afbeelding (36) (1).png>)

7\. Right click the Ethernet adapter and select "Properties".

![](<../../../.gitbook/assets/afbeelding (38) (1) (1) (1).png>)

8\. Select "Internet Protocol Version 4 (TCP/IPv4) and click "Properties".

![](<../../../.gitbook/assets/afbeelding (9).png>)

9\. Copy the following settings:

![](<../../../.gitbook/assets/afbeelding (17) (1).png>)

10\. Click on "OK" and close all the Windows.

## Creating a new forest

### Installing Domain Services

1\. Click on start and open the "Server Manager".

![](../../../.gitbook/assets/spaces-PqGbN7FCY7Xh4OkOtvin-uploads-git-blob-d24564630f70497fba35b7d1a7c867dca7be3db1-image.png)

2\. On the right top click on "Manage" and "Add Roles and Features".

![](<../../../.gitbook/assets/spaces-PqGbN7FCY7Xh4OkOtvin-uploads-git-blob-8b4c50fab8563bed5db78c47f290d256b40e6f66-afbeelding (81).png>)

3\. Check the "Skip this page by default" box and click "Next" three times (For the steps "Before You Begin", "Installation Type" and "Server Selection".

4\. At the step "Server Roles" select "Active Directory Domain Services".

![](../../../.gitbook/assets/spaces-PqGbN7FCY7Xh4OkOtvin-uploads-git-blob-031a92373fc86b3497aaedf7a2a0c84f5f32b0cf-afbeelding.png)

5\. After we have selected "Active Directory Domain Services" a new window will pop-up. Within this window we click on "Add Features".

![](../../../.gitbook/assets/spaces-PqGbN7FCY7Xh4OkOtvin-uploads-git-blob-135915c3c2bd3334789e52e3c4f29e78fbc8c195-afbeelding.png)

6\. After "Active Directory Domain Services" is selected we click on "Next". We also click "Next" at the "Features" and "AD DS" steps.

7\. At the "Confirmation" step click "Install". This can take some minutes.

![](../../../.gitbook/assets/spaces-PqGbN7FCY7Xh4OkOtvin-uploads-git-blob-7fcaf057403592aed790535c2f27f12930cbf2f6-afbeelding.png)

8\. When the installation finishes close the window.

### Promoting to Domain Controller

1\. In the server manager click on the flag and click on "Promote this server to a domain controller"

![](<../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1) (1).png>)

2\. Select "Add a new forest" and fill in the domain name `secure.local`.

3\. At the step "Domain Controller Options" set a DSRM Password. For this domain we will choose `SecureRecoveryKey2022` as password.

![](<../../../.gitbook/assets/image (55) (1) (1) (1) (1) (1) (1).png>)

4\. For the steps "DNS Options", "Additional Options", "Paths" and "Review Options" click Next.

5\. At the step "Prerequisites Check" click "Install".

![](<../../../.gitbook/assets/image (30) (1) (1).png>)

6\. The machine should automatically restart, if not manually restart the machine.

7\. Login with the `Administrator:Welcome01!` credentials.

### Creating DNS records

1. Login to `DC001` as the `Administrator` user with password `Welcome01!`.
2. Open the "Server Manager", click on "Tools" and open the "DNS" tool.

![](<../../../.gitbook/assets/image (4) (1).png>)

3\. In the DNS Server expand "DC01" and right click on "Conditional Forwarders" and click "new".

4\. Add the DNS Domain `secure.local` with the IP `10.0.0.100` and click "OK".&#x20;

![](<../../../.gitbook/assets/image (66) (1) (1) (1) (1) (1) (1).png>)

{% hint style="info" %}
If you see a cross don't worry. I takes some time to validate. You can check the properties later to check if it is validated.
{% endhint %}

5\. Do the same on `DC03` but then for `bank.local` with the IP `10.0.0.2`.

![](<../../../.gitbook/assets/image (54) (1) (1) (1) (1) (1) (1).png>)

### Creating trust to bank.local

1. Open the "Server Manager", click on "Tools" and then "Active Directory Domain and Trusts".

![](<../../../.gitbook/assets/image (20) (1) (1) (1) (1).png>)

2\. Right click the domain object "Secure.local" and click on "Properties".

![](<../../../.gitbook/assets/image (33) (1) (1).png>)

3\. Click on the tab "Trust", then click "New Trust" and then click "Next".

4\. On the "Trust Name" page, type the NetBIOS name `bank.local` and then click "Next".

![](<../../../.gitbook/assets/image (65) (1) (1) (1) (1) (1).png>)

5\. On the "Trust Type" page, select "External trust" and click "Next".

![](<../../../.gitbook/assets/image (61) (1) (1) (1) (1) (1) (1) (1).png>)

6\. On the "Direction of trust" page select "Two-Way" and click "Next".

7\. On the "Sides of Trust" page select "Both this domain and the specified Domain" and click "Next".

![](<../../../.gitbook/assets/image (51).png>)

8\. Fill in the the credentials Administrator and password Welcome01.

![](<../../../.gitbook/assets/image (3) (1).png>)

9\. Select "Domain-wide authentication" for this trust and click "Next".&#x20;

10\. Select "Domain-wide authentication" for this trust and click "Next" till the page "Confirm Outgoing Trust".

11\. On the page "Confirm Outgoing Trust" select "Yes, confirm the outgoing trust".

12\. On the page "Confirm Outgoing Trust" select "Yes, confirm the outgoing trust".

13\. Click "Finish"

14\. Close the Windows.

### Enable RDP

1. Open the Server Manager and click in the left menu on "Local Server".
2. Click on "Disabled" in the "Remote Desktop" section

![](<../../../.gitbook/assets/image (63) (1) (1) (1) (1).png>)

3\. Then select "Allow remote connections to this computer" in the "Remote Desktop" section and click "Apply" and "Ok".

![](<../../../.gitbook/assets/image (72) (1) (1) (1) (1) (1).png>)

Remote Desktop is enabled.
