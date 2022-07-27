# Create a CA

1. Open the "Server Manager" and click on "Manage" and select "Add roles and Features".

![](<../../../../../.gitbook/assets/image (15).png>)

2\. In the step "Installation type" select the default "Role-based or feature-based installation" and click "Next". Then in "Server Selection" click next.

3\. In the step "Server Roles" select "Active Directory Certificate Services" and click "Add Features".

![](<../../../../../.gitbook/assets/image (28).png>)

4\. Click next till the step "Role Services" and select "Certificate authority" and "Certificate Authority Web Enrollment" and select "Add Features".

![](<../../../../../.gitbook/assets/image (3).png>)

5\. Click "Next" and finish the installation.

![](<../../../../../.gitbook/assets/image (1).png>)

6\. Click on the "Flag" in the "Server Manager" it should have an exclamation mark and click the "Configure Active Directory Certificate Services on the...."

![](<../../../../../.gitbook/assets/image (20).png>)

![](<../../../../../.gitbook/assets/image (21).png>)

7\. In the step "Credentials" fill in the credentials for the `bank\Administrator` user. The password should be `Welcome01!`.

![](<../../../../../.gitbook/assets/image (18).png>)

8\. Click on "Next" and in the step "Role Services" select "Certificate Authority" and "Certificate Authority Web Enrollment" and click "Next".

![](../../../../../.gitbook/assets/image.png)

8\. In the step "Setup Type" select the default "Enterprise CA" and click "Next".

9\. In the step "CA Type" select the default "Root CA" and click "Next".

10\. In the step "Private Key" select "Create a new private key" and click "Next". In the step "Cryptography" make sure sha256 is selected and key length 2048 and click "Next".

![](<../../../../../.gitbook/assets/image (12).png>)

11\. In the step "CA Name" fill in the name `amsterdambank-ca` and click "Next".

![](<../../../../../.gitbook/assets/image (5).png>)

12\. In the step "Validity Period" click "Next". Then click "Next" and click "Configure".
