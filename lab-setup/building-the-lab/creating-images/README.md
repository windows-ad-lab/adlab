---
description: >-
  We are expecting that you know how to create a virtual machine and install a
  Windows client or server. Below is a short manual on how to do so for some
  guidance.
---

# Creating images

## Why create images?

It is way easier to create an updated image with all the updates etc. of an operating system and clone it then to install updates, software on every machine all the time.

## Downloading ISO's

Windows ISO's can be downloaded from the [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server). For this lab we will be using the following operating system so please download all four the ISO's under the [Windows](https://www.microsoft.com/en-us/evalcenter/evaluate-windows) and [Windows Server](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server) category:

* Windows Server 2022
* Windows Server 2019
* Windows 10

## Creating a virtual machine VMware

For this lab we will be creating images which can be used to quickly clone and deploy another server or client. Hopefully you already know how to create a virtual machine in the hypervisor you are using. For demonstration I will show how to create a virtual machine in VMware Workstation.

**I would recommend creating all virtual machine images at the same time!**

1. Open VMware workstation and in the top left click on File --> New Virtual Machine and a pop-up should come up to create a new virtual machine.
2. On the step "What type of configuration you want" select "Typical (recommended)" and click Next.
3. On the step "Guest Operating System Installation" select "Installer disc image file (iso)" and select the Windows Server 2019 ISO. Click Next.
4. On the step "Select guest Operating system" select "Microsoft Windows" and in the dropdown menu select "Windows Server 2019. Click Next.
5. Give the the name "Windows Server 2019 Image" and click Next.
6. Choose the default Disk Size and click Next.
7. Click Finish.

Repeat these steps for each Image we would like to create (Windows Server 2019, 2022 and Windows 10)

## Creating a image

1. Start-up the virtual machine and quickly pres a key to boot to the CD.
2. Make sure the language is set to "English" and the keyboard layout to "US" or your own keyboard layout and click Next.

![](<../../../.gitbook/assets/afbeelding (37).png>)

3\. Click Install Now.

* When asked to select an operating system select "Windows Server Standard Evaluation (Desktop Experience)" and click Next.

![](<../../../.gitbook/assets/afbeelding (57).png>)

4\. Accept the licene terms and click Next.

![](<../../../.gitbook/assets/afbeelding (65).png>)

5\. Select "Custom: Install Windows only" and select the disk, click on New and Apply.

![](<../../../.gitbook/assets/afbeelding (82).png>)

![](<../../../.gitbook/assets/afbeelding (8).png>)

6\. Select the Primary Partition and click Next.

![](<../../../.gitbook/assets/afbeelding (21).png>)

7\. After the installation finishes the machine will reboot. Make sure you remove the ISO by going to the settings of the machine and disconnecting the CD drive.

8\. Start the VM and go through the installation. For the Image just choose a simple Administrator password which you could remember, such as `Welcome01!.`

9\. Click through next during the installation steps and I would recommend choosing the most privacy friendly settings.

* For Windows 10 select "Domain Join instead" to create a local user account

### Updating

1. Click Start and select the gear Settings Icon.

![](<../../../.gitbook/assets/image (27) (1).png>)

2\. The Windows Settings menu appears, click on "Update & Security"

![](<../../../.gitbook/assets/image (26) (1).png>)

3\. Click on "Check for Updates" and install all updates.

![](<../../../.gitbook/assets/image (49) (1).png>)

4\. When the system asks for a reboot, reboot is and repeat till there are no more updates left.

Repeat these steps for each Image we would like to create (Windows Server 2019 and 2022, and Windows 10 and Windows 11)

## Optionally

{% content-ref url="optional-install-software-and-settings.md" %}
[optional-install-software-and-settings.md](optional-install-software-and-settings.md)
{% endcontent-ref %}

## Cleanup

1. Make sure the downloads folder is empty.

![](<../../../.gitbook/assets/image (58) (1).png>)

1. Empty the recycle bin by rightcliking on it and selecting "Empty Recycle Bin"

![](<../../../.gitbook/assets/image (44).png>)

1. Open explorer and click on "This PC", right click the C:\ disk and click "Properties".

![](<../../../.gitbook/assets/image (10) (1) (1) (2).png>)

4\. Click on "Disk Cleanup", "Clean System Files", select everything and Click "OK"

![](<../../../.gitbook/assets/image (34) (1).png>)

![](<../../../.gitbook/assets/image (62) (1) (1) (1).png>)

## Sysprep

To be able to clone the virtual machine without having any problems when joining the same machine to the domain, we have to run sysprep.

1. Press Windows + R, type sysprep and click OK.

![](<../../../.gitbook/assets/image (19) (1) (1).png>)

2\. Windows Explorer opens and click on the "Sysprep" application.

3\. Select "Generalize", Choose Shutdown and click on "OK"

![](<../../../.gitbook/assets/image (18) (1).png>)

## Finished

When the machine is fully shutdown the image is finished.
