# Configure LDAPS

1. Search for "Certificate Authority" in the Windows Search Function and Right click on it and select "Run as different user". Then fill in the Enterprise Admin credentials from `bank\administrator` and the password `Welcome01!`.

![](<../../../../../.gitbook/assets/image (6).png>)

2\. Open the directories and click on "Certificate Templates" and select "Manage".

![](<../../../../../.gitbook/assets/image (14).png>)

3\. Look for the template "Kerberos Auithentication" and select "Duplicate Template".

![](<../../../../../.gitbook/assets/image (17).png>)

4\. Open the tab "General" and give the template name `LDAPS` and select "Public certificate Active Directory".

![](<../../../../../.gitbook/assets/image (9).png>)

5\. Open the tab "Request Handling" and select "Allow private key to be exported".

![](<../../../../../.gitbook/assets/image (16).png>)

6\. Open the tab "Subject Name" and select "User Principal Name" and "Serviec Principal Name".

![](<../../../../../.gitbook/assets/image (11).png>)

7\. Click on "Apply" and "OK" and the template should be created.

![](<../../../../../.gitbook/assets/image (42).png>)

8\. Return to "certsrv" window and right click "Certificate Templates" and select "New" and then "Certificate Template to Issue".

![](<../../../../../.gitbook/assets/image (82).png>)

9\. Locate the LDAPS template and select it, click "OK".

![](<../../../../../.gitbook/assets/image (83).png>)

### Generate SSL Cert

1. Search for mmc.exe and open it.

![](<../../../../../.gitbook/assets/image (64).png>)

2\. At the top click on "File" and then "Add/Remove Snap-in" .

![](<../../../../../.gitbook/assets/image (54).png>)

3\. Select "Certificates" and click "Add".

![](<../../../../../.gitbook/assets/image (70).png>)

4\. Select "Computer account"

![](<../../../../../.gitbook/assets/image (53).png>)

5\. Select "local computer" and click "Finish".

6\. Click on "Certificates" again and on "Add". This time select "Service account"

![](<../../../../../.gitbook/assets/image (84).png>)

7\. Select "Local Computer" again and click "Next". Then select "Active Directory Domain Services" and click "Finish". There should be two Certificate snap-ins now:

![](<../../../../../.gitbook/assets/image (65).png>)

8\. In mmc.exe open the "Certificates (Local Computer)" directory --> Personal and right click on "Certificates" and select "All tasks" and click on "Request new certificate".

![](<../../../../../.gitbook/assets/image (86).png>)

9\. Click on "Next" and select "LDAPS" and click on "Enroll".

![](<../../../../../.gitbook/assets/image (58).png>)

10\. Create a new directory in the `C:\` disk with the following powershell command:

```
New-Item -Path C:\ -Name Certs -ItemType Directory
```

11\. Execute the following command to list all Certs and filter for specific information. Then note down the Thumbprint ID of the cert which has KDC authentication etc.:

```
Get-ChildItem Cert:\LocalMachine\My\ | Select-Object ThumbPrint, Subject, NotAfter, EnhancedKeyUsageList
```

![](<../../../../../.gitbook/assets/image (85).png>)

12\. Export the certificate with a password using the following commands and the Thumbprint:

```
$password = ConvertTo-SecureString -String "123456" -Force -AsPlainText
Get-ChildItem -Path Cert:\LocalMachine\My\B152D39E3CC245E22629C58FF6993FE9F47FD05C | Export-PfxCertificate -FilePath C:\Certs\LDAPs.pfx -Password $password
```

![](<../../../../../.gitbook/assets/image (34).png>)

![](<../../../../../.gitbook/assets/image (72).png>)

13\. In mmc.exe the new certificate will be listed here too. You can open it and check in the "details" and "Thumbprint" to check it:

![](<../../../../../.gitbook/assets/image (69).png>)

14\. Next copy the certificate from LocalMachine _Personal_ store to the _Active Directory Domain Services_ Service Account Certificate store under NTDS\Personal Certificates, using below command. Change the thumbprint accordingly:

```
Move-Item "HKLM:\SOFTWARE\Microsoft\SystemCertificates\MY\Certificates\B152D39E3CC245E22629C58FF6993FE9F47FD05C" "HKLM:\SOFTWARE\Microsoft\Cryptography\Services\NTDS\SystemCertificates\MY\Certificates\"
Get-ChildItem -Path "HKLM:\SOFTWARE\Microsoft\Cryptography\Services\NTDS\SystemCertificates\MY\Certificates\"
```

![](<../../../../../.gitbook/assets/image (88).png>)

15\. Open ldp.exe and test the connection to the domain controller:

![](../../../../../.gitbook/assets/image.png)

![](<../../../../../.gitbook/assets/image (1).png>)
