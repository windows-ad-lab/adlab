---
description: >-
  Shares are used a lot in infrastructures and we often see shares that are open
  for all domain users. Sometimes passwords or other sensitive information can
  be found on shares.
---

# File services

## Adding the fileserver role

1. Click on start and open the "Server Manager".

![](<../../../.gitbook/assets/afbeelding (19).png>)

2\. On the right top click on "Manage" and "Add Roles and Features".

![](<../../../.gitbook/assets/afbeelding (9) (2).png>)

3\. Check the "Skip this page by default" box and click "Next" three times (For the steps "Before You Begin", "Installation Type" and "Server Selection".

4\. At the step "Server Roles" expand "File and Storage Services" and "File and ISCSI Services". Then select "File Server".

![](<../../../.gitbook/assets/afbeelding (38) (1).png>)

5\. Click "Next" and then "Install". Wait for the installation to finish.

## Creating a share

1. In the Server Manager click on "File and Storage Services"

![](<../../../.gitbook/assets/afbeelding (4) (2).png>)

2\. Click on "Shares" and then "to create a file share, start the new share wizard"

![](<../../../.gitbook/assets/afbeelding (6) (1) (2).png>)

3\. Make sure "SMB Share - Quick" is selected and click "Next".

![](<../../../.gitbook/assets/afbeelding (2).png>)

4\. Open Windows Explorer and go to the `C:\` disk and create a directory named `Share`.

![](<../../../.gitbook/assets/afbeelding (36) (1).png>)

5\. In the installer select "Type a custom path" and type `C:\Share`. Then click "Next".

![](<../../../.gitbook/assets/afbeelding (1).png>)

6\. Type the share name: `Data` and click "Next".

![](<../../../.gitbook/assets/afbeelding (17).png>)

7\. At the steps "Other Settings", "Permissions" and click "Next", we will leave this default.

8\. Click "Create"

![](<../../../.gitbook/assets/afbeelding (20) (1).png>)

![](<../../../.gitbook/assets/afbeelding (25) (1).png>)

## Testing

1. Create a txt file in the `C:\Share` directory.

![](<../../../.gitbook/assets/afbeelding (3).png>)

2\. Login on `DC01` and access the share `\\file01\data` in Explorer. Check if you can access the share and if the txt file is there.

![](<../../../.gitbook/assets/afbeelding (26) (1).png>)

All domain users should be able to access the share.

