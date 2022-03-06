# SQL Server

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Download SQL Server 2019 from [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-sql-server-2019).

![](<../../../.gitbook/assets/image (9) (1).png>)

3\. When smartscreen comes up select "Run".

![](<../../../.gitbook/assets/image (64) (1).png>)

{% hint style="info" %}
The SQL Server Installation may require internet access, temporally add a second adapter in the machine settings and select the NAT network. The machine should have internet access. (ps our first adapter is different since our lab runs on a ESXI host).

![](<../../../.gitbook/assets/image (60) (1).png>)


{% endhint %}

4\. On "Select an installation type" select "Basic":

![](<../../../.gitbook/assets/image (29) (1).png>)

5\. Accept the License Terms and click "Install".

![](<../../../.gitbook/assets/image (31) (1).png>)

6\. Once the installation is finished, click on "Customize".

![](<../../../.gitbook/assets/image (21).png>)

7\. Check "Use Microsoft Update to check for updates" and click next till the step "License Terms" and accept them:

![](<../../../.gitbook/assets/image (22).png>)

8\. At the step "Feature Selection" choose the following options:

* Database ENgine Services
* SQL Server Replication
* Client Tools Connectivity
* Client Tools SDK

![](<../../../.gitbook/assets/image (51) (1).png>)

![](<../../../.gitbook/assets/image (26).png>)

9\. Click Next and fill in the Instance name `dev`.

![](<../../../.gitbook/assets/image (43) (1).png>)

10\. Change the "Startup Type" for the "SQL Serer Agent" to "Automatic" and make sure all three are set to "Automatic", click "Next".

![](<../../../.gitbook/assets/image (30).png>)

11\. At the next step select "Mixed Mode" and fill in the password `Password1!`. Then select "Add" at the "Specify SQL Server administrators" and add `amsterdam\administrator` then click Next.

![](<../../../.gitbook/assets/image (4) (1).png>)

12\. Click "Install" and wait for the installation to finish.

![](<../../../.gitbook/assets/image (42) (1).png>)

13\. Open the "SQL Configuration Manager".

![](<../../../.gitbook/assets/image (9).png>)

13\. Expand "SQL Server Network Configuration" and select "Protocols for DEV". Double click on it and select the "IP Adresses" tab. Fill in `1433` for "TCP Port".

![](<../../../.gitbook/assets/image (29).png>)

14\. Scroll to the bottom and also configure IPAll:

![](<../../../.gitbook/assets/image (51).png>)

14\. Then click "Apply".

![](<../../../.gitbook/assets/image (36).png>)

15\. Open "SQL Server services" and right click on "SQL Server (DEV)" and click "Restart".

![](<../../../.gitbook/assets/image (47) (1).png>)

![](<../../../.gitbook/assets/image (52).png>)

16\. Do the same for "SQL Server Agent":

![](<../../../.gitbook/assets/image (46).png>)

17\. Download and Install [SQL Server Mangement Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?redirectedfrom=MSDN\&view=sql-server-ver15). Just Click "Install" and reboot the system.



### Configuring Windows Firewall for SQL Server

1. Search for "Windows Defender Firewall with Advanced Security" and select "Windows Defender Firewall with Advanced Security"

![](<../../../.gitbook/assets/image (62).png>)

2\. Rightclick on "Inbound Rules" and select "New rule".

3\. Select "Port" and click "Next"

![](<../../../.gitbook/assets/image (31).png>)

4\. Make sure "TCP" is selected and fill in port `1433` and click "Next":

![](<../../../.gitbook/assets/image (58).png>)

5\. Click next at the step "Profile". Fill in the name "Allow TCP 1433 MSSQL Server for all" and click "Finish":

![](<../../../.gitbook/assets/image (32) (1).png>)

### Testing connectivity

1\. Open the Kali machine and run a quick Nmap to check if the port is open:

```
sudo nmap -p 1433 10.0.0.5 -Pn -n

Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-06 11:39 CET
Nmap scan report for 10.0.0.5
Host is up (0.017s latency).

PORT     STATE SERVICE
1433/tcp open  ms-sql-s

Nmap done: 1 IP address (1 host up) scanned in 0.12 seconds
```





