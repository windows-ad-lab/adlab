# Database-Links

## Configuring

### Configuring SQL user on Data01

1. Login to `DATA01` as the `Administrator` user with password `Welcome01!`.
2. Open "Microsoft SQL Server Management Studio".

![](<../../../.gitbook/assets/image (17).png>)

3\.  Login with the `Administrator` user using Windows Authentication.

![](<../../../.gitbook/assets/image (29).png>)

4\. Right click on the server object "Data01\Data (SQL Server 15.0.2000.5)" and select "Properties".

5\. Open the "Security" tab and select "SQL Server and Windows Authentication mode" under "Server authentication".

![](<../../../.gitbook/assets/image (52) (1).png>)

6\. Open "SQL Server 2019 Configuration Manager"&#x20;

![](<../../../.gitbook/assets/image (6).png>)

7\. Restart the two SQL services with "DATA" in their name.

![](<../../../.gitbook/assets/image (32).png>)

8\. Open "Microsoft SQL Server Management Studio".

9\. Expand "Security" and "Logins". Then right click on "Logins" and select "New Login".

10\. Fill in the login name `SQL_link` and password `Eo6#jAzonQhw` . Then make sure the three password flags "Enforce Password policy", "Enforce password expiration" and "User must change password at next login" are unselected.

![](<../../../.gitbook/assets/image (65).png>)

11\. In the tab "Server Roles" select "setupadmin" and "sysadmin" and click "OK".

![](<../../../.gitbook/assets/image (52).png>)

12\. In the tab "User Mapping" select "Bank Transfers".

![](<../../../.gitbook/assets/image (27).png>)

### Configuring SQL link

1. Login to `WEB01` as the `Administrator` user with password `Welcome01!`.
2. Open "Microsoft SQL Server Management Studio".
3. Login with the `Administrator` user using Windows Authentication.
4. Expand the "Server Objects" and "Linked Servers". Then right click on "Linked Servers" and select "New Linked Server".
5. Fill in `Data01.secure.local` and select "SQL Server".

![](<../../../.gitbook/assets/image (10).png>)

6\. Open the "Security" tab and select "Be made using this security context". Then fill in the credentials we created in the previous section. `SQL_link` and password `Eo6#jAzonQhw` .&#x20;

![](<../../../.gitbook/assets/image (68).png>)

7\. Open the "Server Options" tab and select True for "RPC Out". This is done so we can enable xp\_cmdshell during the attack.

![](<../../../.gitbook/assets/image (14).png>)

8\. Click "OK" and the linked server should show up.

![](<../../../.gitbook/assets/image (64).png>)

## Attacking

### How it works



### Tools



### Executing the attack



## Defending

### Recommendations



### Detection



## References
