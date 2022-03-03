# Creating W10 client - WS01

## General machine info

* Machine Name: `WS01`
* IP Adress: `DHCP`
* Subnetmask: `DHCP`
* Gateway: `DHCP`
* DNS: `DHCP`
* Role: Workstation for end users
* Domain: `amsterdam.bank.local`

## Installation after sysprep

1. Startup the machine
2. When asked if you copied the Virtual Machine, select "I Copied It"

![](<../.gitbook/assets/afbeelding (108).png>)

3\. Choose your region, in our example "Netherlands" and click "Yes"

![](<../.gitbook/assets/afbeelding (94).png>)

4\. Choose your keyboard layout, in our example "United States-International" and click "Yes"

![](<../.gitbook/assets/afbeelding (18).png>)

5\. On the second layout screen, click "Skip". After this our machine will do some basic configurations and updates.

6\. Once the Pop-up comes to agree with the Windows terms and conditions, click "Agreed".

7\. Now we're asked to 'Sign in with Microsoft' account and we click on "Domain join instead".

![](<../.gitbook/assets/afbeelding (33).png>)

8\. Create an user-account with the password `Welcome01!`, because we already created the user-account "User", we need to create a second user. In our example we choose `User02`.

9\. Next they ask for some security questions, we fill in "A" because we will delete the account in a later stadium anyway.

10\. Choose the most privacy friendly options. We choose the following options: No, No, Send Required diagnostic data, No, No, No.

Now the machine will spin up and load our user-profile. After some minutes the machine is ready to use.

![](<../.gitbook/assets/afbeelding (59).png>)

## Renaming

1. Open File Explorer --> right click "This PC" --> Properties

![](<../.gitbook/assets/afbeelding (46).png>)

2\. Click on "Rename this PC"

![](<../.gitbook/assets/afbeelding (115).png>)

3\. Fill in `WS01` and click "Next"

![](<../.gitbook/assets/afbeelding (32).png>)

4\. When asked to restart, click on "Restart Now"
