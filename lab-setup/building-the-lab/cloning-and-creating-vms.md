# Cloning & Creating VM's

## Cloning Virtual Machines from images

1. Go to the directory where the virtual machines are installed `C:\Users\<USER>\Documents\Virtual Machines\`
2. Make a copy of the `Windows Server 2019 - Image` directory and rename it to `DC01`.
3. Make a copy of the `Windows Server 2022 - Image` directory and rename it to `DC02`.
4. Make a copy of the `Windows Server 2022 - Image` directory and rename it to `FILE01`.
5. Make a copy of the `Windows 10 - Image` directory and rename it to `WS01`.
6. Make a copy of the `Windows Server 2019 - Image` directory and rename it to `WEB01`.
7. Make a copy of the `Windows Server 2022 - Image` directory and rename it do `DC03`.
8. Make a copy of the `Windows Server 2022 - Image` directory and rename it do `DATA01`.

## Creating Virtual Machines

1. Open VMware and click on "Open a Virtual Machine"

![](<../../.gitbook/assets/afbeelding (63).png>)

2\. Browse to the virtual machine directory and select the `.vmx` file.

3\. Rename the Virtual Machine in VMware to the correspondant name (DC01, DC02, FILE01, WS01 or WEB01)

4\. Right click on the Virtual Machine --> Settings, change the network adapter to vmnet2 and click on "OK".

![](<../../.gitbook/assets/afbeelding (90).png>)

5\. Repeat this for all machines.

You should have the following machines: DC01, DC02, DC03, FILE01, WEB01, WS01, DATA01 and three images.

![](<../../.gitbook/assets/afbeelding (29) (1).png>)
