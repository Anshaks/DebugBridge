<img src="DebugBridge%20icon.png" alt="DebugBridge icon" width="120" height="120">

A phone-to-phone **ADB and Fastboot toolkit** for Android. Connect a target phone to your controller phone with a USB OTG adapter and run commands from two independent terminals, with no PC needed.

> ### ⚠️ Notice
> DebugBridge uses its **own native ADB and Fastboot engines**, not the official desktop Platform Tools.
>
> - **Not every ADB or Fastboot command is guaranteed to work.** Some commands may work, some may fail, and some are not implemented at all.
> - Results depend on your target device, its bootloader, its Android version, the USB mode and your OTG adapter.
> - Unsupported or rejected commands return the real error from the engine or the device. Nothing is faked.
> - The app has **not been tested on physical hardware** yet. Please report problems in Issues.

## Download

**[⬇ Download APK](https://github.com/Anshaks/DebugBridge/releases/tag/Apk)**

## Screenshots

<p align="center">
  <img src="assets/adb.png" width="30%" alt="ADB terminal">
  <img src="assets/fastboot.png" width="30%" alt="Fastboot terminal">
  <img src="assets/sideload.png" width="30%" alt="ADB sideload">
</p>

## Features

- **ADB Terminal** (monster icon) and **Fastboot Terminal** (lightning icon), each with its own input, history, output and run state
- Native ADB and Fastboot protocol engines inside the app (no desktop binaries, no downloads at runtime)
- USB OTG detection with the official Android USB permission dialog
- Real ADB authorization (the target shows the standard "Allow USB debugging" prompt)
- File picker (system picker, no storage permission) that inserts the file path into the command line
- **Flash Recovery** section (permanent flash or temporary boot)
- **ADB Sideload** section with a progress bar
- Terminal status line: real exit code, elapsed time, output line count

## Requirements

- Controller phone: Android 7.0+ (API 24) with USB host (OTG) support
- A working OTG adapter and data cable
- Target phone with **USB debugging** enabled (for ADB), or booted into bootloader (for Fastboot) or recovery (for sideload)

## Play Protect warning (this is expected)

When you install DebugBridge, Google Play Protect may show **"Blocked by Play Protect"** or **"This app is harmful / unsafe"**. Don't panic, this is a common warning for apps like this, because:

- the app is installed from outside the Play Store,
- it is not published by a Play-verified developer,
- it can talk to other phones over USB and run ADB/Fastboot commands (flashing, unlocking), which Play Protect treats as sensitive.

### What to do

1. Tap **More details → Install anyway** (or **Install without scanning**). This is the best option.
2. If your phone does not offer that button and blocks the install completely, temporarily turn off Play Protect: **Play Store → profile icon → Play Protect → Settings (gear) → turn off "Scan apps with Play Protect"**.
3. Install the app, then **turn Play Protect back on**.
4. Before running any command, open Play Protect and tap **Scan**. Play Protect will scan DebugBridge along with your other apps.

If the result says **"No harmful apps found"**, then Play Protect found nothing wrong with the app. This is reassuring, but it is not a guarantee. The app has no INTERNET permission and the source and build notebook are public, so you can check them yourself.

Only install the APK from the official link in this README, and sign it with your own key first.

## How OTG works

1. Plug the target phone into the controller phone through the OTG adapter.
2. Android shows a USB access dialog on the controller. Accept it.
3. For ADB, the target shows "Allow USB debugging". Accept it.
4. The status line at the top shows the real state (`ADB · device`, `ADB · sideload`, `Fastboot`, ...). Tap the status line to rescan or retry.
5. If the target reboots (for example `adb reboot bootloader`), it reconnects as a new USB device and the app switches to the matching mode automatically.

The ADB terminal only works while the target is in ADB mode. The Fastboot terminal only works in bootloader mode. If the mode is wrong, the terminal says so.

## Terminals

There is no fixed command whitelist. The text you type is parsed by the engine, and unsupported commands return a real error. Typing `adb` or `fastboot` before a command is optional.

### ADB terminal

| Command | Notes |
|---|---|
| `devices [-l]`, `get-state`, `get-serialno`, `features`, `wait-for-device` | Device info |
| `shell <anything>` | Text goes to the device shell **unmodified**, so quotes and spaces work as on the device |
| `shell` (no arguments) | Interactive shell. Type lines at the prompt, `exit` to leave, Stop to detach |
| `exec-out <cmd>`, `logcat [args]` | Streaming output, use Stop to end |
| `push <local>... <remote>`, `pull <remote> [local]` | Single files, real sync protocol |
| `install [opts] file.apk`, `uninstall [opts] pkg` | Install pushes to `/data/local/tmp` and runs `pm install` |
| `reboot [bootloader\|recovery\|sideload]`, `root`, `unroot`, `remount`, `usb`, `tcpip <port>`, `disable-verity`, `enable-verity` | Device services |
| `sideload <file.zip>` | Needs a recovery in sideload mode |
| `raw <service>` | Opens **any** device service string, e.g. `raw shell:id` |

Exit codes are real when the device supports `shell_v2`. Otherwise the status line says the exit code is unavailable.

### Fastboot terminal

| Command | Notes |
|---|---|
| `devices`, `getvar <name\|all>` | Device info |
| `flash <partition> <file>`, `boot <image>`, `erase <partition>` | Image is downloaded first, then the real command runs |
| `reboot [bootloader\|recovery\|fastboot]`, `continue`, `set_active <slot>` | Boot control |
| `oem <args>`, `flashing <args>` | Passed to the bootloader as typed (e.g. `oem unlock`, `flashing unlock`) |
| `create-/resize-/delete-logical-partition`, `snapshot-update` | Dynamic partitions |
| `raw <string>` | Sends **any** command string to the bootloader (max 64 bytes) |

### Relative paths and the file picker

The 📁 button copies the chosen file into the app's work folder and inserts its full path (quoted if it has spaces). Relative paths in commands refer to that folder. `pull` saves into the app's external files folder and prints the exact path.

### Safety prompts

Destructive-looking commands (flash, erase, flashing/oem, raw, `rm`/`dd` in shell, and similar) show a confirmation first. This is a warning only. The command still runs exactly as typed after you confirm.

## Flash Recovery

1. Open the Fastboot tab, expand **Flash Recovery**, select a recovery `.img`.
2. The app checks the file header and asks the bootloader whether a `recovery` partition exists.
3. Choose **Permanent flash** (`flash recovery`) or **Temporary boot** (`boot`).

The bootloader must be **unlocked**. Devices without a recovery partition (many A/B devices) or bootloaders that don't support `boot` will return a real error.

## ADB Sideload

1. Boot the target into recovery and choose **Apply update from ADB**.
2. In the ADB tab, expand **ADB Sideload**, select the ZIP, press **Start sideload**.
3. The file streams straight from storage with a real progress bar.

"Transfer complete" only means the device received the whole file. **It does not confirm installation.** Read the result on the recovery screen. Some recoveries accept only signed ZIPs.

## Bootloader unlock

The Fastboot terminal sends `oem unlock` and `flashing unlock` (and `flashing lock`, `flashing unlock_critical`, ...) to the bootloader exactly as typed.

- Enable **OEM unlocking** in Developer options on the target first.
- Unlocking **erases all user data** on most devices.
- Newer devices usually use `flashing unlock`; older ones use `oem unlock`.
- Some manufacturers (for example Xiaomi, Huawei and others) require their own account, waiting period or official tool. DebugBridge cannot bypass that, and the bootloader will reject the command.

## Limitations

- Not implemented: desktop adb server commands (`forward`, `backup`, TCP pairing), directory push/pull, split APK install, sparse-splitting for images larger than the bootloader's `max-download-size`, `fastboot -w/--slot/flashall/update/format`.
- Anything else the device supports can still be reached through `shell`, `raw` (ADB) and `raw` (Fastboot).
- Behavior depends on the target device, its USB mode and its bootloader. Universal compatibility is not guaranteed.
- No INTERNET permission is requested.

## Build from source

Upload `DebugBridge_Builder.ipynb` to Google Colab and press **Runtime → Run all**. It installs JDK 17, the Android SDK and Gradle, generates the whole project, builds an unsigned APK, verifies it is unsigned, and downloads it.

## Disclaimer

**The developer is not responsible for any damage, data loss or misuse resulting from this app.**

- DebugBridge is provided **"as is"**, without warranty of any kind, express or implied. This includes fitness for a particular purpose and compatibility with any device.
- Flashing, erasing, unlocking the bootloader, rooting and similar operations can **erase all data, brick a device, void its warranty or break security features**. You run every command at your own risk.
- The developer is **not responsible** for bricked or damaged devices, lost data, warranty loss, or any direct or indirect damage caused by using this app or any command executed through it.
- The app has **not been tested on physical hardware** across devices. Behaviour differs between devices, bootloaders and Android versions.
- Use DebugBridge **only on devices you own or have explicit permission to work on**. Using it on someone else's device without permission may be illegal. You alone are responsible for how you use it.
- Back up your data before flashing or erasing anything.
- By downloading or using this app, you agree to these terms. If you do not agree, do not use it.
