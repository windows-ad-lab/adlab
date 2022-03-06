# SQL Server

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Download SQL Server 2019 from [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-sql-server-2019).

![](<../../../.gitbook/assets/image (9).png>)

3\. When smartscreen comes up select "Run".

![](<../../../.gitbook/assets/image (64).png>)

{% hint style="info" %}
The SQL Server Installation may require internet access, temporally add a second adapter in the machine settings and select the NAT network. The machine should have internet access. (ps our first adapter is different since our lab runs on a ESXI host).

![](<../../../.gitbook/assets/image (60).png>)


{% endhint %}

4\. On "Select an installation type" select "Basic":

![](<../../../.gitbook/assets/image (29).png>)

5\. Accept the License Terms and click "Install".

![](<../../../.gitbook/assets/image (31).png>)

6\. Once the installation is finished, click on "Customize".

![](<../../../.gitbook/assets/image (21).png>)

7\. Check "Use Microsoft Update to check for updates" and click next till the step "License Terms" and accept them:

![](<../../../.gitbook/assets/image (22).png>)

8\. At the step "Feature Selection" choose the following options:

* Database ENgine Services
* SQL Server Replication
* Client Tools Connectivity
* Client Tools SDK

![](<../../../.gitbook/assets/image (51).png>)

![](<../../../.gitbook/assets/image (26).png>)

9\. Click Next and fill in the Instance name `dev`.

![](<../../../.gitbook/assets/image (43).png>)

10\. Change the "Startup Type" for the "SQL Serer Agent" to "Automatic" and make sure all three are set to "Automatic", click "Next".

![](<../../../.gitbook/assets/image (30).png>)

11\. At the next step select "Mixed Mode" and fill in the password `Password1!`. Then select "Add" at the "Specify SQL Server administrators" and add `amsterdam\administrator` then click Next.

![](<../../../.gitbook/assets/image (4).png>)

12\. Click "Install" and wait for the installation to finish.

13\. Download [SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15).





