---
description: >-
  Optionally we could install some software such as BGInfo from sysinternals,
  Notepad++ or VMware tools.
---

# Optional: Install software & Settings

## BGInfo sysinternals

Show general info on the background:

![](<../../../.gitbook/assets/image (22) (1) (1) (1) (1) (1).png>)

1. Download [BGinfo](https://docs.microsoft.com/en-us/sysinternals/downloads/bginfo) from the Sysinternals Suite from Microsoft in the VM.
2. Go to the Download folder and unzip the BGInfo zip.
3. Copy the folder to the C:\ drive and empty the Downloads folder.
4. Run the Bginfo64 program and Agree to the License Terms.

![](<../../../.gitbook/assets/image (48).png>)

5\. Click on "File" --> Save as and select "All files". Then save the file in `C:\BGInfo` with the name `bginfo.bgi`

6\. Click on "Preview", "Apply" and "OK"

7\. Create a new txt file in `C:\BGInfo` with the name `bginfo.txt` and enter the following lines:

```batch
@echo off 
cd 
CALL “C:\BGInfo\Bginfo64.exe” “C:\BGInfo\bginfo.bgi” /timer:0 /nolicprompt
```

![](<../../../.gitbook/assets/image (66) (1) (1).png>)

8\. Click on "File" --> Save as --> Select "All Files" and save the file as `bginfo.bat` in `C:\BGInfo\`

![](<../../../.gitbook/assets/image (20) (1) (1) (1).png>)

![](<../../../.gitbook/assets/image (50).png>)

9\. Open explorer, click on "View" and select "Hidden items"

![](<../../../.gitbook/assets/image (30) (1).png>)

10\. Copy the `bginfo.bat` file and browse to `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\` and paste the `bginfo.bat` file

![](<../../../.gitbook/assets/image (29) (1) (1).png>)

## VMware tools

Vmware tools is required to automatically adapt the size to the monitor and for copy & pasting etc.

1. Right click the Virtual machine and select "Install VMWare tools".

![](<../../../.gitbook/assets/afbeelding (87).png>)

2\. A DVD Drive should show up, click on it and run setup64.exe.

![](<../../../.gitbook/assets/afbeelding (91).png>)

3\. Click next till you can click Install.

![](<../../../.gitbook/assets/afbeelding (70).png>)

## Desktop cleanup

**Will reset after cloning a VM, so needs to be redone for each user/vm.**

1. Right click the Taskbar and deselect "Show Task View button"
2. Right click the Taskbar --> Search and click "Hidden"
3. Right click the Taskbar and deselect "Show Cortana button"
4. Right click the Taskbar --> News and interest and select "Turn off"
5. Right click the Windows Store button and select "Unpin from taskbar"
6. Right Click the Mail button and select "Unpin from taskbar"

### Notepadd++

1. Download [Notepad++](https://download.sysinternals.com/files/BGInfo.zip), run the installer and click next next next.

![](<../../../.gitbook/assets/image (64) (1) (1) (1) (1) (1) (1).png>)
