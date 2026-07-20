If you don't have a developer headset, you can install ALVR from the Meta Store.  
  
[![ALVR on Meta Store](image.png)](https://www.meta.com/en-gb/experiences/alvr/7674846229245715/?srsltid=AfmBOopPVGRvDa_7YtnrZf8v6XsKe9FqQ4F1kp_R3hVU9uwIeKFgl0Ab)  
  
https://www.meta.com/en-gb/experiences/alvr/7674846229245715/  
  
However, if you are using a Lynx XR, Android XR, or any alternative to the Quest 3, you will need to install the APK manually.  
  
## Prerequisites  
  
> These steps are not explained here because they are standard. There are many tutorials on YouTube, and if you are attending my workshop, we already completed them together.  
  
- Create a Meta Developer Account.  
- Enable Developer Mode on your Quest.  
- Connect your Quest via USB and authorize the connection when the popup appears.  
  
## Install Scrcpy  
  
Go to the Scrcpy GitHub repository.  
  
[![Scrcpy GitHub](image-1.png)](https://github.com/Genymobile/scrcpy)  
  
https://github.com/Genymobile/scrcpy  
  
Open the **Releases** page.  
  
![Scrcpy Releases](image-2.png)  
  
Download the Windows 64-bit version.  
  
![Download Scrcpy](image-3.png)  
  
I extracted it to my `C:` drive:  
  
`C:\Exe\scrcpy`  
  
![Scrcpy Folder](image-4.png)  
  
As a bonus, I also copied my toolbox into the same folder:  
  
https://github.com/EloiStree/2025_01_12_pyhton_build_run_apk_broadcaster  
  
```bash
git clone https://github.com/EloiStree/2025_01_12_pyhton_build_run_apk_broadcaster.git toolbox
```
  
It contains several `scrcpy` and `adb` commands to communicate with Android phones and Quest headsets.  
  
For example, open a Command Prompt and run:  
  
`adb devices`  
  
![ADB Devices](image-5.png)  
  
This displays the connected Android devices.  
  
You can also request more detailed information by specifying the device with `-s` and using `-l` to list additional details.  
  
```bash
adb -s 292918750041 devices -l
```
  
![ADB Device Details](image-6.png)  
  
If only one Android device is connected, you can simply double-click `scrcpy.exe`.  
  
![Launch Scrcpy](image-7.png)  
  
You should see your phone's screen.  
  
![Phone Screen](image-8.png)  
  
## Download the APK to Install  
  
Download ALVR from GitHub:  
  
https://github.com/alvr-org/ALVR/releases/tag/v20.14.1  
  
Download the APK.  
  
![Download ALVR APK](image-9.png)  
  
Then drag and drop it onto your Quest 3 screen in Scrcpy.  
(Here I'm using my phone because my Quest wasn't nearby.)  
  
![Install APK](image-10.png)  
  
Do the same for F-Droid and MRTK/XRTK/VRTK.  
  
[![F-Droid](image-12.png)](https://f-droid.org/en/)  
  
https://f-droid.org/en/  
  
[![Workshop Releases](image-11.png)](https://github.com/EloiStree/2024_07_16_workshop_mons_xr_design/releases/tag/V0)  
  
https://github.com/EloiStree/2024_07_16_workshop_mons_xr_design/releases/tag/V0  
  
ALVR can work over a wired connection, while Steam Link only works over a local Wi-Fi network.  
  
You can also connect your headset to your Wi-Fi hotspot if needed.  
  
![Wi-Fi Hotspot](image-13.png)  
  
As a reminder:  
  
![Reminder](image-14.png)  
  
Don't forget to set an easy password for your VR hotspot and disable power saving, otherwise the hotspot may turn off every 10 minutes.  
  
![Power Saving](image-15.png)  
  
## Launch a Manually Installed APK  
  
If you installed an APK manually, open **Applications**.  
  
![Applications](image-16.png)  
  
Then open **Unknown Sources**.  
  
![Unknown Sources](image-17.png)  
  
Find **ALVR**, **F-Droid**, or any other manually installed application.  
  
Note that most standard Android APKs also run on the Quest.  
  
For example, this thermal camera application:  
  
![Thermal Camera 1](image-18.png)  
  
![Thermal Camera 2](image-19.png)  
  
Once you launch ALVR, you should see this menu.  
  
![ALVR Menu](image-20.png)  
  
Select **Wired Connection** and trust the device for future connections.  
  
![Trusted Device](image-21.png)  
  
Launch SteamVR if it was not started automatically by ALVR.  
  
![SteamVR](image-22.png)  
  
You can also request to see the headset's view.  
  
![Headset View](image-23.png)  
  
If you've reached this point, congratulations! You have successfully installed ALVR on Windows using Scrcpy, and you're ready for some SteamVR adventures with OpenXR on your device. 😜  
  
Note that SteamVR also supports hand tracking.  
  
![Hand Tracking](image-24.png)  