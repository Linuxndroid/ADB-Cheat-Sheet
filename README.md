
<div align="center">
  <h1>Android™ Debug Bridge (adb)</h1>
  <h2>The Swiss Army Knife for Android</h2>
  <h3>Your Journey to Mastering <i>adb shell</i> Begins Here</h3>

</div>

<hr>



I have created customized scripts specifically for activities, and I am confident that you will find them useful. Here is an example script that specifically targets activities.

To utilize the script, you can run it in any terminal with ADB installed, or directly on your device. Simply navigate through the graphical user interface (GUI) on 
your phone and explore different options and GUIs. Whenever you perform an action that triggers an activity, the script will provide you with a complete ADB command. 

```bash
#!/usr/bin/env bash
# Author: wuseman

adb logcat | awk '                                                                                    
/act=android.intent.action.MAIN/ && /cmp=/ {
    match($0, /act=[^ ]+/);
    act="\033[95m" substr($0, RSTART+4, RLENGTH-4) "\033[0m";
    match($0, /cmp=[^ ]+/);
    cmp="\033[93m" substr($0, RSTART+4, RLENGTH-4) "\033[0m";
    print "adb shell am start -a '\''" act "'\'' -n '\''" cmp "'\''"
}'
```

Here is another one for swiping, this command retrieves touch event coordinates using getevent, and then uses awk to construct a touchscreen swipe command for input. Here's a description for it:

This command reads the output of adb shell getevent -l and searches for lines containing EV_ABS (absolute position event) and EV_SYN (event synchronization). It captures the X and Y coordinates of the touch event and constructs a touchscreen swipe command for input. The command then prints the generated touchscreen swipe command with the captured coordinates.

The `ABS_MT_POSITION_X` and `ABS_MT_POSITION_Y` values are stored in the event array. When a line with EV_ABS is encountered, the corresponding `X` and `Y` coordinates are captured. When a line with EV_SYN is encountered, it checks if both X and Y coordinates are available. If so, it constructs the touchscreen swipe command with the captured coordinates and a duration of 1000 milliseconds.

This command is useful for simulating touch events on an Android device using input commands:

```bash
#!/usr/bin/env bash
# Author: wuseman

adb shell getevent -l | awk '
    /EV_ABS/ {
        event[$3] = strtonum("0x" $NF)
    }
    /EV_SYN/ {
        if (event["ABS_MT_POSITION_X"] && event["ABS_MT_POSITION_Y"]) {
            printf "adb shell input touchscreen swipe %d %d %d %d 1000\n",
                event["ABS_MT_POSITION_X"], event["ABS_MT_POSITION_Y"],
                event["ABS_MT_POSITION_X"], event["ABS_MT_POSITION_Y"]
        }
    }
'
```
The final generated command will be in the format:

```bash
adb shell input touchscreen swipe <start_x> <start_y> <end_x> <end_y> 1000
```

### Highlight 'string' with Rainbow Colors
    
```bash
#!/usr/bin/env bash
# Author: wuseman

 adb logcat | awk '{
    gsub("string", "\033[1;31m\033[47mT\033[32m\033[33me\033[34ml\033[1;36mi\033[1;35ma\033[0m")
    print
}'
```

    
### Highlight Lines Containing 'Wifi' in Red color"
    
```bash
#!/usr/bin/env bash
# Author: wuseman

 adb logcat | awk -v color="\033[31m" '{
    gsub(/\033\[([0-9]{1,2}(;[0-9]{1,2})?)?[mGK]/, "")
     if (index($0, "Wifi") > 0 || index($0, "wifi") > 0) {
        gsub(/(Wifi|wifi)/, color "&" "\033[0m")
    }
    print
 }'
```

### Mark Matching Lines with Red Background

```bash
#!/usr/bin/env bash
# Author: wuseman

adb logcat | awk '
    {
        line = tolower($0)
        if (line ~ /<changemeToAStringToMatch>/) {
            printf "\033[41m%s\033[0m\n", $0
        } else {
            print
        }
    }
    `
```

### Colorize Words Individually



```bash
adb logcat | awk '{ for(i=1; i<=NF; i++) printf "\033[38;5;%dm%s\033[0m ", int(rand()*216)+16, $i; print "" }'
```

### Colorize Tags with Unique Colors

```bash
adb logcat | awk '{
    if ($6 in colorMap) {
        color = colorMap[$6]
    } else {
        color = colorCode
        colorCode = (colorCode + 1) % 216 + 16
        colorMap[$6] = color
    }
    gsub($6, "\033[38;5;" color "m&\033[0m")
} 1'
```

### Colorize Entire Lines with Unique Colors

```bash
adb logcat | awk 'BEGIN{
    colorCode = 16
} {
    if ($6 in colorMap) {
        color = colorMap[$6]
    } else {
        color = colorCode
        colorCode = (colorCode + 1) % 216 + 16
        colorMap[$6] = color
    }
    printf "\033[38;5;%dm%s\033[0m\n", color, $0
}'
```


### To dump all data from database files in the data folder using 8 parallel threads

* Resulting in more than 100% faster performance compared to using a single core, you can use the following sentence:

```bash
find /data/data -name '*db' -type f -print0 i|xargs -0 -n1 -P$(nproc) sh -c 'echo .dump | sqlite3 "$0"'
```



<details>
  <summary><h2>Android™ Source Code</h2></summary>
    <ul>
      <li><a href="https://cs.android.com/android/platform/superproject/">Android™ Source Code</a></li>
    </ul>
</details>







<details>
  <summary><h2>Android™ Issue Tracker</h2></summary>
    <ul>
      <li><a href="https://code.google.com/p/android/issues/entry">Android™ Issue Tracker</a></li>
    </ul>
</details>







<details>
  <summary><h2>Android™ SDK Tools</h2></summary>
    <ul>
      <li><a href="https://dl.google.com/android/repository/platform-tools_r32.0.0-linux.zip">Download SDK Platform-Tools for Linux</a></li>
      <li><a href="https://dl.google.com/android/repository/platform-tools_r32.0.0-darwin.zip">Download SDK Platform-Tools for MacOSX</a></li>
      <li><a href="https://dl.google.com/android/repository/platform-tools-latest-windows.zip">Download SDK Platform-Tools for Windows</a></li>
    </ul>
</details>









<details>
  <summary><h2>Android™ Tools</h2></summary>
  <ul>
    <li><a href="https://github.com/androguard/androguard/">Androguard - Disassemble DEX/ODEX bytecodes</a></li>
    <li><a href="https://github.com/SlackingVeteran/frija/releases/download/v1.4.4/Frija-v1.4.4.zip">Frija - Samsung firmware downloader</a></li>
  </ul>
</details>










<details>
  <summary><h2>Open Android™ Default Applications</h2></summary>

  <blockquote>
  <p><strong>Note:</strong></p>
  <p>If they are not available from GitHub, please visit: <a href="http://www.nr1.nu/android/applicationLauncher/">http://www.nr1.nu/android/applicationLauncher/</a> to get clickable URLs to launch the application.</p>
</blockquote>

  <ul>
    <li><a href="intent://com.sec.android.app.modemui.activities.USB.settings/#Intent;scheme=android-app;end">Click to Open - ADB Settings</a></li>
    <li><a href="intent://com.sec.android.app.popupcalculator/#Intent;scheme=android-app;end">Click to Open - Calculator</a></li>
    <li><a href="intent://com.sec.android.app.launcher/#Intent;scheme=android-app;end">Click to Open - Home Launcher</a></li>
    <li><a href="intent://com.google.android.gms/#Intent;scheme=promote_smartlock_scheme;end">Click to Open - Screen Smartlock</a></li>
    <li><a href="intent://com.android.settings/#Intent;scheme=android-app;end">Click to Open - Settings</a></li>
    <li><a href="intent://com.sec.android.app.servicemodeapp/#Intent;scheme=promote_USBSettings_scheme;end">Click to Open - USB Settings</a></li>
  </ul>
</details>










<details>
  <summary><h2>Open Android™ Applications</h2></summary>

  <blockquote>
  <p><strong>Note:</strong></p>
  <p>If they are not available from GitHub, please visit: <a href="http://www.nr1.nu/android/applicationLauncher/">http://www.nr1.nu/android/applicationLauncher/</a> to get clickable URLs to launch the application.</p>
</blockquote>

  <ul>
    <li><a href="https://galaxy.store/alliance">Click to Open - Alliance Shield X (Galaxy Store)</a></li>
    <li><a href="intent://com.rrivenllc.shieldx/#Intent;scheme=android-app;end">Click to Open - Alliance Shield X (Application)</a></li>
  </ul>
</details>














<details>
  <summary><h2>Open Android™ Samsung Applications</h2></summary>
<blockquote>
  <p><strong>Note:</strong></p>
  <p>If they are not available from GitHub, please visit: <a href="http://www.nr1.nu/android/applicationLauncher/">http://www.nr1.nu/android/applicationLauncher/</a> to get clickable URLs to launch the application.</p>
</blockquote>

  <ul>
    <li><a href="intent://com.sec.android.app.samsungapps/#Intent;scheme=android-app;end">Click to Open - Samsung Apps</a></li>
    <li><a href="intent://com.android.settings/com.samsung.android.settings.biometrics.fingerprint.FingerprintEntry/#Intent;scheme=android-app;end">Click to Open - Samsung Biometrics Fingerprint ID</a></li>
    <li><a href="intent://com.samsung.android.dialer/#Intent;scheme=android-app;end">Click to Open - Samsung Dialer Call</a></li>
    <li><a href="intent://com.sec.android.app.myfiles/#Intent;scheme=android-app;end">Click to Open - Samsung My Files</a></li>
    <li><a href="intent://com.samsung.knox.securefolder/#Intent;scheme=android-app;end">Click to Open - Samsung Knox Secure Folder</a></li>
    <li><a href="intent://com.sec.android.app.samsungapps/#Intent;scheme=android-app;end">Click to Open - Samsung Galaxy Store</a></li>
    <li><a href="intent://com.samsung.knox.securefolder/#Intent;scheme=android-app;end">Click to Open - Samsung Secure Folder</a></li>
    <li><a href="intent://com.sec.android.easyMover/#Intent;scheme=android-app;end">Click to Open - Samsung Smart Switch</a></li>
    <li><a href="intent://com.samsung.knox.securefolder/#Intent;scheme=android-app;end">Click to Open - Samsung Smart Switch</a></li>
    <li><a href="intent://com.android.settings/com.samsung.android.settings.biometrics.fingerprint.FingerprintEntry/#Intent;scheme=android-app;end">Click to Open - Samsung Touch ID</a></li>
  </ul>
</details>








<details>
  <summary><h2>Open Android™ Google Applications</h2></summary>

<blockquote>
  <p><strong>Note:</strong></p>
  <p>If they are not available from GitHub, please visit: <a href="http://www.nr1.nu/android/applicationLauncher/">http://www.nr1.nu/android/applicationLauncher/</a> to get clickable URLs to launch the application.</p>
</blockquote>

  <ul>
    <li><a href="intent://com.google.android.apps.googleassistant/#Intent;scheme=android-app;end">Click to Open - Google Assistant</a></li>
    <li><a href="intent://com.android.chrome/#Intent;scheme=android-app;end">Click to Open - Google Chrome Browser</a></li>
    <li><a href="intent://com.google.android.gm/#Intent;scheme=android-app;end">Click to Open - Google Gmail</a></li>
    <li><a href="intent://com.google.android.gsf.login.LoginActivity/#Intent;scheme=android-app;end">Click to Open - Google Login Account</a></li>
    <li><a href="intent://com.google.android.apps.maps/#Intent;scheme=android-app;end">Click to Open - Google Maps</a></li>
    <li><a href="intent://com.google.android.googlequicksearchbox/#Intent;scheme=android-app;end">Click to Open - Google Quicksearchbox</a></li>
    <li><a href="intent://com.google.android.youtube/#Intent;scheme=android-app;end">Click to Open - YouTube</a></li>
  </ul>
</details>











<details>
  <summary><h2>Open Android™ Motorola Applications</h2></summary>

  <blockquote>
  <p><strong>Note:</strong></p>
  <p>If they are not available from GitHub, please visit: <a href="http://www.nr1.nu/android/applicationLauncher/">http://www.nr1.nu/android/applicationLauncher/</a> to get clickable URLs to launch the application.</p>
</blockquote>

  <ul>
    <li><a href="intent://com.motorola.launcher3/#Intent;scheme=android-app;end">Click to Open - Motorola Default Launcher</a></li>
  </ul>
</details>










<details>
  <summary><h2>Open Android™ Xiaomi Applications</h2></summary>

  <blockquote>
  <p><strong>Note:</strong></p>
  <p>If they are not available from GitHub, please visit: <a href="http://www.nr1.nu/android/applicationLauncher/">http://www.nr1.nu/android/applicationLauncher/</a> to get clickable URLs to launch the application.</p>
</blockquote>

  <ul>
    <li><a href="intent://com.mi.android.globalFileexplorer/#Intent;scheme=android-app;end">Click to Open - Mi Manager</a></li>
  </ul>
</details>










<details>
  <summary><h2>Open Android™ Secret Codes</h2></summary>
  <blockquote>
  <p><strong>Note:</strong></p>
  <p>If they are not available from GitHub, please visit: <a href="http://www.nr1.nu/android/applicationLauncher/">http://www.nr1.nu/android/applicationLauncher/</a> to get clickable URLs to launch the application.</p>
</blockquote>
  <ul>
    <li><a href="tel:%20*#06*"># Click to Open USSD Code - Show IMEI - *#06*</a></li>
    <li><a href="tel:%20*#0*#"># Click to Open USSD Code - Show Secret Diagnostic Mode - *#0#*</a></li>
  </ul>
</details>








<details>
  <summary><h2>Secret Codes™ (Unstructured Supplementary Service Data codes))</h2></summary>
  
## Secret Codes™ <small>Samsung Latest Models</small>

| Secret Code          | Description      | ADB Activity                                                                                               |
|----------------------|----------------------------------|--------------------------------------------------------------------------------------------|
| `*#0*#`              | Diagnostic Test Menu             | com.sec.android.app.hwmoduletest/com.sec.android.app.hwmoduletest.HwModuleTest
| `*#06#`              | IMEI number                      | com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.ShowIMEI         |
| `*#0228#`            | Battery status                   | com.sec.android.app.factorykeystring/com.sec.android.app.status.BatteryStatus              |
| `*#0*#`              | HW Module Test                   | com.sec.android.app.hwmoduletest/com.sec.android.app.hwmoduletest.HwModuleTest             |
| `*#1234`             | Firmware info                    | com.sec.android.app.factorykeystring/com.sec.android.app.version.SimpleVersion             |
| `*#2222#`            | Service Mode                     | com.sec.android.RilServiceModeApp/com.sec.android.RilServiceModeApp.ServiceModeApp         |
| `*#2222#`            | WiFi Info                        | com.sec.android.app.servicemodeapp/com.sec.android.app.servicemodeapp.WifiInfoActivity     |
| `*#2663`             | Touch Firmware                   | com.sec.android.app.factorykeystring/com.sec.android.app.status.touch_firmware             |
| `*#0808`             | Samsung USB Settings             | com.sec.usbsettings/com.sec.usbsettings.USBSettings                                        |
| `*#2683662#`         | Access Samsung Service Mode      | com.sec.android.RilServiceModeApp/com.sec.android.RilServiceModeApp.ServiceModeApp         |
| `*#9090`             | Check Diagnostic Configuration   | com.sec.android.RilServiceModeApp/com.sec.android.RilServiceModeApp.ServiceModeApp |
| `*#9900`             | Access Samsung SysDump Mode      | com.sec.android.app.servicemodeapp/com.sec.android.app.servicemodeapp.SysDump
| `*#12580*369#`       | Software and Hardware Information| com.sec.android.app.factorykeystring/com.sec.android.app.version.MainVersion
| `*#0283*`            | Check the Audio Loopback Control | com.sec.android.app.factorykeystring/com.sec.android.app.status.LoopbackTestNew
| `*#34971539#`        | Check Camera Status and Firmware | com.sec.factory.camera/com.sec.android.app.camera.firmware.CameraFirmwareActivity
| `*#22558463#`        | Reset Total Call Time            | com.sec.android.app.servicemodeapp/com.sec.android.app.servicemodeapp.ResetTotalCallTime
| `*#1111#`            | Check FTA Software Version       | com.sec.android.RilServiceModeApp/com.sec.android.RilServiceModeApp.ServiceModeApp
| `**04*[old Pin]*[new Pin]*[new Pin]` |  Change SIM Card PIN | N/A |












