# Bypassing UAC

## Attacking

### How it works

Bypassing UAC is required if you got a shell as an user which has local administrator privileges but isn't running with high privileges. This is a low privileged shell. You want to run in a high privileged shell to migrate processes and dump the LSASS process or SAM. This can be achieved by a UAC bypass and spawning an high integrity shell.

For more information about UAC refer to the documentation from Microsoft: [https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works)

### Tools

* [UACME](https://github.com/hfiref0x/UACME)

### Building UACME

1. Download Visual Studio 2019 on a local W10 machine from [here](https://visualstudio.microsoft.com/vs/older-downloads/).
2. Clone the UACME project.

```
git clone https://github.com/hfiref0x/UACME
```

3\. Open the UACME project by clicking on `uacme.sln`.

![](<../../.gitbook/assets/image (7) (3).png>)

4\. In the "Solution Explorer" right click on "Akagi" and click on "Properties". Make sure the "Platform Toolset" is set to V142 and click on "Apply" and "OK".

![](<../../.gitbook/assets/image (6).png>)

5\. At the top select the "Release" version and "X64" bit.

![](<../../.gitbook/assets/image (12).png>)

6\. In the "Solution Explorer" right click on "solution aucme" and click on "Build".

![](<../../.gitbook/assets/image (15).png>)

7\. The "Output" pane should show that 5 builds are succeeded.

![](../../.gitbook/assets/image.png)

8\. Go to `C:\Tools\Evasion\UACME\Source\Akagi\output\x64\Release` and copy `Akagi64.exe` to the desktop.

### Executing the attack

1\. The syntax for the executable is `./Akagi64.exe <METHOD> <EXECUTABLE>`. If we run the following command we will abuse the fodhelper method:

![](<../../.gitbook/assets/image (41).png>)

```
.\Akagi64.exe 34 cmd.exe
```

![](<../../.gitbook/assets/image (5) (3).png>)

2\. And an elevatged prompt started.

## Defending

### Recommendations

* Change the UAC level to always prompt for passwords.
* Keep the Windows version up-to-date, although a lot of UAC bypasses will still work! They get patched from time to time.

### Detection



## References

{% embed url="https://www.youtube.com/watch?v=RXX0FHM9SEk" %}

{% embed url="https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works" %}
