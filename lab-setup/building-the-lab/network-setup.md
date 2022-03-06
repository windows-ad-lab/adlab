# Network setup

For the lab environment we will create a Host only network. This is so that the hosts in this network are unable to connect through the internet. Only the host or other virtual machines inside this network will be able to interact with the lab. Incase internet access is required, the machine could temporarely be connected to a NAT network.

## General network configuration:

\
Adapter: Host-only\
Internet access: No\
Subnet: 10.0.0.0\
Subnetmask: 255.255.255.0\
DHCP: Disabled

## Creating a host-only network

1. Open VMware workstation and click on "Edit" and then "Virtual Network Editor"

![](<../../.gitbook/assets/image (63).png>)

2\. Click on "Change Settings" and accept the UAC prompt.

3\. Click on "Add Network" and Click "OK"

![](<../../.gitbook/assets/image (9) (1) (1).png>)

4\. Select the newly created network and copy the following settings:

* Host-Only
* Deselect "Use local DHCP service to distribute IP address to VM"
* Subnet IP: 10.0.0.0
* Subnet Mask: 255.255.255.0

![](<../../.gitbook/assets/image (57).png>)

5\. Click "OK"

**From now on every machine that we clone should be setup on the VMnet2 network.**