### Secret USSD short codes for <small>AT&T</small>

| Code                      | Description
|---------------------------|----------------------------------------------------------------------------------------------------------------|
| `*61`                     | Block individual unwanted inbound calls                                                                         |
| `*80`                     | Turn off call blocking                                                                                          |
| `*78#`                    | Do Not Disturb mode (blocks all incoming calls). Callers will hear a busy signal                                |
| `*79#`                    | Turn off Do Not Disturb mode                                                                                    |
| `*67+Phone number+#`      | Blocks outgoing Caller ID on a per-call basis. Information still visible to toll-free numbers and 911           |
| `*370#`                   | Disable call waiting                                                                                            |
| `*371#`                   | Reactivate call waiting                                                                                         |

</details>



















<details>
  <summary><h2>Secret USSD short codes for AT&T</h2></summary>
  
  | Code | Description |
  |------|-------------|
  | `*61` | Block individual unwanted inbound calls |
  | `*80` | Turn off call blocking |
  | `*78#` | Do Not Disturb mode (blocks all incoming calls). Callers will hear a busy signal |
  | `*79#` | Turn off Do Not Disturb mode |
  | `*67+Phone number+#` | Blocks outgoing Caller ID on a per-call basis. Information still visible to toll-free numbers and 911 |
  | `*370#` | Disable call waiting |
  | `

*371#` | Reactivate call waiting |
</details>
















<details>
  <summary><h2>Android™ Default paths</h2></summary>
  
  | Paths | Description |
  |-------|-------------|
  | `/data/data/<package>/databases` | Default storage for `.db` files |
  | `/data/data/<package>/shared_prefs/` | Default storage for `shared` preferences |
  | `/data/app` | Default storage for `third-party` apps |
  | `/system/app` | Default storage for `system` apks |
  | `/mmt/asec` | Default storage for `encrypted` files |
  | `/mmt/emmc` | Default storage for `internal` storage |
  | `/mmt/sdcard` | Default storage for `external` mmc/sdcard |
</details>
























<details>
  <summary><h2>Android Versions</h2></summary>
  
  | Version | API/SDK | Version Code | Codename | Release Year |
  |---------|---------|--------------|----------|--------------|
  | `Android 13` | Level 33 | TIRAMISU | Tiramisu 2 | 2022 |
  | `Android 12` | Level 32 | S _V2 | Snow Cone 2 | 2022 |
  | `Android 12` | Level 31 | S | Snow Cone | 2021 |
  | `Android 11` | Level 30 | R | Red Velvet Cake | 2020 |
  | `Android 10` | Level 29 | Q | Quince Tart | 2019 |
  | `Android 9` | Level 28 | P | Pie | 2018 |
  | `Android 8.1` | Level 27 | O_MR1 | Oreo | 2017 |
  | `Android 8.0` | Level 26 | O | Oreo | 2017 |
  | `Android 7.1` | Level 25 | N_MR1 | Nougat | 2016 |
  | `Android 7.0` | Level 24 | N | Nougat | 2016 |
  | `Android 6` | Level 23 | M | Marshmallow | 2015 |
  | `Android 5.1` | Level 22 | LOLLIPOP_MR1 | LOLLIPOP | 2015 |
  | `Android 5.0` | Level 21 | LOLLIPOP, L | LOLLIPOP | 2014 |
  | `Android 4.4W` | Level 20 | KITKAT_WATCH | KitKat | 2014 |
  | `Android 4.4` | Level 19 | KITKAT | KitKat | 2013 |
  | `Android 4.3` | Level 18 | JELLY_BEAN_MR2 | Jelly Bean | 2012 |
  | `Android 4.2` | Level 17 | JELLY_BEAN_MR1 | Jelly Bean | 2012 |
  | `Android 4.1` | Level 16 | JELLY_BEAN | Jelly Bean | 2012 |
  | `Android 4.0.3 –> 4.0.4` | Level 15 | ICE_CREAM_SANDWICH_MR1 | Ice Cream Sandwich | 2011 |
  | `Android 4.0.1 –> 4.0.2` | Level 14 | ICE_CREAM_SANDWICH | Ice Cream Sandwich | 2011 |
  | `Android 3.2` | Level 13 | HONEYCOMB_MR2 Honeycomb | Honeycomb | 2011 |
  | `Android 3.1` | Level 12 | HONEYCOMB_MR1 | Honeycomb | 2011 |
  | `Android 3.0` | Level 11 | HONEYCOMB | Honeycomb | 2011 |
  | `Android 2.3.3 –> 2.3.7` | Level 10 | GINGERBREAD_MR1 | Gingerbread | 2011 |
  | `Android 2.3.0 –> 2.3.2` | Level 9 | GINGERBREAD | Gingerbread | 2010 |
  | `Android 2.2` | Level 8 | FROYO | Froyo | 2010 |
  | `Android 2.1` | Level 7 | ECLAIR_MR1 | Eclair | 2010 |
  | `Android 2.0.1` | Level 6 | ECLAIR_0_1 | Eclair | 2010 |
  | `Android 2.0` | Level 5 | ECLAIR | Eclair | 2010 |
  | `Android 1.6` | Level 4 | DONUT Donut | Donut | 2009 |
  | `Android 1.5` | Level 3 | CUPCAKE Cupcake | Cupcake | 2009 |
  | `Android 1.1` | Level 2 | BASE_1_1 | Petit Four | 2009 |
  | `Android 1.0` | Level 1 | BASE | N/A | 2008 |
</details>








































<details>
  <summary><h2>ADB Shell / Fastboot install</h2></summary>
  
  ### Arch Linux (pacman)
  
  ```bash
  pacman -S android-tools
  ```
  
  ### Debian Linux (apt)
  
  ```bash
  apt install adb fastboot -y    
  ```
  
  ### Gentoo Linux (portage)
  ```bash
  emerge --ask dev-util/android-sdk-update-manager dev-util/android-tools
  ```
  
  ### Fedora Linux (dnf)
  
  ```bash
  dnf install adb
  ```
  
  ### GNU/Linux (source)
  ```bash
  1) Download the Android SDK Platform Tools ZIP file for Linux.
  2) Extract the ZIP to an easily-accessible location (like the Desktop for example).
  3) Open a Terminal window.
  4) Enter the following command: cd /path/to/extracted/folder/
  5) This will change the directory to where you extracted the ADB files.
  6) So for example: cd /Users/Doug/Desktop/platform-tools/
  7) Connect your device to your Linux machine with your USB cable. 
     Change the connection mode to “file transfer (MTP)” mode. 
     This is not always necessary for every device, but it’s recommended so you don’t run into any issues.
  8) Once the Terminal is in the same folder your ADB tools are in, you can execute the following command to launch the ADB daemon: ./adb devices
  9) Back on your smartphone or tablet device, you’ll see a prompt asking you to allow USB debugging. Go ahead and grant it.
  ```
  
  ### Windows (source)
  
  ```bash
  1) Download: https://dl.google.com/android/repository/platform-tools-latest-windows.zip
  2) Extract the contents of this ZIP file into an

 easily accessible folder (such as C:\platform-tools)
  3) Open Windows explorer and browse to where you extracted the contents of this ZIP file
  4) Then open up a Command Prompt from the same directory as this ADB binary. This can be done by holding 
       Shift and Right-clicking within the folder then click the “Open command window here” option. 
  5) Connect your smartphone or tablet to your computer with a USB cable. 
        Change the USB mode to “file transfer (MTP)” mode. Some OEMs may or may not require this, 
        but it’s best to just leave it in this mode for general compatibility.
  6) In the Command Prompt window, enter the following command to launch the ADB daemon: adb devices
  7) On your phone’s screen, you should see a prompt to allow or deny USB Debugging access. Naturally, 
        you will want to grant USB Debugging access when prompted (and tap the always allow check box if you never want to see that prompt again).
  8) Finally, re-enter the command from step #6. If everything was successful,
        you should now see your device’s serial number in the command prompt (or the PowerShell window).
  ```
  
  </details>


<details>
  <summary><h2>Awesome Aliases For ADB</h2></summary>
  
  Copy and paste to add the below aliases in ~/.bashrc
  
  ```bash
  cat <<! >> ~/.bashrc

function startintent() {
  adb devices \
    |tail -n +2 \
    |cut -sf 1 \
    |xargs -I X adb -s X shell am start $(1)
}

function apkinstall() {
  adb devices \
    |tail -n +2 \
    |cut -sf 1 \
    |xargs -I X \
      adb -s X install -r $(1)
}

function rmapp() {
  adb devices \
    |tail -n +2 \
    |cut -sf 1 \
    |xargs -I X adb -s X uninstall $(1)
}

function clearapp() {
  adb devices \
    |tail -n +2 \
    |cut -sf 1 \
    |xargs -I X adb -s X shell cmd package clear $(1)
}
```
</details>
  














<details>
  <summary><h2>Simple for loop for dump global, secure, and system settings in one command</h2></summary>
  
```bash
for options in system security global; do 
  settings list ${options}; 
done
```
</details>





















<details>
  <summary><h2>Setup and connect to the device via WiFi</h2></summary>
  
This requires that the USB cable is connected until you connect. Once connected via USB, copy and paste the following:


```bash
#!/bin/bash
# Author: wuseman

port="5555"

interface=$(adb shell ip addr | awk '/state UP/ {print $2}' | sed 's/.$//'; )
ip=$(adb shell ifconfig ${interface} \
    |egrep  -o '(\<([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\>\.){3}\<([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\>' 2> /dev/null \
    |head -n 1)

adb tcpip ${port};sleep 0.5
adb connect $ip:${port}; sleep 1.0
adb devices; adb shell
```
</details>

<details>
  <summary><h2>Grab all activities that are available via am start</h2></summary>
  

















```bash
#!/bin/bash
# Author: wuseman

for package in $(cmd package list packages|cut -d: -f2); do 
    cmd package dump $package \
        |grep -i "activ" \
        |grep -Eo "^[[:space:]]+[0-9a-f]+[[:space:]]+.*/[^[:space:]]+" \
        |grep -oE "[^[:space:]]+$"; 
done > /tmp/full_activity_package_list.txt
```
</details>
























<details>
  <summary><h2>Enter Termux environment via ADB</h2></summary>
  
Source: [rewida17 - termux](https://gist.githubusercontent.com/rewida17/f8564bee5a196a8f51b98cd2e53813e4/raw/6c1fbf160438e351c79238e976ee8e3c4bf4742d/termux)

```bash
#!/system/xbin/bash
# Author: rewida17
# Modded by: wuseman

if [[ $(id -u) -ne "0" ]]; then 
  echo "root is required"; 
  exit
else
  export PREFIX='/data/data/com.termux/files/usr'
  export HOME='/data/data/com.termux/files/home'
  export LD_LIBRARY_PATH='/data/data/com.termux/files/usr/lib'
  export PATH="/data/data/com.termux/files/usr/bin:/data/data/com.termux/files/usr/bin/applets:$PATH"
  export LANG='en_US.UTF-8'
  export SHELL='/data/data/com.termux/files/usr/bin/bash'
  export BIN='/data/data/com.termux/files/usr/bin' 
  export TERM=vt220
  export AR="arm-linux-androideabi-ar"
  export CPP="arm-linux-androideabi-cpp"
  export GCC="arm-linux-androideabi-gcc"
  export LD="arm-linux-androideabi-ld"
  export

 NM="arm-linux-androideabi-nm"
  export OBJDUMP="arm-linux-androideabi-objdump"
  export RANLIB="arm-linux-androideabi-ranlib"
  export READELF="arm-linux-androideabi-readelf"
  export STRIP="arm-linux-androideabi-strip"
  export TERMUX="/data/data/com.termux/"

  resize
  cd "$HOME"
  exec "$SHELL" -l
fi
```
</details>

















<details>
  <summary><h2>Print esim via uiautomator</h2></summary>
  
Read the screen via uiautomator and print IMEI for all active esim/sim cards on the device.

```bash
#!/bin/bash
# Author: wuseman

### Launch IMEI (same result if you type in the caller app: *#06#) 
#### and dump screen to /tmp/read_screen.txt via uiautomator and parse esim IMEI:

adb shell input keyevent KEYCODE_CALL;
sleep 1;
input text '*#06#'; 
uiautomator dump --compressed /dev/stdout\
    |tr ' ' '\n'\
    |awk -F'"' '{print $2}'|grep "^[0-9]\{15\}$" \
    |nl -w 1 -s':'\
    |sed 's/^/IMEI/g'    
```
</details>

















<details>
  <summary><h2>Print Device screen state</h2></summary>
  
```bash
screenState="$(adb shell dumpsys input_method \
   |grep -i "mSystemReady=true mInteractive=" \
   |awk -F= '{print $3}')"

if [[ $screenState = "true" ]]; then 
    echo "Screen is on"; 
else 
    echo "Screen is off"; 
fi
```
</details>






















<details>
  <summary><h2>How reboot works on Android</h2></summary>

Reboot Commands:

- Reboot to Bootloader:
```bash
adb reboot bootloader
```

- Reboot to Fastboot:
```bash
adb reboot fastboot
```

- Reboot to Recovery:
```bash
adb reboot recovery
```

- Reboot to System:
```bash
adb reboot
```

What is a hot and warm reboot?

In order to answer your question, we need to define what a hot (or warm) reboot is on an Android device. Terms cold (or hard) boot and warm (or soft) boot are more associated with PCs, particularly Windows. For mobile phones or embedded devices, it's difficult to draw a clear line between cold and warm boot. In the case of a cold reboot, power is usually cut to CPUs, RAM, and even the whole motherboard. A soft reboot only kills and starts the processes while retaining power to hardware components. Power management is part of the open-source ACPI/UEFI/BIOS standard on PCs, while on phones, PMIC firmware is usually used with SoCs.

How does reboot really work on Android?

On (re)boot, SoC firmware loads bootloaders in memory which then load executable binaries and start processes (the actual OS). Android is based on the Linux kernel, which is the very first executable of the operating system that runs during the boot process. The kernel initializes necessary hardware and prepares a basic environment before executing `init`, the very first userspace process we can see. It's `init` that then starts and takes care of all services and processes.

A civilized way to reboot or shutdown is to let all processes terminate themselves, saving any pending work, unmounting filesystem

s, and then ask the kernel to reverse the boot process. `init` can handle this on modern OSes, or you can do it manually through the `/proc/sysrq-trigger` interface. Alternatively, we can ask the kernel to perform a quick reboot by killing everything. However, this may cause data loss, particularly due to filesystem corruption.

A brutal way is the long press of the power button (handled by PMIC), which is a cold reboot (or shutdown) in the true sense because the power to CPUs (and RAM) is suddenly cut without waiting for userspace processes and the kernel to terminate gracefully.

Does Android really perform a cold reboot?

On Android phones (and on other systems as well), a normal reboot is not completely cold as power is not cut, at least to RAM because it holds an area where kernel panic logs are stored, which can be accessed on the next boot (refer to ramoops used for last_kmsg or pstore). Similarly, some other memory regions allocated to SoC components and signed firmware that are isolated from the application processor (AP on which the main OS runs) may also not be erased. They include the Baseband Processor (modem), Digital Signal Processor (DSP), WiFi/BT module, etc.

However, a normal reboot isn't a warm reboot either. During reboot, the kernel kills itself and hands over control to bootloaders, which may boot the device in different possible modes (fastboot/bootloader, recovery, or normal boot). The low-level details are vendor and hardware-specific, whether a device performs a complete power-on reset (PoR) or if the hardware is not reset at all. Which components are powered down during different types of reboots depends on the interaction between the kernel, bootloader, SoC, PMIC, watchdog hardware, etc.

How to do a hot reboot?

The Linux kernel also supports another form of warm reboot: `kexec`. The kernel can terminate userspace processes and itself, executing a new kernel which can then start a new userspace environment without doing a hardware reset, POST, and re-initialization by BIOS. This approach is theoretically possible on Android too, i.e., the kernel re-executes itself with the proper command line and then starts `init`. However, it requires some device-specific changes to the kernel and ROM.

Stock Android doesn't provide a soft reboot functionality, but some custom ROMs implement this feature by triggering the restart method of the activity service. However, `init` itself and other core daemons like ueventd, vold, installd, surfaceflinger, logd, servicemanager, healthd, and a long list of vendor daemons aren't restarted.

To restart the device, you can use the following commands:

- On Android 9, the code for the restart method is `179`:
```bash
adb shell service call activity 179
```

- It's also possible to ask `init` to restart `zygote` and dependent services. However, SELinux won't let the property be set, so root access is required:
```bash
adb shell setprop ctl.restart zygote
```

</details>





<details>
  <summary><h2>ADB Shell / Fastboot install</h2></summary>
  
  ### Arch Linux (pacman)
  
  ```bash
  pacman -S android-tools
  ```
  
  ### Debian Linux (apt)
  
  ```bash
  apt install adb fastboot -y    
  ```
  
  ### Gentoo Linux (portage)
  ```bash
  emerge --ask dev-util/android-sdk-update-manager dev-util/android-tools
  ```
  
  ### Fedora Linux (dnf)
  
  ```bash
  dnf install adb
  ```
  
  ### GNU/Linux (source)
  ```bash
  1) Download the Android SDK Platform Tools ZIP file for Linux.
  2) Extract the ZIP to an easily-accessible location (like the Desktop for example).
  3) Open a Terminal window.
  4) Enter the following command: cd /path/to/extracted/folder/
  5) This will change the directory to where you extracted the ADB files.
  6) So for example: cd /Users/Doug/Desktop/platform-tools/
  7) Connect your device to your Linux machine with your USB cable. 
     Change the connection mode to “file transfer (MTP)” mode. 
     This is not always necessary for every device, but it’s recommended so you don’t run into any issues.
  8) Once the Terminal is in the same folder your ADB tools are in, you can execute the following command to launch the ADB daemon: ./adb devices
  9) Back on your smartphone or tablet device, you’ll see a prompt asking you to allow USB debugging. Go ahead and grant it.
  ```
  
  ### Windows (source)
  
  ```bash
  1) Download: https://dl.google.com/android/repository/platform-tools-latest-windows.zip
  2) Extract the contents of this ZIP file into an

 easily accessible folder (such as C:\platform-tools)
  3) Open Windows explorer and browse to where you extracted the contents of this ZIP file
  4) Then open up a Command Prompt from the same directory as this ADB binary. This can be done by holding 
       Shift and Right-clicking within the folder then click the “Open command window here” option. 
  5) Connect your smartphone or tablet to your computer with a USB cable. 
        Change the USB mode to “file transfer (MTP)” mode. Some OEMs may or may not require this, 
        but it’s best to just leave it in this mode for general compatibility.
  6) In the Command Prompt window, enter the following command to launch the ADB daemon: adb devices
  7) On your phone’s screen, you should see a prompt to allow or deny USB Debugging access. Naturally, 
        you will want to grant USB Debugging access when prompted (and tap the always allow check box if you never want to see that prompt again).
  8) Finally, re-enter the command from step #6. If everything was successful,
        you should now see your device’s serial number in the command prompt (or the PowerShell window).
  ```
  
  ***
  
  ### Awesome Aliases For ADB
  
  Copy and paste to add the below aliases in ~/.bashrc
  
  ```bash
  cat <<! >> ~/.bashrc

function startintent() {
  adb devices \
    |tail -n +2 \
    |cut -sf 1 \
    |xargs -I X adb -s X shell am start $(1)
}

function apkinstall() {
  adb devices \
    |tail -n +2 \
    |cut -sf 1 \
    |xargs -I X \
      adb -s X install -r $(1)
}

function rmapp() {
  adb devices \
    |tail -n +2 \
    |cut -sf 1 \
    |xargs -I X adb -s X uninstall $(1)
}

function clearapp() {
  adb devices \
    |tail -n +2 \
    |cut -sf 1 \
    |xargs -I X adb -s X shell cmd package clear $(1)
}
```
</details>
  

<details>
  <summary><h2>Simple for loop for dump global, secure, and system settings in one command</h2></summary>
  
```bash
for options in system security global; do 
  settings list ${options}; 
done
```
</details>

<details>
  <summary><h2>Setup and connect to the device via WiFi</h2></summary>
  
This requires that the USB cable is connected until you connect. Once connected via USB, copy and paste the following:

```bash
#!/bin/bash
# Author: wuseman

port="5555"

interface=$(adb shell ip addr | awk '/state UP/ {print $2}' | sed 's/.$//'; )
ip=$(adb shell ifconfig ${interface} \
    |egrep  -o '(\<([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\>\.){3}\<([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\>' 2> /dev/null \
    |head -n 1)

adb tcpip ${port};sleep 0.5
adb connect $ip:${port}; sleep 1.0
adb devices; adb shell
```
</details>

<details>
  <summary><h2>Grab all activities that are available via am start</h2></summary>
  
```bash
#!/bin/bash
# Author: wuseman

for package in $(cmd package list packages|cut -d: -f2); do 
    cmd package dump $package \
        |grep -i "activ" \
        |grep -Eo "^[[:space:]]+[0-9a-f]+[[:space:]]+.*/[^[:space:]]+" \
        |grep -oE "[^[:space:]]+$"; 
done > /tmp/full_activity_package_list.txt
```
</details>

<details>
  <summary><h2>Enter Termux environment via ADB</h2></summary>
  
Source: [rewida17 - termux](https://gist.githubusercontent.com/rewida17/f8564bee5a196a8f51b98cd2e53813e4/raw/6c1fbf160438e351c79238e976ee8e3c4bf4742d/termux)

```bash
#!/system/xbin/bash
# Author: rewida17
# Modded by: wuseman

if [[ $(id -u) -ne "0" ]]; then 
  echo "root is required"; 
  exit
else
  export PREFIX='/data/data/com.termux/files/usr'
  export HOME='/data/data/com.termux/files/home'
  export LD_LIBRARY_PATH='/data/data/com.termux/files/usr/lib'
  export PATH="/data/data/com.termux/files/usr/bin:/data/data/com.termux/files/usr/bin/applets:$PATH"
  export LANG='en_US.UTF-8'
  export SHELL='/data/data/com.termux/files/usr/bin/bash'
  export BIN='/data/data/com.termux/files/usr/bin' 
  export TERM=vt220
  export AR="arm-linux-androideabi-ar"
  export CPP="arm-linux-androideabi-cpp"
  export GCC="arm-linux-androideabi-gcc"
  export LD="arm-linux-androideabi-ld"
  export

 NM="arm-linux-androideabi-nm"
  export OBJDUMP="arm-linux-androideabi-objdump"
  export RANLIB="arm-linux-androideabi-ranlib"
  export READELF="arm-linux-androideabi-readelf"
  export STRIP="arm-linux-androideabi-strip"
  export TERMUX="/data/data/com.termux/"

  resize
  cd "$HOME"
  exec "$SHELL" -l
fi
```
</details>

<details>
  <summary><h2>Print esim via uiautomator</h2></summary>
  
Read the screen via uiautomator and print IMEI for all active esim/sim cards on the device.

```bash
#!/bin/bash
# Author: wuseman

### Launch IMEI (same result if you type in the caller app: *#06#) 
#### and dump screen to /tmp/read_screen.txt via uiautomator and parse esim IMEI:

adb shell input keyevent KEYCODE_CALL;
sleep 1;
input text '*#06#'; 
uiautomator dump --compressed /dev/stdout\
    |tr ' ' '\n'\
    |awk -F'"' '{print $2}'|grep "^[0-9]\{15\}$" \
    |nl -w 1 -s':'\
    |sed 's/^/IMEI/g'    
```
</details>

<details>
  <summary><h2>Print Device screen state</h2></summary>
  
```bash
screenState="$(adb shell dumpsys input_method \
   |grep -i "mSystemReady=true mInteractive=" \
   |awk -F= '{print $3}')"

if [[ $screenState = "true" ]]; then 
    echo "Screen is on"; 
else 
    echo "Screen is off"; 
fi
```
</details>

<details>
  <summary><h2>How reboot works on Android</h2></summary>

Reboot Commands:

- Reboot to Bootloader:
```bash
adb reboot bootloader
```

- Reboot to Fastboot:
```bash
adb reboot fastboot
```

- Reboot to Recovery:
```bash
adb reboot recovery
```

- Reboot to System:
```bash
adb reboot
```

What is a hot and warm reboot?

In order to answer your question, we need to define what a hot (or warm) reboot is on an Android device. Terms cold (or hard) boot and warm (or soft) boot are more associated with PCs, particularly Windows. For mobile phones or embedded devices, it's difficult to draw a clear line between cold and warm boot. In the case of a cold reboot, power is usually cut to CPUs, RAM, and even the whole motherboard. A soft reboot only kills and starts the processes while retaining power to hardware components. Power management is part of the open-source ACPI/UEFI/BIOS standard on PCs, while on phones, PMIC firmware is usually used with SoCs.

How does reboot really work on Android?

On (re)boot, SoC firmware loads bootloaders in memory which then load executable binaries and start processes (the actual OS). Android is based on the Linux kernel, which is the very first executable of the operating system that runs during the boot process. The kernel initializes necessary hardware and prepares a basic environment before executing `init`, the very first userspace process we can see. It's `init` that then starts and takes care of all services and processes.

A civilized way to reboot or shutdown is to let all processes terminate themselves, saving any pending work, unmounting filesystem

s, and then ask the kernel to reverse the boot process. `init` can handle this on modern OSes, or you can do it manually through the `/proc/sysrq-trigger` interface. Alternatively, we can ask the kernel to perform a quick reboot by killing everything. However, this may cause data loss, particularly due to filesystem corruption.

A brutal way is the long press of the power button (handled by PMIC), which is a cold reboot (or shutdown) in the true sense because the power to CPUs (and RAM) is suddenly cut without waiting for userspace processes and the kernel to terminate gracefully.

Does Android really perform a cold reboot?

On Android phones (and on other systems as well), a normal reboot is not completely cold as power is not cut, at least to RAM because it holds an area where kernel panic logs are stored, which can be accessed on the next boot (refer to ramoops used for last_kmsg or pstore). Similarly, some other memory regions allocated to SoC components and signed firmware that are isolated from the application processor (AP on which the main OS runs) may also not be erased. They include the Baseband Processor (modem), Digital Signal Processor (DSP), WiFi/BT module, etc.

However, a normal reboot isn't a warm reboot either. During reboot, the kernel kills itself and hands over control to bootloaders, which may boot the device in different possible modes (fastboot/bootloader, recovery, or normal boot). The low-level details are vendor and hardware-specific, whether a device performs a complete power-on reset (PoR) or if the hardware is not reset at all. Which components are powered down during different types of reboots depends on the interaction between the kernel, bootloader, SoC, PMIC, watchdog hardware, etc.

How to do a hot reboot?

The Linux kernel also supports another form of warm reboot: `kexec`. The kernel can terminate userspace processes and itself, executing a new kernel which can then start a new userspace environment without doing a hardware reset, POST, and re-initialization by BIOS. This approach is theoretically possible on Android too, i.e., the kernel re-executes itself with the proper command line and then starts `init`. However, it requires some device-specific changes to the kernel and ROM.

Stock Android doesn't provide a soft reboot functionality, but some custom ROMs implement this feature by triggering the restart method of the activity service. However, `init` itself and other core daemons like ueventd, vold, installd, surfaceflinger, logd, servicemanager, healthd, and a long list of vendor daemons aren't restarted.

To restart the device, you can use the following commands:

- On Android 9, the code for the restart method is `179`:
```bash
adb shell service call activity 179
```

- It's also possible to ask `init` to restart `zygote` and dependent services. However, SELinux won't let the property be set, so root access is required:
```bash
adb shell setprop ctl.restart zygote
```

</details>

## acpi

Show help for `acpi`:
```bash
adb shell acpi --help
```

Print Battery Percentage:
```bash
adb shell acpi 2> /dev/null
```

Show Cooling Device State:
```bash
adb shell 'su -c acpi -c'    
```

Show Temperatures:
```bash
adb shell acpi -t 2> /dev/null
```

Just print everything from `acpi`:
```bash
adb shell acpi -V


```

## adb

Environment variables:

| Variable                        | Description                                                  |
|---------------------------------|--------------------------------------------------------------|
| `$ADB_TRACE`                    | List of debug info to log                                    |
| `$ADB_VENDOR_KEYS`              | Colon Separated list of keys                                 |
| `$ANDROID_SERIAL`               | Serial number to connect to                                  |
| `$ANDROID_LOG_TAGS`             | Tags to be used by logcat                                    |
| `$ADB_LOCAL_TRANSPORT_MAX_PORT` | Max emulator scan port                                       |
| `$ADB_MDNS_AUTO_CONNECT`        | Comma Separated list of mdns services to allow auto-connect  |

Debug Commands:

Create `bugreport.zip` in `/sdcard` path:
```bash
adb bugreport /sdcard
```

List pids of processes hosting a JDWP transport:
```bash
adb jdwp
```

Show device log (logcat --help for more):
```bash
adb logcat
```

Sideload the given full OTA package:
```bash
adb sideload OTAPACKAGE
```

Restart `adbd` with root permissions:
```bash
adb root
```

Restart `adbd` without root permissions:
```bash
adb unroot
```

Restart `adbd` listening on USB:
```bash
adb usb
```

Restart `adbd` listening on TCP on `PORT`:
```bash
adb tcpip <port>
```

Restart userspace:
```bash
adb reboot userspace
```

Internal debugging:

Start `adb` server:
```bash
adb start-server
```

Kill `adb` server:
```bash
adb kill-server
```

Stop `adb` server:
```bash
adb stop-server
```

Kick connection from the host side to force reconnect:
```bash
adb reconnect
```

Kick connection from the device side to force reconnect:
```bash
adb reconnect device
```

Reset offline/unauthorized devices to force reconnect:
```bash
adb reconnect offline
```

USB Commands:

Attach a detached USB device:
```bash
adb attach
```

Detach from a USB device to allow use by others:
```bash
adb detach
```

Shell Commands:

Enter the device `shell`:
```bash
adb shell
```

Choose escape character or `"none"` (default '`~`'):
```bash
adb shell -e
```

Don't read from `stdin`:
```bash
adb shell -n
```

Disable pty allocation:
```bash
adb shell -T
```

Allocate a `pty` if on a `tty`:
```bash
adb shell -t
```

Force allocate a `pty` if on a `tty`:
```bash
adb shell -tt
```

Disable remote exit codes and `stdout/stderr` separation:
```bash
adb shell -x
```

Run `emulator` console command:
```bash
adb shell emu COMMAND
```

Enter the device shell when there is more than one device connected:

- USB connected:
```bash
adb -s <serial> shell
```

- Network connected:
```bash
adb -s <ip:port> shell
```

Print connection status:
```bash
adb devices -l
```

Print adb help:
```bash
adb help
```
Print the currend:
```bash
adb version
```


## Network Commands

Connect to a device via TCP/IP:

```bash
adb connect <ip:port>
```

Disconnect all connected devices:

```bash
adb disconnect all
```

Disconnect from a given TCP/IP device:

```bash
adb disconnect <ip:port>
```

Pair with a device for secure TCP/IP communication:

```bash
adb pair <ip:port> <pairing code>
```

List all forward socket connections:

```bash
adb forward --list
```

Forward socket connection:

- Port 0 = Any port

```bash
adb forward tcp:<port>
```

```bash
adb forward localabstract:<socket name>
```

```bash
adb forward localreserved:<socket name>
```

```bash
adb forward localfilesystem:<socket name>
```

```bash
adb forward jdwp:<process pid> (remote only)
```

```bash
adb forward vsock:<CID>:<port> (remote only)
```

```bash
adb forward acceptfd:<fd> (listen only)
```

Remove a specific forward socket connection:

```bash
adb forward --remove 'local/remote'
```

Remove all forward socket connections:

```bash
adb forward --remove-all
```

Run PPP over USB:

```bash
adb ppp TTY <parameter>
```

List all reverse socket connections:

```bash
adb reverse --list
```

Reverse socket connection using:

```bash
adb reverse tcp:<port>
```

```bash
adb reverse localabstract:<socket name>
```

```bash
adb reverse localreserved:<socket name>
```

```bash
adb reverse localfilesystem:<socket name>
```

Remove a specific reverse socket connection:

```bash
adb reverse --remove 'LOCAL/REMOTE'
```

Remove all reverse socket connections:

```bash
adb reverse --remove-all
```

Check if mdns discovery is available:

```bash
adb mdns check
```

List all discovered services:

```bash
adb mdns services
```

</details>

<details>
  <summary><h2>Security Commands</h2></summary>

Disable dm-verity checking on userdebug builds:

```bash
adb disable-verity
```

Re-enable dm-verity checking on userdebug builds:

```bash
adb enable-verity
```

Generate adb public/private key:

```bash
adb keygen <filename>
```

</details>

<details>
  <summary><h2>Scripting Commands</h2></summary>

Wait for the device to be in a given state:

- STATE: `device`, `recovery`, `rescue`, `sideload`, `bootloader`, or `disconnect`
- TRANSPORT: `usb`, `local`, or any `[default=any]`

```bash
adb wait-for[-TRANSPORT]-STATE...
```

Print the device state (offline|bootloader|device):

```bash
adb get-state
```

Print the device serial number:

```bash
adb get-serialno
```

Print dev path:

```bash
adb get-devpath
```

Remount partitions read-write:

```bash
adb remount -R
```

</details>

<details>
  <summary><h2>am (Activity Manager) Commands</h2></summary>

Print all activities that we can launch for the preferred application:

```bash
#!/bin/bash
# Author: wuseman

dumpsys package com.android.settings \
    |grep -Eo "^[[:space:]]+[0-9a-f]+[[:space:]]+com.android.settings/[^[:space:]]+Activity" \
    |grep -io

 com.* \
    |sed "s/^/am start -n '/g" \
    |sed "s/$/'/g"
```

Print all activities that we can launch for all installed applications on the device:

```bash
#!/bin/bash
# Author: wuseman

for packages in $(cmd package list packages|awk -F':' '{print $2}'|sort); do
    dumpsys package ${packages} \
    |grep -Eo "^[[:space:]]+[0-9a-f]+[[:space:]]+${packages}/[^[:space:]]+Activity" \
    |grep -io com.* \
    |sed "s/^/am start -n '/g" \
    |sed "s/$/'/g" \
    |awk '!seen[$0]++'
done
```

Dump all activities for all packages enabled on the device and store it in a file:

```bash
for packages in $(cmd package list packages|awk -F':' '{print $2}'|sort); do
    dumpsys package ${packages} \
    |grep -Eo "^[[:space:]]+[0-9a-f]+[[:space:]]+${packages}/[^[:space:]]+Activity" \
    |grep -io com.* \
    |sed "s/^/am start -n '/g" \
    |sed "s/$/'/g" \
    |awk '!seen[$0]++'
done > /sdcard/wuseman/all_activitys_for_am.txt
```

Launch an application and enter the activity section of the application's settings:

```bash
am start -n 'com.android.settings/.Settings$ApnSettingsActivity'
```

Launch sysdump menu:

```bash
adb shell am start com.sec.android.app.servicemodeapp/.SysDump
```

Launch cp debug menu:

```bash
adb shell am start com.sec.android.app.servicemodeapp/.CPDebugLevel
```

Launch rtn view menu:

```bash
adb shell am start com.sec.android.app.servicemodeapp/.RTN_View
```

Launch RAMDUMP settings:

```bash
adb shell am start com.sec.android.app.servicemodeapp/.CPDebugLevel
```

Launch reset call time:

```bash
adb shell am start com.sec.android.app.servicemodeapp/.ResetTotalCallTime
```

Launch total call menu:

```bash
adb shell am start com.sec.android.app.servicemodeapp/.TotalCallTime
```

Launch wifi menu:

```bash
adb shell am start com.sec.android.app.servicemodeapp/.WifiInfoActivity
```

Launch NAND Flash:

```bash
adb shell am start com.sec.android.app.servicemodeapp/.NandFlashHeaderRead
```

Launch modemui activities:

```bash
adb shell am start com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.PhoneUtil
```

Launch ESC settings:

```bash
adb shell am start com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.PhoneUtil_ESC
```

Launch UART USBC TC settings:

```bash
adb shell am start com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.SetPortUartUSBCTCModel
```

Launch SGLTE settings:

```bash
adb shell am start com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.PhoneUtil_SGLTE
```

Launch TD settings:

```bash
adb shell am start com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.PhoneUtil_TD
```

Launch MarvellVIA settings:

```bash
adb shell am start

 com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.PhoneUtil_MarvellVIA
```

Launch BCOM settings:

```bash
adb shell am start com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.PhoneUtil_Bcom
```

Launch and show IMEI activity:

```bash
adb shell am start com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.ShowIMEI
```

Launch UART USB MSM8960 Port settings:

```bash
adb shell am start com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.SetPortUartUsbMSM8960
```

Launch MDM 9x15 settings:

```bash
adb shell am start com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.PhoneUtil_MDM9x15
```

Launch USB Settings:

```bash
adb shell am start com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.USBSettings
```

Launch auto answer settings:

```bash
adb shell am start com.sec.android.app.servicemodeapp/com.sec.android.app.modemui.activities.AutoAnswer
```

</details>

<details>
  <summary><h2>adb Shell Commands</h2></summary>

Enter the device shell:

```bash
adb shell
```

Choose an escape character or "none" (default: '~'):

```bash
adb shell -e
```

Don't read from stdin:

```bash
adb shell -n
```

Disable pty allocation:

```bash
adb shell -T
```

Allocate a pty if on a tty:

```bash
adb shell -t
```

Force allocate a pty if on a tty:

```bash
adb shell -tt
```

Disable remote exit codes and stdout/stderr separation:

```bash
adb shell -x
```

Run emulator console command:

```bash
adb shell emu COMMAND
```

Enter the device shell when there is more than one device connected:

- USB connected:

```bash
adb -s <serial> shell
```

- Network connected:

```bash
adb -s <ip:port> shell
```

Print the connection status:

```bash
adb devices -l
```

Print adb help:

```bash
adb help
```

Print the current adb version installed:

```bash
adb version
```

</details>

The provided commands allow you to perform various actions related to the `com.google.android.gms` package, launch different settings and activities, and interact with other applications. Here are the commands:

Launch Find My Device settings:

```bash
adb shell am start -n com.google.android.gms/.mdm.settings.FindMyDeviceSettingsActivity
```

Launch Nearby Sharing settings:

```bash
adb shell am start -n com.google.android.gms/.nearby.sharing.ReceiveSurfaceActivity
```

Launch Personal Google Setup (requires root):

```bash
adb shell su -c am start com.google.android.gms/.accountsettings.mg.ui.main.MainActivity
```

Launch hidden settings for SMS verification codes (requires root):

```bash
adb shell su -c am start com.google.android.gms/.auth.api.phone.ui.AutofillSettingsCollapsingActivity
```

Set wallpaper for the current image opened:

```bash
adb shell am start -a android.intent.action.SET_WALLPAPER
```

Open SIM ID settings for APN:

```bash
adb shell am start -a android.intent.action.INSERT -d content://telephony/carriers --ei simId
```

Launch default action view:

```bash
adb shell am start -a android.intent.action.VIEW
```

Open the default browser and visit a URL:

```bash
adb shell am start -a android.intent.action.VIEW -d https://www.example.com
```

Launch Google Maps with fixed coordinates:

```bash
adb shell am start -a android.intent.action.VIEW -d "geo:46.457398,-119.407305"
```

Launch Facebook inbox URI:

```bash
adb shell am start -a android.intent.action.VIEW -d facebook://facebook.com/inbox
```

Open a vCard file from the SD card:

```bash
adb shell am start -a android.intent.action.VIEW -d file:///sdcard/me.vcard -t text/x-vcard
```

Launch the default music player and play a file:

```bash
adb shell am start -a android.intent.action.VIEW -d file:////storage/9A8A-1069/wuseman/ringtones/<mp3_track>.mp3 -t audio/mp3
```

Launch the default video player and play a file:

```bash
adb shell am start -a android.intent.action.VIEW -d file:///sdcard/sound.ogg -t audio/ogg
```

Launch the default video player and play a video file:

```bash
adb shell am start -a android.intent.action.VIEW -d file:///sdcard/video.mkv -t video/mkv
```

Launch Android settings:

```bash
adb shell am start -n com.android.settings/com.android.settings.Settings
```

Launch Android sub-settings:

```bash
adb shell am start com.android.settings/com.android.settings.SubSettings
```

Open the camera in photo mode:

```bash
adb shell am start -a android.media.action.IMAGE_CAPTURE
```

Open the camera app in video mode:

```bash
adb shell am start -a android.media.action.VIDEO_CAMERA
```

Open the camera app in QR mode:

```bash
adb shell am start -n 'com.sec.android.app.camera/.QrScannerActivity'
```

Open My Files (file manager):

```bash
adb shell am start com.sec.android.app.myfiles/com.sec.android.app.myfiles.external.ui.MainActivity
```

Open SIM card settings from the setup wizard:

```bash
su -c am start -n 'com.sec.android.app.SecSetupWizard/.UI.SimTssSetupActivity'
```

Open the last page in the setup wizard that indicates everything has been done:

```bash
su -c am start -n '

com.sec.android.app.SecSetupWizard/.UI.OutroActivity'
```

Open the last page in the setup wizard with the spinning bar:

```bash
su -c am start -n 'com.sec.android.app.SecSetupWizard/.kme.B2bDeviceCheckActivity'
```

Open Samsung Features window from the setup wizard:

```bash
su -c am start -n 'com.sec.android.app.SecSetupWizard/.UI.AlternativePermissionActivity'
```

Open Take Care of Your Device window from the setup wizard:

```bash
su -c am start -n 'com.sec.android.app.SecSetupWizard/.UI.NoticeActivity'
```

Open the language window from the setup wizard:

```bash
su -c am start -n 'com.sec.android.app.SecSetupWizard/.UI.LanguageSelectionActivity'
```

Send a simple notification:

```bash
adb shell am broadcast -n your.package.name/com.google.firebase.iid.FirebaseInstanceIdReceiver -c your.package.name -a com.google.android.c2dm.intent.RECEIVE
```

Send a notification:

```bash
adb shell am broadcast -n com.android.google.youtube/com.google.firebase.iid.FirebaseInstanceIdReceiver -a "com.google.android.c2dm.intent.RECEIVE" -es "title" "Title" --es "body" "Body"
```

Broadcast a push notification locally using ADB without a network connection:

```bash
adb shell am broadcast -n com.your.app/com.google.firebase.iid.FirebaseInstanceIdReceiver -a "com.google.android.c2dm.intent.RECEIVE" --es "extra1" "65" --es "guid" "1f400184-9215-479c-b19a-a9cd9a1d9dc9" --es "extra3" "VALUE" --es "extra4" "'Long string with spaces'"
```

Add a value to the default shared preferences:

```bash
adb shell am broadcast -a org.example.app.sp.PUT --es key key_name --es value "hello world!"
```

Remove a value from the default shared preferences:

```bash
adb shell am broadcast -a org.example.app.sp.REMOVE --es key key_name
```

Clear all default shared preferences:

```bash
adb shell am broadcast -a org.example.app.sp.CLEAR --es key key_name
```

Restart the application process after making changes:

```bash
adb shell am broadcast -a org.example.app.sp.CLEAR --ez restart true
```

Set default preferences for an app:

```bash
adb shell am broadcast -a org.example.app.sp.CLEAR --es key key_name
```

Factory reset your device after the next reboot:

```bash
adb shell am broadcast -a android.intent.action.MASTER_CLEAR
reboot
```

Launch the Launcher activities (requires root):

```bash
adb shell am start com.sec.android.app.launcher/com.sec.android.app.launcher.activities.LauncherActivity
```

Launch the homescreen (requires root):

```bash
adb shell am start com.sec.android.app.launcher/com.android.launcher3.uioverrides.QuickstepLauncher
```

  Here are some additional ADB shell commands:

Open hidden menu and select "enable":

```bash
adb shell "su -c am broadcast -a android.provider.Telephony.SECRET_CODE -d android_secret_code://HIDDENMENUENABLE"
```

Open internal operation test menu:

```bash
adb shell "su -c am broadcast -a android.provider.Telephony.SECRET_CODE -d android_secret_code://IOTHIDDENMENU"
```

Open a dialog box followed by another dialog asking for the unlock key code:

```bash
adb shell "su -c am broadcast -a android.provider.Telephony.SECRET_CODE -d android_secret_code://UNLOCKKERNEL"
```

Send an SMS:

```bash
adb shell am broadcast -a com.whereismywifeserver.intent.TEST --es sms_body "test from adb"
```

Trigger test GSM cell broadcasts:

```bash
adb shell am broadcast -a com.android.internal.telephony.gsm.TEST_TRIGGER_CELL_BROADCAST --es pdu_string <pdu_string>
```

Simulate wake mode:

```bash
adb shell am set-inactive <packageName>
adb shell am set-inactive <packageName> false
```

Enable Demo Mode:

```bash
adb shell settings put global sysui_demo_allowed 1
```

Enable car dialer:

```bash
adb shell am broadcast -a com.android.car.dialer.intent.action.adb -es "action" "connect"
```

Make a call:

```bash
adb shell am broadcast -a com.android.car.dialer.intent.action.adb --es "action" "addCall" --es "id" "4085524874"
```

Receive an incoming call:

```bash
adb shell am broadcast -a com.android.car.dialer.intent.action.adb --es "action" "rcvCall" --es "id" "4085524874"
```

Merge calls:

```bash
adb shell am broadcast -a com.android.car.dialer.intent.action.adb --es "action" "unholdCall"
```

Hold a call:

```bash
adb shell am broadcast -a com.android.car.dialer.intent.action.adb --es "action" "holdCall"
```

Unhold a call:

```bash
adb shell am broadcast -a com.android.car.dialer.intent.action.adb --es "action" "unholdCall"
```

End a call:

```bash
adb shell am broadcast -a com.android.car.dialer.intent.action.adb --es "action" "endCall" --es "id" "4085524874"
```

Clear call history:

```bash
adb shell am broadcast -a com.android.car.dialer.intent.action.adb --es "action" "clearAll"
```

Press home and print call status:

```bash
adb shell am start -W -c android.intent.category.HOME -a android.intent.action.MAIN
```

Display time in `hhmm` format:

```bash
adb shell am broadcast -a com.android.systemui.demo -e command clock -e hhmm 1200
```

Print network data type:

```bash
adb shell am broadcast -a com.android.systemui.demo -e command network -e mobile show -e level 4 -e datatype false
```

Hide notifications:

```bash
adb shell am broadcast -a com.android.systemui.demo -e command notifications -e visible false
```

Restart the system service:

```bash
adb shell am startservice -n com.android.systemui/.SystemUIService
```

Open Google Camera (Pixel 4):

```bash
adb shell am start com.google.android.GoogleCamera
```

Find all available modes to launch in GUI (Samsung):

```bash


adb shell cmd package dump com.samsung.android.app.telephonyui | grep "Activity filter" | awk '{print $2}' | awk '!seen[$0]++'
```

Launch Samsung Dialer:

```bash
adb shell am start com.samsung.android.dialer/.DialtactsActivity
```

Launch Samsung SMS application:

```bash
adb shell am start com.samsung.android.messaging/com.samsung.android.messaging.ui.view.setting.MainSettingActivity
```

Launch Samsung Messenger conversation composer:

```bash
adb shell am start com.samsung.android.messaging/com.android.mms.ui.ConversationComposer
```

Launch Samsung Messenger in contacts view:

```bash
adb shell am start com.samsung.android.messaging/com.samsung.android.messaging.ui.view.recipientspicker.PickerActivity
```

Launch Samsung Messenger with recent activity:

```bash
adb shell am start com.samsung.android.dialer/com.samsung.android.dialer.calllog.view.picker.CallLogPickerActivity
```

Launch Samsung Gallery:

```bash
adb shell am start com.sec.android.gallery3d/com.samsung.android.gallery.app.activity.GalleryActivity
```

Launch secret project menu (Huawei only):

```bash
adb shell am start com.huawei.android.projectmenu/com.huawei.android.projectmenu.ProjectMenuActivity
```

Set the application run in the background behavior:

```bash
adb shell cmd appops set <package_name> RUN_IN_BACKGROUND ignore
```

Set any application run in the background behavior:

```bash
adb shell cmd appops set <package_name> RUN_ANY_IN_BACKGROUND ignore
```

Set the application to launch in the foreground:

```bash
adb shell cmd appops set <package_name> START_FOREGROUND ignore
```

Set the application settings for instant launch in the foreground:

```bash
adb shell cmd appops set <package_name> INSTANT_APP_START_FOREGROUND ignore
```

Set application permission for clipboard:

```bash
adb shell cmd appops set <packagename> READ_CLIPBOARD allow
```

Paste clipboard:

```bash
adb shell input keyevent PASTE
```

Set application with read permissions for the clipboard:

```bash
adb cmd appops set com.bankid.bus READ_CLIPBOARD allow
```

Add text to clipboard:

```bash
am broadcast -a clipper.set -e text "text"
```

Get airplane mode status:

```bash
adb shell cmd connectivity airplane-mode
```

Set airplane mode (enable/disable):

```bash
adb shell cmd connectivity airplane-mode enable|disable
```

Send notify and push notice to the notification bar:

```bash
adb shell su -lp 2000 -c "cmd notification post -S bigtext -t 'adb pwnz' 'Tag' 'it rly does'"
```

Set resume on reboot provider package:

```bash
adb shell cmd lock_settings set-resume-on-reboot-provider-package <package_name>
```

Remove cached unified challenge for the managed profile:

```bash
adb shell cmd lock_settings remove-cache --user 0
```

Verify the lock credentials:

```bash
adb shell cmd lock_settings verify --old 1234 --user 0
```

Clear the lock credentials:

```bash
adb shell cmd lock_settings clear --old 1234 --user 0
```

Enable/disable synthetic password:

```bash
adb shell cmd lock_settings sp --old 1234 --user 0 <1|0>
```

Get whether synthetic password is enabled:

```bash
adb shell cmd lock_settings sp --old 1234 --user 0
```

Set the lock screen as a password using the given password to unlock:

```bash
adb shell cmd lock_settings set-password --old 1234 --user 0

 'newPassword'
```

Set the lock screen as a PIN using the given PIN to unlock:

```bash
adb shell cmd lock_settings set-pin --old 1234 --user 0 'newPin'
```

Set the lock screen as a pattern using the given pattern to unlock:

```bash
adb shell cmd lock_settings set-pattern --old 1234 --user 0 'newPattern'
```

When true, disable lock screen:

```bash
adb shell cmd lock_settings set-disabled --old 1234 --user 0 true|false
```

Check whether the lock screen is disabled:

```bash
adb shell cmd lock_settings get-disabled --old 1234 --user 0
```

Print stats:

```bash
adb shell cmd stats print-stats
```

Send a broadcast that triggers the subscriber to fetch metrics:

```bash
adb shell cmd stats send-broadcast uid name
```

Flush all data in memory to disk:

```bash
adb shell cmd stats write-to-disk
```

Print the UID, app name, version mapping:

```bash
adb shell cmd stats print-uid-map
```

Log a binary push state changed event:

```bash
adb shell cmd stats log-binary-push NAME VERSION STAGING ROLLBACK_ENABLED LOW_LATENCY STATE EXPERIMENT_IDS
```

Hide all notification icons on the status bar:

```bash
adb shell cmd statusbar send-disable-flag notification-icons
```

Reset all flags to default:

```bash
adb shell cmd statusbar send-disable-flag none
```

Print status bar icons:

```bash
adb shell cmd statusbar get-status-icons
```

Print preferences for the status bar:

```bash
adb shell cmd statusbar prefs list-prefs
```

Expand the status bar:

```bash
adb shell cmd statusbar expand-notifications
```

Collapse the status bar:

```bash
adb shell cmd statusbar collapse
```

Expand full settings:

```bash
adb shell cmd statusbar expand-settings
```

Get the auth user:

```bash
adb shell cmd user list
```

Here are some additional commands related to UI mode, Wi-Fi, media session, package management, and app operations:

Enable night mode (Dark Mode):
```bash
adb shell cmd uimode night yes
```

Disable night mode:
```bash
adb shell cmd uimode night no
```

Enable car mode:
```bash
adb shell cmd uimode car yes
```

Disable car mode:
```bash
adb shell cmd uimode car no
```

Scan for nearby SSIDs and fetch Wi-Fi data:
```bash
#!/bin/bash

adb shell cmd -w wifi start-scan
sleep 7
adb shell cmd -w wifi list-scan-results
```

Sets whether we are in the middle of an emergency call:
```bash
adb shell cmd -w wifi set-emergency-call-state enabled|disabled
```

Sets whether Emergency Callback Mode (ECBM) is enabled:
```bash
adb shell cmd -w wifi set-emergency-callback-mode enabled|disabled
```

Lists the suggested networks from the app:
```bash
adb shell cmd -w wifi list-suggestions-from-app com.app.example
```

Lists all suggested networks on this device:
```bash
adb shell cmd -w wifi list-all-suggestions
```

Queries whether network requests from the app are approved or not:
```bash
adb shell cmd -w wifi network-requests-has-user-approved com.app.example
```

Sets whether network requests from the app are approved or not:
```bash
adb shell cmd -w wifi network-requests-set-user-approved com.app.example yes|no
```

Lists the requested networks added via the shell:
```bash
adb shell cmd -w wifi list-requests
```

Removes all active requests added via the shell:
```bash
adb shell cmd -w wifi remove-all-requests
```

Remove a network request with the provided SSID of the network:
```bash
adb shell cmd -w wifi remove-request <ssid>
```

Add a network request with the provided parameters:
```bash
adb shell cmd -w wifi add-request <ssid> open|owe|wpa2|wpa3 [<passphrase>] [-b <bssid>]
```

Initiates Wi-Fi settings reset:
```bash
adb shell cmd -w wifi settings-reset
```

Gets soft AP supported features:
```bash
adb shell cmd -w wifi get-softap-supported-features
```

Sets whether Wi-Fi watchdog should trigger recovery:
```bash
adb shell cmd -w wifi set-wifi-watchdog enabled|disabled
```

Sets country code to `<two-letter code>` or left for normal value:
```bash
adb shell cmd -w wifi force-country-code enabled <two-letter code> | disabled
```

Manually triggers a link probe:
```bash
adb shell cmd -w wifi send-link-probe
```

Clears the user-disabled networks list:
```bash
adb shell cmd -w wifi clear-user-disabled-networks
```

Removes all user-approved network requests for the app:
```bash
adb shell cmd -w wifi network-requests-remove-user-approved-access-points com.app.example
```

Clear the user choice on Imsi protection exemption for the carrier:
```bash
adb shell cmd -w wifi imsi-protection-exemption-clear-user-approved-for-carrier <carrier id>
```

Queries whether Imsi protection exemption for the carrier is approved or not:
```bash
adb shell cmd -w wifi imsi-protection-exemption-has-user-approved-for-carrier <carrier id>
```

Sets whether Imsi protection exemption for the carrier is approved or not:
```bash
adb shell cmd -w wifi imsi-protection-exemption-set-user-approved-for

-carrier <carrier id> yes|no
```

Print all available apps that can be manually started from the activity manager:
```bash
adb shell cmd package dump com.android.com | grep "Activity filter" | awk '{print $2}'
```

List UID owner of a package:
```bash
adb shell cmd package list packages -U
```

Print all applications sorted by alpha:
```bash
adb shell cmd package list packages | awk -F: '{print $2}' | sort
```

List packages:
```bash
adb shell cmd package list packages -l
```

List disabled packages:
```bash
adb shell cmd package list packages -d
```

Filter to only show enabled packages:
```bash
adb shell cmd package list packages -e
```

Filter to only show third-party packages:
```bash
adb shell cmd package list packages -3
```

Set the default home activity (aka launcher):
```bash
adb shell cmd package set-home-activity [--user USER_ID] TARGET-COMPONENT
```

Print all features of the system:
```bash
adb shell cmd package list features
```

Print briefs:
```bash
adb shell cmd package resolve-activity --brief com.facebook.katana
```

Connect to AudioService:
```bash
adb shell cmd media_session volume
```

Set the volume of media to a value (0-15):
```bash
adb shell media volume --show --stream 3 --set N
```

Set fine volume in Samsung devices (0-150):
```bash
adb shell cmd media_session volume --show --setfine 150
```

Control the stream (3=STREAM_MUSIC) and set the volume index to 11:
```bash
adb shell cmd media_session volume --show --stream 3 --set 11
```

Print the current volume from all streams:
```bash
adb shell cmd media_session volume --get
```

Print the current volume range from stream 3:
```bash
adb shell cmd media_session volume --stream 3 --get
```

Print the current fine volume:
```bash
adb shell cmd media_session volume --getfine
```

Increase media volume:
```bash
adb shell cmd media_session volume --show --adj raise
```

Decrease media volume:
```bash
adb shell cmd media_session volume --show --adj lower
```

Print the current volume (1-15):
```bash
adb shell cmd media_session volume --get
```

Print a list of current media sessions:
```bash
adb shell cmd media_session list-sessions
```

Monitor updates to the specified session (use tag output from list-sessions):
```bash
cmd media_session monitor spotify-media-session
```

Set application run-in-background behavior:
```bash
cmd appops set <package_name> RUN_IN_BACKGROUND ignore
```

Set any application run-in-background behavior:
```bash
cmd appops set <package_name> RUN_ANY_IN_BACKGROUND ignore
```

Set application to launch in the foreground:
```bash
cmd appops set <package_name> START_FOREGROUND ignore
```

Set application settings for instant launch in the foreground:
```bash
cmd appops set <package_name> INSTANT_APP_START_FOREGROUND ignore
```

Set application permission for clipboard:
```bash
cmd appops set <packagename> READ_CLIPBOARD allow
```

Please note that some commands may require root access or specific privileges.

Here are some additional commands related to media dispatch and content management:

Dispatch media commands:

Press `play` key:
```bash
adb shell cmd media_session dispatch play
```

Dispatch `pause`:
```bash
adb shell cmd media_session dispatch pause
```

Dispatch `play-pause`:
```bash
adb shell cmd media_session dispatch play-pause
```

Dispatch `mute`:
```bash
adb shell cmd media_session dispatch mute
```

Dispatch `headsethook`:
```bash
adb shell cmd media_session dispatch headsethook
```

Dispatch `stop`:
```bash
adb shell cmd media_session dispatch stop
```

Dispatch `next`:
```bash
adb shell cmd media_session dispatch next
```

Dispatch `previous`:
```bash
adb shell cmd media_session dispatch previous
```

Dispatch `rewind`:
```bash
adb shell cmd media_session dispatch rewind
```

Dispatch `record`:
```bash
adb shell cmd media_session dispatch record
```

Dispatch `fast-forward`:
```bash
adb shell cmd media_session dispatch fast-forward
```

Content management:

Find contents in SDK map and create samples for this cheatsheet:
```bash
rg . | rg 'content://.*"' -o | cut -d '"' -f1 | sed 's/^/adb shell content query --uri /g' | sed 'i### Print\''
```

Find contents in SDK map and create samples for this cheatsheet:
```bash
adb shell "su -c content query --uri content://com.samsung.rcs.autoconfigurationprovider/root/* | tr ' ' '\n'"
```

Print device number:
```bash
adb shell "su -c content query --uri content://com.samsung.rcs.autoconfigurationprovider/root/application/1/im/fthttpcsuser"
```

Delete a certain setting:
```bash
adb shell content delete --uri content://settings/secure --where "name='new_setting'"
```

Insert a setting and value to foo:
```bash
adb shell content insert --uri content://settings/secure --bind name:s:user_setup_complete --bind value:s:1
```

Change setting to another setting:
```bash
adb shell content update --uri content://settings/secure --bind value:s:newer_value --where "name='new_setting'"
```

Select "name" and "value" columns from secure settings where "name" is equal to "new_setting" and sort the result by name in ascending order:
```bash
adb shell content query --uri content://settings/secure --projection name:value --where "name='new_setting'" --sort "name ASC"
```

Read and redirect file:
```bash
adb shell 'content read --uri content://settings/system/ringtone_cache' > foo.ogg
```

Set ringtone:
```bash
adb shell 'content write --uri content://settings/system/ringtone_cache' < host.ogg
```

Print current type for a content:
```bash
adb shell content gettype --uri content://media/internal/audio/media/
```

Print cell broadcasts, call logs, downloads, contacts, MMS, and more:

Print cell broadcasts:
```bash
adb shell content query --uri content://cellbroadcasts
```

Print call logs:
```bash
adb shell content query --uri content://call_log/calls
```

Print downloads:
```bash
adb shell content query --uri content://downloads/my_downloads
```

Print contacts:
```bash
adb shell content query --uri content://contacts/people
```

Print MMS:
```bash
adb shell content query --uri content://mms
```

Print SMS:
```bash
adb shell content query --uri content://sms


Print device carriers:
```bash
adb shell su -c content query --uri content://telephony/carriers
```

## ffplay

### Stream via FFplay 

FFPlay Default - No Settings

```bash
adb shell screenrecord --Example Output-format=h264 - | ffplay -
```

FFplay Customized

```bash
adb exec-out screenrecord |    --Example Output-format=h264 - |        |ffplay |        -framerate 60 |        -probesize 32 |        -sync video  -
```

```bash
adb shell screenrecord |    --bit-rate=16m |    --Example Output-format=h264 |    --size 800x600 - |        |ffplay |        -framerate 60 |        -framedrop |        -bufsize 16M -
```

FFplay Customized - Stream in 1080p quality

```bash
adb exec-out screenrecord |    --bit-rate=16m |    --Example Output-format=h264 |    --size 1920x1080 -
```

## getprop

Use the `getprop` command to get system properties.

### Example Usage

```bash
adb shell getprop | egrep "model|version.sdk|manufacturer|hardware|platform|revision|serialno|product.name|brand" 2>/dev/null
```

Print CPU ABI

```bash
adb shell getprop ro.product.cpu.abi
```

Get info if OEM Unlock is Allowed

```
1 = Enabled
0 = Disabled
```

```bash
adb shell getprop sys.oem_unlock_allowed 
```

Is System boot completed

```bash
adb shell getprop sys.boot_completed
```


## input tap 

```bash
Origin (0, 0)
+----------------------------------+
|  Status Bar   (500,0) X         |
|----------------------------------+
|                                  |
|                                  |
|                                  |
|  (100,1150) O                    |
|                                  |
|                                  |
|                                  |
|                                  |
|                                  | ---- O (250,1150)
|                                  |
|                                  |
|                                  |
|                                  |
|                                  || --- Power Button
|                                  |
|                                  |
|                                  |
|                                  |
|                                  |
|                                  |
|                                  || ---- O (1000,1000)
|                                  |
|                                  |
|              Antenna             |
|(0,2300) X                        |
------------------------------------ ---- Y(2300)
           100       300      500

```

In this ASCII art:
- `(500,0)` marks the top-right corner of the screen.
- `(0,2300)` marks the bottom-left corner of the screen.
- `(100,1150)` marks a location towards the left-middle of the screen.
- `(250,1150)` marks a location at the middle of the screen.
- `(1000,1000)` marks a location a bit lower but more towards the right side.


- To tap on the center of the screen:
  ```
  adb shell input tap 250 1150
  ```

- To tap on the Power button:
  ```
  adb shell input tap 150 350
  ```

- To tap on the top-right corner of the screen:
  ```
  adb shell input tap 500 0
  ```

- To tap on the bottom-left corner of the screen:
  ```
  adb shell input tap 0 2300
  ```

- To tap on the middle-left of the screen:
  ```
  adb shell input tap 100 1150
  ```

- To tap on the middle-right of the screen:
  ```
  adb shell input tap 400 1150
  ```

- To simulate a swipe from the middle of the screen to the top (useful for scrolling):
  ```
  adb shell input swipe 250 1150 250 300
  ```

- To simulate a swipe from the middle of the screen to the bottom (also for scrolling):
  ```
  adb shell input swipe 250 1150 250 2000
  ```

- To simulate a long press at the middle of the screen (for context menus or similar functionality):
  ```
  adb shell input swipe 250 1150 250 1150 2000
  ```

- To simulate a pinch gesture (zoom out), we can use two swipe commands that start from different points and converge in the middle of the screen:
  ```
  adb shell input swipe 100 1000 250 1150 & adb shell input swipe 400 1300 250 1150
  ```

- To simulate a spread gesture (zoom in), we can use two swipe commands that start from the same point in the middle of the screen and move towards different ends:
  ```
  adb shell input swipe 250 1150 100 1000 & adb shell input swipe 250 1150 400 1300
  ```


- To tap on the top-left corner of the screen:
    ```
    adb shell input tap 0 0
    ```

- To tap on the bottom-right corner of the screen:
    ```
    adb shell input tap 500 2300
    ```

- To simulate a swipe from the bottom to the top of the screen (reverse scrolling):
    ```
    adb shell input swipe 250 2000 250 300
    ```

- To simulate a swipe from top to the bottom of the screen (reverse scrolling):
    ```
    adb shell input swipe 250 300 250 2000
    ```

- To simulate a long press at the top-left of the screen:
    ```
    adb shell input swipe 0 0 0 0 2000
    ```

- To simulate a long press at the bottom-right of the screen:
    ```
    adb shell input swipe 500 2300 500 2300 2000
    ```

- To simulate a drag gesture from the middle of the screen to the top-right:
    ```
    adb shell input swipe 250 1150 500 0
    ```

- To simulate a drag gesture from the middle of the screen to the bottom-left:
    ```
    adb shell input swipe 250 1150 0 2300
    ```

- To tap on a point in the upper-middle section of the screen:
    ```
    adb shell input tap 250 600
    ```

- To tap on a point in the lower-middle section of the screen:
    ```
    adb shell input tap 250 1800
    ```

- To simulate a swipe diagonally from the top-left to the bottom-right of the screen:
    ```
    adb shell input swipe 0 0 500 2300
    ```

- To simulate a swipe diagonally from the top-right to the bottom-left of the screen:
    ```
    adb shell input swipe 500 0 0 2300
    ```

- To simulate a swipe diagonally from the bottom-left to the top-right of the screen:
    ```
    adb shell input swipe 0 2300 500 0
    ```

- To simulate a swipe diagonally from the bottom-right to the top-left of the screen:
    ```
    adb shell input swipe 500 2300 0 0
    ```


- To simulate a swipe from left to right across the screen (useful for going to the next item in a carousel):
    ```
    adb shell input swipe 0 1150 500 1150
    ```

- To simulate a swipe from right to left across the screen (useful for going to the previous item in a carousel):
    ```
    adb shell input swipe 500 1150 0 1150
    ```

- To simulate a tap on the "Back" button area (assuming it's at the bottom-left of the screen):
    ```
    adb shell input tap 50 2250
    ```

- To simulate a tap on the "Home" button area (assuming it's at the bottom-middle of the screen):
    ```
    adb shell input tap 250 2250
    ```

- To simulate a tap on the "Recent Apps" button area (assuming it's at the bottom-right of the screen):
    ```
    adb shell input tap 450 2250
    ```

- To simulate a swipe from the "Recent Apps" button to the middle of the screen (useful for opening the recent apps view):
    ```
    adb shell input swipe 450 2250 250 1150
    ```

- To simulate a swipe from the top to the middle of the screen (useful for pulling down the notification shade):
    ```
    adb shell input swipe 250 0 250 1150
    ```

- To simulate a swipe from the middle to the top of the screen (useful for pushing up the notification shade):
    ```
    adb shell input swipe 250 1150 250 0
    ```

- To simulate a long press on the "Home" button (useful for triggering Google Assistant or any other bound service):
    ```
    adb shell input swipe 250 2250 250 2250 2000
    ```

- To simulate a tap on the upper-middle-left of the screen (might be useful for some apps):
    ```
    adb shell input tap 125 575
    ```

- To simulate a tap on the upper-middle-right of the screen (might be useful for some apps):
    ```
    adb shell input tap 375 575
    ```

- To simulate a tap on the lower-middle-left of the screen (might be useful for some apps):
    ```
    adb shell input tap 125 1725
    ```

- To simulate a tap on the lower-middle-right of the screen (might be useful for some apps):
    ```
    adb shell input tap 375 1725
    ```

- To simulate a long press in the upper-middle-left of the screen:
    ```
    adb shell input swipe 125 575 125 575 2000
    ```

- To simulate a long press in the upper-middle-right of the screen:
    ```
    adb shell input swipe 375 575 375 575 2000
    ```

- To simulate a long press in the lower-middle-left of the screen:
    ```
    adb shell input swipe 125 1725 125 1725 2000
    ```

- To simulate a long press in the lower-middle-right of the screen:
    ```
    adb shell input swipe 375 1725 375 1725 2000
    ```

- To simulate a diagonal swipe from upper-middle-left to lower-middle-right of the screen:
    ```
    adb shell input swipe 125 575 375 1725
    ```

- To simulate a diagonal swipe from upper-middle-right to lower-middle-left of the screen:
    ```
    adb shell input swipe 375 575 125 1725
    ```

- To simulate a diagonal swipe from lower-middle-left to upper-middle-right of the screen:
    ```
    adb shell input swipe 125 1725 375 575
    ```

- To simulate a diagonal swipe from lower-middle-right to upper-middle-left of the screen:
    ```
    adb shell input swipe 375 1725 125 575
    ```

- To simulate a tap on the center of the status bar (useful for some quick settings):
    ```
    adb shell input tap 250 50
    ```

- To simulate a tap on the center of the antenna area (might be useful for some games or full-screen apps):
    ```
    adb shell input tap 250 2250
    ```

- To simulate a swipe from the center of the antenna area to the center of the screen (useful for some games or full-screen apps):
    ```
    adb shell input swipe 250 2250 250 1150
    ```

- To simulate a swipe from the center of the status bar to the center of the screen (useful for pulling down the notification shade):
    ```
    adb shell input swipe 250 50 250 1150
    ```

- To simulate a "zig-zag" swipe from the top-left to the bottom-right of the screen:
    ```
    adb shell input swipe 0 0 500 1150 & adb shell input swipe 500 1150 0 2300
    ```

- To simulate a "zig-zag" swipe from the top-right to the bottom-left of the screen:
    ```
    adb shell input swipe 500 0 0 1150 & adb shell input swipe 0 1150 500 2300
    ```

- To simulate a pinch gesture at the top of the screen:
    ```
    adb shell input swipe 125 300 250 600 & adb shell input swipe 375 300 250 600
    ```

- To simulate a spread gesture at the top of the screen:
    ```
    adb shell input swipe 250 600 125 300 & adb shell input swipe 250 600 375 300
    ```

- To simulate a pinch gesture at the bottom of the screen:
    ```
    adb shell input swipe 125 2000 250 1700 & adb shell input swipe 375 2000 250 1700
    ```

- To simulate a spread gesture at the bottom of the screen:
    ```
    adb shell input swipe 250 1700 125 2000 & adb shell input swipe 250 1700 375 2000
    ```

- To simulate a complex gesture (like drawing an "X" from corner to corner):
    ```
    adb shell input swipe 0 0 500 2300 & adb shell input swipe 500 0 0 2300
    ```
    
- To simulate a swipe from left to right across the screen (useful for going to the next item in a carousel):
    ```
    adb shell input swipe 0 1150 500 1150
    ```

- To simulate a swipe from right to left across the screen (useful for going to the previous item in a carousel):
    ```
    adb shell input swipe 500 1150 0 1150
    ```

- To simulate a tap on the "Back" button area (assuming it's at the bottom-left of the screen):
    ```
    adb shell input tap 50 2250
    ```

- To simulate a tap on the "Home" button area (assuming it's at the bottom-middle of the screen):
    ```
    adb shell input tap 250 2250
    ```

- To simulate a tap on the "Recent Apps" button area (assuming it's at the bottom-right of the screen):
    ```
    adb shell input tap 450 2250
    ```

- To simulate a swipe from the "Recent Apps" button to the middle of the screen (useful for opening the recent apps view):
    ```
    adb shell input swipe 450 2250 250 1150
    ```

- To simulate a swipe from the top to the middle of the screen (useful for pulling down the notification shade):
    ```
    adb shell input swipe 250 0 250 1150
    ```

- To simulate a swipe from the middle to the top of the screen (useful for pushing up the notification shade):
    ```
    adb shell input swipe 250 1150 250 0
    ```

- To simulate a long press on the "Home" button (useful for triggering Google Assistant or any other bound service):
    ```
    adb shell input swipe 250 2250 250 2250 2000
    ```

- To simulate a tap on the upper-middle-left of the screen (might be useful for some apps):
    ```
    adb shell input tap 125 575
    ```

- To simulate a tap on the upper-middle-right of the screen (might be useful for some apps):
    ```
    adb shell input tap 375 575
    ```

- To simulate a tap on the lower-middle-left of the screen (might be useful for some apps):
    ```
    adb shell input tap 125 1725
    ```

- To simulate a tap on the lower-middle-right of the screen (might be useful for some apps):
    ```
    adb shell input tap 375 1725
    ```

- To simulate a long press in the upper-middle-left of the screen:
    ```
    adb shell input swipe 125 575 125 575 2000
    ```

- To simulate a long press in the upper-middle-right of the screen:
    ```
    adb shell input swipe 375 575 375 575 2000
    ```

- To simulate a long press in the lower-middle-left of the screen:
    ```
    adb shell input swipe 125 1725 125 1725 2000
    ```

- To simulate a long press in the lower-middle-right of the screen:
    ```
    adb shell input swipe 375 1725 375 1725 2000
    ```

- To simulate a diagonal swipe from upper-middle-left to lower-middle-right of the screen:
    ```
    adb shell input swipe 125 575 375 1725
    ```

- To simulate a diagonal swipe from upper-middle-right to lower-middle-left of the screen:
    ```
    adb shell input swipe 375 575 125 1725
    ```

- To simulate a diagonal swipe from lower-middle-left to upper-middle-right of the screen:
    ```
    adb shell input swipe 125 1725 375 575
    ```

- To simulate a diagonal swipe from lower-middle-right to upper-middle-left of the screen:
    ```
    adb shell input swipe 375 1725 125 575
    ```

- To simulate a tap on the center of the status bar (useful for some quick settings):
    ```
    adb shell input tap 250 50
    ```

- To simulate a tap on the center of the antenna area (might be useful for some games or full-screen apps):
    ```
    adb shell input tap 250 2250
    ```

- To simulate a swipe from the center of the antenna area to the center of the screen (useful for some games or full-screen apps):
    ```
    adb shell input swipe 250 2250 250 1150
    ```

- To simulate a swipe from the center of the status bar to the center of the screen (useful for pulling down the notification shade):
    ```
    adb shell input swipe 250 50 250 1150
    ```

- To simulate a "zig-zag" swipe from the top-left to the bottom-right of the screen:
    ```
    adb shell input swipe 0 0 500 1150 & adb shell input swipe 500 1150 0 2300
    ```

- To simulate a "zig-zag" swipe from the top-right to the bottom-left of the screen:
    ```
    adb shell input swipe 500 0 0 1150 & adb shell input swipe 0 1150 500 2300
    ```

- To simulate a pinch gesture at the top of the screen:
    ```
    adb shell input swipe 125 300 250 600 & adb shell input swipe 375 300 250 600
    ```

- To simulate a spread gesture at the top of the screen:
    ```
    adb shell input swipe 250 600 125 300 & adb shell input swipe 250 600 375 300
    ```

- To simulate a pinch gesture at the bottom of the screen:
    ```
    adb shell input swipe 125 2000 250 1700 & adb shell input swipe 375 2000 250 1700
    ```

- To simulate a spread gesture at the bottom of the screen:
    ```
    adb shell input swipe 250 1700 125 2000 & adb shell input swipe 250 1700 375 2000
    ```

- To simulate a complex gesture (like drawing an "X" from corner to corner):
    ```
    adb shell input swipe 0 0 500 2300 & adb shell input swipe 500 0 0 2300
    ```

Absolutely, here are more examples for screen interactions on an Android device:

- To simulate a long press in the middle and then a swipe to the right (useful for triggering slide menus):
    ```
    adb shell input swipe 250 1150 400 1150 2000
    ```

- To simulate a long press in the middle and then a swipe to the left (useful for triggering slide menus):
    ```
    adb shell input swipe 250 1150 100 1150 2000
    ```

- To simulate a swipe from the center of the screen to the "Back" button (might be useful for some full-screen apps):
    ```
    adb shell input swipe 250 1150 50 2250
    ```

- To simulate a swipe from the center of the screen to the "Home" button (might be useful for some full-screen apps):
    ```
    adb shell input swipe 250 1150 250 2250
    ```

- To simulate a swipe from the center of the screen to the "Recent Apps" button (might be useful for some full-screen apps):
    ```
    adb shell input swipe 250 1150 450 2250
    ```

- To simulate a long press on the top-middle of the screen (might be useful for some apps):
    ```
    adb shell input swipe 250 300 250 300 2000
    ```

- To simulate a long press on the bottom-middle of the screen (might be useful for some apps):
    ```
    adb shell input swipe 250 2000 250 2000 2000
    ```

- To simulate a pinch gesture in the left-half of the screen:
    ```
    adb shell input swipe 0 1150 125 1150 & adb shell input swipe 250 1150 125 1150
    ```

- To simulate a spread gesture in the left-half of the screen:
    ```
    adb shell input swipe 125 1150 0 1150 & adb shell input swipe 125 1150 250 1150
    ```

- To simulate a pinch gesture in the right-half of the screen:
    ```
    adb shell input swipe 500 1150 375 1150 & adb shell input swipe 250 1150 375 1150
    ```

- To simulate a spread gesture in the right-half of the screen:
    ```
    adb shell input swipe 375 1150 500 1150 & adb shell input swipe 375 1150 250 1150
    ```

- To simulate a swipe in the shape of a square (might be useful for some games or apps):
    ```
    adb shell input swipe 125 875 375 875 & adb shell input swipe 375 875 375 1425 & adb shell input swipe 375 1425 125 1425 & adb shell input swipe 125 1425 125 875
    ```

- To simulate a swipe in the shape of a circle (might be useful for some games or apps):
    ```
    # This is a bit complex and might not be perfect, but it's a way to simulate a circular swipe:
    adb shell input swipe 250 1150 375 1150 500 & adb shell input swipe 375 1150 375 1425 500 & adb shell input swipe 375 1425 125 1425 500 & adb shell input swipe 125 1425 125 875 500 & adb shell input swipe 125 875 250 875 500
    ```

- To simulate a tap on the center of the screen with a delay (useful for timed inputs):
    ```
    adb shell input tap 250 1150; sleep 1
    ```

- To simulate multiple taps on the center of the screen with a delay between each (useful for timed inputs):
    ```
    adb shell input tap 250 1150; sleep 1; adb shell input tap 250 1150; sleep 1; adb shell input tap 250 1150
    ```

- To simulate a swipe from the middle to the left of the screen (useful for some slide menus):
    ```
    adb shell input swipe 250 1150 0 1150
    ```

- To simulate a swipe from the middle to the right of the screen (useful for some slide menus):
    ```
    adb shell input swipe 250 1150 500 1150
    ```

- To simulate a swipe from the left to the middle of the screen (useful for some slide menus):
    ```
    adb shell input swipe 0 1150 250 1150
    ```

- To simulate a swipe from the right to the middle of the screen (useful for some slide menus):
    ```
    adb shell input swipe 500 1150 250 1150
    ```

- To simulate a swipe from the "Back" button to the "Home" button (useful for some full-screen apps):
    ```
    adb shell input swipe 50 2250 250 2250
    ```

- To simulate a swipe from the "Home" button to the "Recent Apps" button (useful for some full-screen apps):
    ```
    adb shell input swipe 250 2250 450 2250
    ```

- To simulate a swipe from the "Recent Apps" button to the "Home" button (useful for some full-screen apps):
    ```
    adb shell input swipe 450 2250 250 2250
    ```

- To simulate a swipe from the "Home" button to the "Back" button (useful for some full-screen apps):
    ```
    adb shell input swipe 250 2250 50 2250
    ```

- To simulate a long press on the "Recent Apps" button (useful for some services):
    ```
    adb shell input swipe 450 2250 450 2250 2000
    ```

Simulate input events using the `input` command.

### Erase all text

```bash
adb shell input keyevent KEYCODE_MOVE_END
adb shell input keyevent |    --longpress $(printf 'KEYCODE_DEL %.0s' {1..250})
```

Start Calculator via Keyevent

```bash
adb shell input keyevent KEYCODE_CALCULATOR
```

Start Calendar via Keyevent

```bash
adb shell input keyevent KEYCODE_CALENDAR
```

Start Call Application via Keyevent

```bash
adb shell input keyevent KEYCODE_CALL
```

Start Camera via Keyevent

```bash
adb shell input keyevent KEYCODE_CAMERA
```

Press Caps Lock via Keyevent

```bash
adb shell input keyevent KEYCODE_CAPS_LOCK
```

Start Captions via Keyevent

```bash
adb shell input keyevent KEYCODE_CAPTIONS
```

Open Contacts Application via Keyevent

```bash
adb shell input keyevent KEYCODE_CONTACTS
```

Copy via Keyevent

```bash
adb shell input keyevent KEYCODE_COPY
```

Cut via Keyevent

```bash
adb shell input keyevent KEYCODE_CUT
```

Delete via Keyevent

```bash
adb shell input keyevent KEYCODE_DEL
```

EndCall via Keyevent

```bash
adb shell input keyevent KEYCODE_ENDCALL
```

Press END via Keyevent

```bash
adb shell input keyevent KEYCODE_END
```

Jump to the beginning of the line

```bash
adb shell input keyevent KEYCODE_DPAD_UP
```

Jump to the end of the line

```bash
adb shell input keyevent KEYCODE_DPAD_DOWN
```

Move the cursor one step to the left/right

```bash
adb shell input keyevent KEYCODE_DPAD_LEFT
adb shell input keyevent KEYCODE_DPAD_RIGHT


```

Press Home Button

```bash
adb shell input keyevent KEYCODE_HOME
```

Press + and - buttons

```bash
adb shell input keyevent KEYCODE_MINUS
adb shell input keyevent KEYCODE_PLUS
```

Press + in the numpad

```bash
adb shell input keyevent KEYCODE_NUMPAD_ADD
```

Press * button

```bash
adb shell input keyevent KEYCODE_NUMPAD_MULTIPLY
```

Press search key

```bash
adb shell input keyevent KEYCODE_SEARCH
```

Open Settings

```bash
adb shell input keyevent KEYCODE_SETTINGS
```

Press # button

```bash
adb shell input keyevent KEYCODE_NUMPAD_MULTIPLY
```

Start default music app

```bash
adb shell input keyevent KEYCODE_POUND
```

Mute Volume

```bash
adb shell input keyevent KEYCODE_MUTE
```

Open the notification bar and close

```bash
adb shell input keyevent KEYCODE_NOTIFICATION
adb shell input keyevent KEYCODE_NOTIFICATION
```

Cancel long press

```bash
adb shell input keyevent FLAG_CANCELED_LONG_PRESS
```

Open App Switch for changing applications

```bash
adb shell input keyevent KEYCODE_APP_SWITCH
```

Open Default Assistant

```bash
adb shell input keyevent KEYCODE_BRIGHTNESS_DOWN
adb shell input keyevent KEYCODE_BRIGHTNESS_UP
```

Select

```bash
adb shell input keyevent KEYCODE_BUTTON_SELECT
```

Swipe from top to bottom

```bash
adb shell input swipe 0 0 0 1000
```

Swipe from bottom to top

```bash
adb shell input swipe 0 1000 0 0
adb shell input swipe 100 4000 200 400
```

Swipe slower from bottom to top

```bash
adb shell input swipe 500 1000 0 0
```

Pinch out slowly

```bash
adb shell input swipe 100 100 20 20
```

Pinch out harder

```bash
adb shell input swipe 100 100 20 1000
```

Swipe your finger up and move window down

```bash
adb shell input swipe 100 1000 20 100
```

Simulate a swipe down for notification bar

```bash
adb shell input swipe 0 0 0 300 
```

Swipe and unlock the screen

```bash
adb shell input swipe 300 1000 300 500 
```

### Erase all text

```bash
adb shell input keyevent KEYCODE_MOVE_END
adb shell input keyevent |    --longpress $(printf 'KEYCODE_DEL %.0s' {1..250})
```

Start Calculator via Keyevent

```bash
adb shell input keyevent KEYCODE_CALCULATOR
```

Start Calendar via Keyevent

```bash
adb shell input keyevent KEYCODE_CALENDAR
```

Start Call Application via Keyevent

```bash
adb shell input keyevent KEYCODE_CALL
```

Start Camera via Keyevent

```bash
adb shell input keyevent KEYCODE_CAMERA
```

Press Caps Lock via Keyevent

```bash
adb shell input keyevent KEYCODE_CAPS_LOCK
```

Start Captions via Keyevent

```bash
adb shell input keyevent KEYCODE_CAPTIONS
```

Open Contacts Application via Keyevent

```bash
adb shell input keyevent KEYCODE_CONTACTS
```

Copy via Keyevent

```bash
adb shell input keyevent KEYCODE_COPY
```

Cut via Keyevent

```bash
adb shell input keyevent KEYCODE_CUT
```

Delete via Key

event

```bash
adb shell input keyevent KEYCODE_DEL
```

EndCall via Keyevent

```bash
adb shell input keyevent KEYCODE_ENDCALL
```

Press END via Keyevent

```bash
adb shell input keyevent KEYCODE_END
```

Jump to the beginning of the line

```bash
adb shell input keyevent KEYCODE_DPAD_UP
```

Jump to the end of the line

```bash
adb shell input keyevent KEYCODE_DPAD_DOWN
```

Move the cursor one step to the left/right

```bash
adb shell input keyevent KEYCODE_DPAD_LEFT
adb shell input keyevent KEYCODE_DPAD_RIGHT
```

Press the Grave (`) key

```bash
adb shell input keyevent KEYCODE_GRAVE
```

Press Home Button

```bash
adb shell input keyevent KEYCODE_HOME
```

Press + and - buttons

```bash
adb shell input keyevent KEYCODE_MINUS
adb shell input keyevent KEYCODE_PLUS
```

Press + in the numpad

```bash
adb shell input keyevent KEYCODE_NUMPAD_ADD
```

Press * button

```bash
adb shell input keyevent KEYCODE_NUMPAD_MULTIPLY
```

Press search key

```bash
adb shell input keyevent KEYCODE_SEARCH
```

Open Settings

```bash
adb shell input keyevent KEYCODE_SETTINGS
```

Press # button

```bash
adb shell input keyevent KEYCODE_NUMPAD_MULTIPLY
```

Start default music app

```bash
adb shell input keyevent KEYCODE_POUND
```

Mute Volume

```bash
adb shell input keyevent KEYCODE_MUTE
```

Open the notification bar and close

```bash
adb shell input keyevent KEYCODE_NOTIFICATION
adb shell input keyevent KEYCODE_NOTIFICATION
```

Cancel long press

```bash
adb shell input keyevent FLAG_CANCELED_LONG_PRESS
```

Open App Switch for changing applications

```bash
adb shell input keyevent KEYCODE_APP_SWITCH
```

Open Default Assistant

```bash
adb shell input keyevent KEYCODE_BRIGHTNESS_DOWN
adb shell input keyevent KEYCODE_BRIGHTNESS_UP
```

Select

```bash
adb shell input keyevent KEYCODE_BUTTON_SELECT
```

### Add a Contact, fill info, and press save on the device

```bash
adb shell am start -a android.intent.action.INSERT -t vnd.android.cursor.dir/contact -e name 'wuseman' -e phone '+467777701' -e email 'wuseman@nr1.nu' -e postal 'Street 10, New York'
```

Press save contact via shell from the above command

```bash
adb shell input keyevent 4
adb shell input keyevent 4 
```

```bash
adb shell am start -a android.intent.action.INSERT -t vnd.android.cursor.dir/contact -e name 'wuseman' -e phone '+46728999329' -e email 'wuseman@nr1.nu' 
```

## keytool

Keytool is a command-line tool for managing keys and certificates.

### Generate hash from keystore (typically used in Facebook)

```bash
keytool -exportcert -alias your_alias -keystore debug.keystore | openssl sha1 -binary | openssl base64 
```

Typically used in Google Maps

```bash
keytool -list -v -keystore ~/.android/debug.keystore -alias your_alias           
```

## logcat

Use the `logcat` command to view logs generated by the system and applications.

| Tag | Description |
|-----|------------|
| V   | Verbose (lowest

 priority)  |
| D   | Debug                      |
| I   | Info (default priority)    |
| W   | Warning                    |
| E   | Error                      |
| F   | Fatal                      |
| S   | Silent (highest priority, on which nothing is ever printed) |

Print the most recent lines since the specified time

```bash
adb logcat -t '01-26 20:52:41.820'
```

Print log only from a specific process ID

```bash
adb logcat --pid=<pid>
```

Log multiple options

```bash
adb logcat -b main -b radio -b events
```

Run all options at once

```bash
adb logcat -v brief \
    -v long \
    -v process \
    -v raw \
    -v tag \
    -v thread \
    -v threadtime \
    -v time \
    -v color
```

Print log from `lock_settings` only

```bash
adb logcat | grep "LockSettingsService|LockPatternUtilsKeyStorage|vold|vold|keystore2|keymaster_tee|LockSettingsService|vold_prepare_subdirs"
```

## pm

The `pm` command is used to manage packages on the device.

Disable AutoUpdate for any Package

```bash
adb shell pm disable-user --user 0 <package_name>
```

Disable AutoUpdate for all applications

```bash
adb shell pm disable-user com.android.vending
```

Print all applications in use

```bash
adb shell pm list packages | sed -e "s/package://" | while read x; do cmd package resolve-activity --brief $x | tail -n 1 | grep -v "No activity found"; done
```

List all packages installed on the device

```bash
adb shell pm list packages
```

List enabled packages

```bash
adb shell pm list packages -e
```

List disabled packages

```bash
adb shell pm list packages -d
```

List third-party packages installed by the user

```bash
adb shell pm list packages -3
```

List users

```bash
adb shell pm list users
```
### List permission groups

```bash
adb shell pm list permission-groups
```

### List features

```bash
adb shell pm list features
```

### Uninstall any installed package

```bash
adb shell pm uninstall --user 0 com.package.name
```

### Uninstall multiple apps

```sh
for packages in com.package1 com.package2; do
    adb shell pm uninstall --user 0 "${packages}"
done
```

### Clear application data

```bash
adb shell pm clear PACKAGE_NAME
```

### Grant permission to an app

```bash
adb shell pm grant com.application android.permission.READ_LOGS
```

### Revoke permission from an app

```bash
adb shell pm revoke com.application android.permission.READ_LOGS
```

### Reset all permissions for an app

```bash
adb shell pm reset-permissions -p your.app.package
```

## reboot

### Reboot system

```sh
adb reboot
```

### Reboot to recovery

```bash
adb reboot recovery
```

### Reboot to bootloader

```bash
adb reboot bootloader
```

### Reboot to fastboot (some brands)

```bash
adb reboot fastboot
```

## Android Shell Resources

- [Dirty Pagetable: A Novel Exploitation Technique To Rule Linux Kernel](https://yanglingxi1993.github.io/dirty_pagetable/dirty_pagetable.html)
- [Android™ Developer - Emulator Console](https://developer.android.com/studio/run/emulator-console)
- [Android™ Developer - Write your app](https://developer.android.com/studio/write)
- [Android™ Google Source - Source Code](https://android.googlesource.com/)
- [Android™ Google Source - CMD Command](https://android.googlesource.com/platform/frameworks/native/+/master/cmds/cmd/)
- [Android™ Generic Project](https://android-generic.github.io/#documentation)
- [Android™ GDB](https://source.android.com/devices/tech/debug/gdb)
- [Android™ Source - Understand Logging](https://source.android.com/devices/tech/debug/understanding-logging)
- [Android™ Source - Network Connectivity Tests](https://source.android.com/devices/tech/connect/connect_tests)
- [Android™ Platform Tools](https://android.googlesource.com/platform/prebuilts/cmdline-tools/+/34a182b3646de1051ea2c9b23132d073bcaa5087/tools/bin/)
- [Github Randorise - Mobile Hacking CheatSheet](https://github.com/randorisec/MobileHackingCheatSheet)
- [Mazhuang - Awesome ADB - Another Cheatsheet Wiki](https://mazhuang.org/awesome-adb/README.en.html)
- [Jfsso - Preferences Editor](https://github.com/jfsso/PreferencesEditor)
- [Nahamsec - Resources For Beginner - Bug Bounty Hunters](https://github.com/nahamsec/Resources-for-Beginner-Bug-Bounty-Hunters/blob/master/assets/mobile.md)
- [Raywenderlich - Forensic Artifacts](https://www.raywenderlich.com/3419415-hack-an-android-app-finding-forensic-artifacts#toc-anchor-002)
- [Noobsec - Bypass Fingerprint Lock In Just 1 Second](https://noobsec.org/project/2019-12-22-bypass-fingerprint-lock-in-just-1-second/)
- [Noobsec - Cara Reverse Engineering](https://noobsec.org/project/2018-11-04-cara-reverse-engineering-apk/)
- [Oracle - JVMS](https://docs.oracle.com/javase/specs/jvms/se9/html/jvms-4.html)
- [Tjtech - Analyze OEM Unlocking Under Android](http://tjtech.me/analyze-oem-unlocking-under-android.html)
- [U'Smile - How to change the IMEI on Android devices](https://usmile.at/blog/how-to-change-imei-on-android-devices)
- [Android™ Q Navigation - Gesture Controls](https://www.xda-developers.com/android-q-navigation-gesture-controls/#fitvid892986)
- [Good review for clipboard control](https://www.smartspate.com/how-to-copy-text-from-the-clipboard-to-android-devices/)
- [Utilizing adb for daily tasks](https://www.droidcon.com/2021/10/21/utilizing-adb-for-daily-tasks/)


## Greetings

Greetings to all my esteemed colleagues who have placed their trust in me and granted me the opportunity to explore various devices. Your unwavering support is deeply appreciated, and I hold great affection for each one of you.

<br>
<p align="center">Learn with ❤️ By <a href="https://www.youtube.com/channel/UC2O1Hfg-dDCbUcau5QWGcgg">Linuxndroid</a></p>


# Follow Me on :

[![Instagram](https://img.shields.io/badge/IG-linuxndroid-yellowgreen?style=for-the-badge&logo=instagram)](https://www.instagram.com/linuxndroid)

[![Youtube](https://img.shields.io/badge/Youtube-linuxndroid-redgreen?style=for-the-badge&logo=youtube)](https://www.youtube.com/channel/UC2O1Hfg-dDCbUcau5QWGcgg)

[![Browser](https://img.shields.io/badge/Website-linuxndroid-yellowred?style=for-the-badge&logo=browser)](https://www.linuxndroid.com)


