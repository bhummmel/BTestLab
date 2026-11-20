# Skylight Calendar Android Integration and ADB Troubleshooting

## Overview

The Skylight Calendar Android Integration project investigated repurposing a proprietary smart calendar display into a more flexible, locally managed household information terminal.

The project centered on a Skylight Calendar 15-inch device, identified in the existing system information as a `150-CAL` model with a firmware identifier beginning `skylightOS-rk3562_15_inch`.

Although the device was sold as a dedicated family calendar, its underlying Android operating system suggested that it might support additional applications, alternative launchers, and integration with the existing Home Assistant infrastructure.

The primary objective was to reduce dependence on Skylight's proprietary application environment and investigate whether the display could serve as a general-purpose household dashboard.

The proposed functionality included Google Calendar and Apple Calendar synchronization, Home Assistant controls, locally hosted calendar software, and potentially a custom Android launcher.

The investigation used a desktop computer running Bazzite Linux to access the Android device through Android Debug Bridge (ADB).

Developer Options and USB debugging were successfully enabled on the calendar.

However, the computer did not detect an authorized Android device. Repeated `adb devices` commands returned an empty device list, and an attempt to launch Android's Settings application through ADB failed.

The project also evaluated community documentation for accessing Android settings and installing alternative applications on Skylight hardware.

An important limitation was that the referenced community walkthrough targeted the larger Skylight Calendar Max, not necessarily the 15-inch device being investigated.

The project demonstrates experience with Android debugging, Linux USB device management, proprietary hardware evaluation, self-hosted application architecture, and smart home integration.

**Documented status:** Investigation and troubleshooting. Successful ADB connectivity, root access, custom launcher installation, and complete Home Assistant integration were not established in the original project record.

---

## Project Objectives

The project aimed to transform a dedicated smart calendar into a flexible home automation interface.

Specific objectives included:

- Investigating the Skylight Calendar's Android operating system.
- Identifying the device model and firmware.
- Accessing Android Developer Options.
- Enabling USB debugging.
- Establishing an ADB connection from Bazzite Linux.
- Evaluating alternative Android launchers.
- Investigating application sideloading.
- Evaluating third-party application stores.
- Exploring Home Assistant dashboard integration.
- Supporting Google Calendar synchronization.
- Supporting Apple Calendar synchronization.
- Investigating self-hosted calendar applications.
- Reducing reliance on proprietary cloud services.
- Preserving touchscreen functionality.
- Maintaining the ability to restore the original Skylight interface.
- Avoiding unnecessary firmware modifications.
- Developing a repeatable troubleshooting procedure.

The broader objective was to reuse existing hardware rather than purchasing a separate Android tablet or dedicated Home Assistant control panel.

---

## Infrastructure Environment

| Component | Role |
|---|---|
| Skylight Calendar 15-inch | Target Android display |
| Bazzite Linux | Development workstation |
| Android Debug Bridge | Android device communication |
| Android Studio | Android development and debugging environment |
| USB | Physical device connection |
| Home Assistant | Smart home automation platform |
| Unraid | Self-hosted application server |
| Docker Compose | Potential calendar service deployment |
| Google Calendar | External calendar source |
| Apple Calendar | External calendar source |
| Android launcher | Potential replacement home interface |
| Kiosk browser | Potential dashboard frontend |
| OPNsense | Local network routing and firewall |

The existing homelab already hosted Home Assistant on Unraid.

This made it possible to investigate a thin-client architecture in which the Skylight device would display applications or dashboards hosted elsewhere on the local network.

---

## Hardware Identification

The target device was identified as a Skylight Calendar 15-inch model.

Previously observed identifiers included:

```text
Model: 150-CAL

Firmware identifier:
skylightOS-rk3562_15_inch
```

The firmware identifier suggests an Android-based Rockchip platform, but it is not sufficient by itself to establish the exact system-on-chip, bootloader, or firmware layout.

Community research has identified multiple hardware and firmware variants of Skylight calendar displays. Consequently, instructions developed for one model should not automatically be applied to another.

This distinction became important when evaluating an ADB walkthrough written for the 27-inch Skylight Calendar Max.

---

## Existing Device Architecture

The Skylight Calendar combines touchscreen hardware, an Android operating system, and a proprietary calendar interface.

Conceptually, the system operates as follows:

```text
                 Skylight Cloud Services
                           |
                           v
                    Internet Connection
                           |
                           v
                 +-------------------+
                 | Skylight Calendar |
                 |                   |
                 | Android OS        |
                 | Skylight App      |
                 | Touchscreen UI    |
                 +-------------------+
```

The project investigated replacing or supplementing the proprietary application layer.

The goal was not necessarily to replace Android itself.

A less invasive solution would retain the existing operating system while installing a different launcher, browser, or Home Assistant application.

---

## Proposed Local-First Architecture

The proposed architecture used the Skylight device primarily as a display and interaction terminal.

```text
                 Google Calendar
                        |
                        |
                 Apple Calendar
                        |
                        v
              Calendar Integration
                        |
                        v
                 Unraid Server
                        |
          +-------------+-------------+
          |                           |
          v                           v
    Home Assistant              Calendar Service
          |                           |
          +-------------+-------------+
                        |
                  Local Network
                        |
                        v
                Skylight Calendar
                        |
                 Android Browser
                        |
                        v
                Household Dashboard
```

This architecture would allow the server to manage calendar data and smart home integrations.

The Skylight hardware would provide the touchscreen interface.

The principal advantage was separating application functionality from the proprietary device software.

---

## Why Android Was Important

Android provided a potential route to installing software outside the manufacturer's intended interface.

Possible applications included:

- Home Assistant Companion.
- A Chromium-based browser.
- A kiosk browser.
- Alternative Android launchers.
- Calendar applications.
- Local dashboard clients.
- Remote management utilities.

However, an Android operating system does not automatically guarantee unrestricted application installation.

Manufacturers may restrict system settings, USB debugging, package installation, and launcher selection.

The project therefore focused first on obtaining a reliable debugging connection.

---

## Android Debug Bridge

Android Debug Bridge is a command-line tool used to communicate with Android devices.

ADB supports operations such as:

- Listing connected devices.
- Opening an interactive shell.
- Installing APK packages.
- Launching Android activities.
- Reading system logs.
- Transferring files.
- Rebooting devices.
- Inspecting system properties.

A typical ADB connection consists of three components:

```text
               Linux Workstation
                       |
                 ADB Client
                       |
                  ADB Server
                       |
                 USB or TCP/IP
                       |
                       v
                  Android Device
                       |
                    adbd
                       |
                 Android System
```

The Android device must expose a compatible debugging transport and authorize the host before normal shell commands can be executed.

Google's official ADB documentation describes the device authorization and debugging requirements. 

---

## Accessing Android Developer Options

The project successfully reached Android settings and enabled USB debugging.

On standard Android devices, Developer Options can usually be enabled by repeatedly selecting the build number in the device information screen.

A conventional procedure is:

1. Open Android Settings.
2. Navigate to the device information screen.
3. Locate the build number.
4. Tap the build number seven times.
5. Return to Developer Options.
6. Enable USB debugging.

The exact navigation path depends on the manufacturer's Android build.

The Skylight interface made reaching standard Android settings more complicated than on an ordinary tablet.

Nevertheless, USB debugging was enabled during the original investigation.

---

## Community ADB Walkthrough

A community project titled `SkylightMaxCalendarADB` documented a method for accessing Android settings and installing applications on the Skylight Calendar Max.

The walkthrough included:

- Accessing Android system settings.
- Enabling Developer Options.
- Enabling USB debugging.
- Connecting a computer through USB.
- Installing an alternative launcher.
- Installing a navigation bar.
- Installing Aurora Store.
- Changing the default home application.

The repository also demonstrated using ADB to launch Android Settings directly.

However, the documented procedure was developed for the Calendar Max. Its applicability to the smaller 15-inch model was uncertain. <Cite ref="turn313713search0"/>

The project emphasized verifying device compatibility before applying model-specific procedures.

---

## Bazzite Linux Development Environment

The workstation used for the investigation ran Bazzite Linux.

Bazzite is a Fedora Atomic-based distribution with an image-oriented operating system architecture.

This affected how development tools could be installed and maintained.

Potential installation methods included:

- Existing host utilities.
- Flatpak applications.
- Toolbox or Distrobox environments.
- Supported package layering.
- Standalone Android SDK platform tools.

The project investigated Android Studio installation on Bazzite.

However, Android Studio itself was not necessary for every ADB troubleshooting operation.

The Android SDK Platform Tools package provides the `adb` executable independently.

---

## Checking the ADB Installation

The first step in troubleshooting an ADB connection is verifying that the executable is available.

```bash
command -v adb
```

The installed version can be checked using:

```bash
adb version
```

A successful result confirms that the shell can locate the ADB executable.

It does not establish that an Android device is connected.

The project emphasized separating host tool installation from device communication.

---

## Initial ADB Connection Attempt

After enabling USB debugging, the Skylight Calendar was connected to the Bazzite workstation.

The following command was executed:

```bash
adb devices
```

The output was:

```text
List of devices attached
```

No device identifier appeared beneath the heading.

The command was repeated several times, but the device list remained empty.

This was the principal confirmed technical obstacle.

An empty list indicates that the ADB server has not discovered an available device through its current transports.

It differs from a device appearing as `unauthorized` or `offline`.

---

## Attempting to Open Android Settings

The project also attempted to launch Android's Settings application through ADB.

The command was:

```bash
adb shell am start -a android.settings.SETTINGS
```

The resulting error was:

```text
adb: no devices/emulators found
```

This confirmed that the workstation did not have an available ADB target.

The command itself was a valid approach for launching Android Settings on a connected device.

However, ADB could not execute it without a functioning connection.

---

## Understanding ADB Connection States

ADB can report several different connection states.

| Result | Interpretation |
|---|---|
| Empty device list | No ADB transport discovered |
| `unauthorized` | Device detected but host authorization pending |
| `offline` | Transport exists but communication is not ready |
| `device` | Authorized device available |
| `no devices/emulators found` | Command cannot locate an available target |

These states require different troubleshooting approaches.

For example, accepting an authorization prompt may resolve an `unauthorized` device, but it cannot resolve a USB connection that is not being enumerated at all.

The project emphasized diagnosing the actual connection state before changing settings.

---

## USB Debugging Versus USB Connectivity

Enabling USB debugging is only one requirement for an ADB connection.

The physical connection must also support data transfer.

Potential obstacles include:

- A charge-only USB cable.
- An incompatible USB port.
- Incorrect USB connection mode.
- A USB port intended only for peripheral attachment.
- Missing USB device enumeration.
- Linux permissions.
- An inactive ADB daemon.
- Device firmware restrictions.
- A hardware-specific debugging interface.

The project emphasized that a powered device and a connected cable do not necessarily establish a USB data connection.

---

## Inspecting USB Devices on Linux

The next logical diagnostic step is to determine whether Linux detects the Skylight hardware.

A standard command is:

```bash
lsusb
```

The command lists USB devices visible to the Linux host.

For more detailed information:

```bash
lsusb -t
```

This displays the USB device tree and interface information.

The project emphasized comparing the device list before and after connecting the calendar.

If no new device appears, troubleshooting should focus on the physical connection, USB port role, and device firmware behavior before changing ADB permissions.

---

## Monitoring USB Connection Events

Linux kernel logs can reveal USB enumeration activity.

A useful diagnostic command is:

```bash
sudo journalctl -kf
```

The command follows kernel log messages.

The calendar can then be disconnected and reconnected while observing the output.

Potential messages include:

- New USB device detected.
- Device descriptor errors.
- USB configuration failures.
- Device disconnection.
- USB interface initialization.

These messages help determine whether the operating system recognizes any activity from the connected hardware.

The original project history does not preserve a confirmed Skylight USB enumeration result.

---

## USB Cable Verification

A USB cable may provide power without carrying data.

Therefore, one of the simplest troubleshooting steps is to test a known-good data cable.

A reliable test cable should be verified with another Android device or USB peripheral.

The project emphasized avoiding assumptions based on connector appearance.

USB-C connectors can be used for charging, data transfer, display output, and other functions, but the supported capabilities depend on the cable and connected hardware.

---

## USB Port Role

USB ports can operate in different roles.

A port may be configured as:

- USB host.
- USB peripheral.
- Dual-role USB.
- Charging-only.
- Service or maintenance interface.

For ADB over USB, the Android device normally needs to expose an appropriate peripheral debugging interface to the host computer.

A port that functions only as a USB host may not provide this behavior.

The project emphasized verifying the actual capabilities of the Skylight's USB connection rather than assuming that every physical port supports debugging.

---

## Restarting the ADB Server

ADB uses a background server process on the workstation.

The server can be restarted using:

```bash
adb kill-server
```

Followed by:

```bash
adb start-server
```

Then:

```bash
adb devices -l
```

This can resolve some stale transport or server-state problems.

However, restarting the ADB server cannot compensate for a missing physical USB data connection.

Google's Android development documentation recommends restarting the ADB server as part of device connection troubleshooting. <Cite ref="turn313713search13"/>

---

## Linux Device Permissions

Linux uses device permissions and udev rules to control access to USB devices.

A device may be visible to the kernel but inaccessible to the current user.

This can prevent ADB from communicating with the device.

On conventional Linux distributions, Android development tools may require appropriate udev rules or group membership.

Google's Android development documentation describes Linux-specific permission requirements. <Cite ref="turn313713search9"/>

The project emphasized checking permissions only after confirming that the device is actually enumerating.

---

## Bazzite-Specific Permission Considerations

Bazzite's atomic system architecture introduces additional considerations when installing system-level development utilities.

For example:

- Development tools may run inside a containerized environment.
- USB devices may not automatically be exposed to every container.
- System directories may have different persistence characteristics.
- Traditional package installation procedures may not apply directly.

A practical diagnostic approach is to use the host's ADB executable when available.

If ADB is running inside Toolbox, Distrobox, or another container, USB device access must be verified separately.

The project emphasized understanding where the debugging tool actually executes.

---

## Wireless ADB as an Alternative

Some Android versions support wireless debugging.

Depending on the firmware, this may allow a computer to connect without a USB cable.

A conventional Android wireless debugging workflow can involve:

```bash
adb pair DEVICE_IP:PAIRING_PORT
```

Followed by:

```bash
adb connect DEVICE_IP:ADB_PORT
```

The pairing port and connection port are supplied by the Android device.

They should not be assumed to be the same.

The project considered wireless debugging as a potential alternative to USB.

However, the available history does not establish that the Skylight firmware exposed a usable wireless debugging interface.

---

## Security of Wireless Debugging

ADB provides powerful access to an Android device.

Wireless debugging should therefore be restricted to trusted networks.

Recommended precautions include:

- Enabling debugging only when necessary.
- Avoiding exposure to untrusted Wi-Fi networks.
- Disabling debugging after maintenance.
- Reviewing authorized debugging hosts.
- Avoiding public exposure of ADB ports.
- Protecting the device from unauthorized physical access.

The project emphasized that debugging access is an administrative capability rather than an ordinary application service.

---

## Application Sideloading

Once an authorized ADB connection is established, Android applications can potentially be installed using APK files.

A conventional installation command is:

```bash
adb install application.apk
```

This requires a compatible application package and an Android environment that permits installation.

Potential applications included:

- Alternative launchers.
- Home Assistant Companion.
- Kiosk browsers.
- Calendar applications.
- Local dashboard clients.

The project did not establish that APK installation succeeded on the Skylight device.

---

## Alternative Android Launchers

A custom launcher could allow the calendar to behave more like a conventional Android tablet.

Possible benefits include:

- Access to installed applications.
- A configurable home screen.
- Easier navigation.
- Direct Home Assistant access.
- Reduced dependence on the original Skylight interface.

The referenced community walkthrough used Nova Launcher as an example.

It also described installing a navigation bar application to improve navigation on a device without conventional Android navigation controls. <Cite ref="turn313713search0"/>

The project emphasized preserving a path back to the original application whenever possible.

---

## Home Assistant Companion Application

One potential use for the device was running the Home Assistant Companion application.

This would allow the calendar to display Home Assistant dashboards and interact with configured smart home entities.

Potential capabilities included:

- Lighting control.
- Thermostat control.
- Household status.
- Calendar entities.
- Weather information.
- Media playback controls.
- Automation triggers.
- Presence information.

The exact capabilities would depend on the Android version, application compatibility, and device permissions.

The project did not establish that Home Assistant Companion was installed successfully.

---

## Browser-Based Home Assistant Dashboard

A less invasive alternative was to use a browser as the primary interface.

The architecture would be:

```text
                Unraid Server
                     |
                Home Assistant
                     |
                  HTTP/HTTPS
                     |
                     v
              Android Browser
                     |
                     v
             Skylight Touchscreen
```

This approach would avoid requiring a dedicated Android application for every service.

The browser would display a Home Assistant dashboard hosted on the local network.

A kiosk browser could potentially provide full-screen operation.

The project emphasized that browser-based dashboards are easier to maintain centrally than locally installed custom applications.

---

## Calendar Synchronization Requirements

The original project also investigated using the display as a household calendar.

The desired sources included:

- Google Calendar.
- Apple Calendar.
- Potentially other CalDAV-compatible services.

The objective was to present events from multiple accounts through one interface.

This could be accomplished through a calendar application, a Home Assistant dashboard, or a separate self-hosted calendar service.

The project emphasized that calendar synchronization and touchscreen interface customization are separate technical problems.

---

## Home Assistant Calendar Integration

Home Assistant supports calendar entities and integrations.

These can provide calendar event information for dashboards and automations.

A conceptual arrangement is:

```text
               Google Calendar
                      |
                      v
                 Home Assistant
                      ^
                      |
                Apple Calendar
                      |
                      v
                Calendar Entities
                      |
                      v
                Custom Dashboard
                      |
                      v
                Skylight Display
```

The exact integration method depends on the calendar provider and authentication requirements.

The project considered this a practical way to consolidate calendar data without developing an entirely new calendar application.

---

## Self-Hosted Calendar Alternatives

The project also considered hosting calendar software on Unraid.

Potential approaches included:

- A CalDAV server.
- A web-based calendar frontend.
- A family scheduling dashboard.
- A Home Assistant calendar interface.
- A locally hosted scheduling application.

A self-hosted calendar server could provide centralized storage and access through multiple devices.

However, synchronizing Apple and Google calendars bidirectionally may require additional integration components.

The project emphasized that hosting a calendar service locally does not automatically provide seamless synchronization with proprietary cloud calendar providers.

---

## Docker-Based Calendar Architecture

A self-hosted calendar service could operate as a Docker container on Unraid.

A conceptual architecture is:

```text
                    Unraid
                      |
                Docker Engine
                      |
               Calendar Service
                      |
                 CalDAV / Web
                      |
                Local Network
                      |
             +--------+--------+
             |                 |
             v                 v
       Skylight Display    Other Devices
```

The Skylight would function as a client rather than hosting the calendar database itself.

This would simplify backups and allow calendar functionality to remain independent of the display hardware.

The project did not establish that a specific self-hosted calendar platform was deployed.

---

## Evaluating Existing Community Projects

The investigation included reviewing community work involving Skylight hardware.

The `SkylightMaxCalendarADB` project documented accessing Android settings and installing applications on the larger Calendar Max.

Other community research has explored modifying 15-inch Skylight devices and replacing their original interface with custom dashboards.

For example, the `photo-frame-dashboard` project documents hardware-specific investigation, ADB access, backups, and alternative interfaces for supported Android frames, including certain Skylight variants. <Cite ref="turn313713search3"/>

These projects provide useful reference material, but their procedures must be matched to the exact hardware and firmware.

---

## Firmware Compatibility Risks

Proprietary Android devices may use different firmware even when their external appearance is similar.

Potential differences include:

- Bootloader configuration.
- Partition layout.
- USB controller configuration.
- Android version.
- Kernel drivers.
- Display controller.
- Touchscreen driver.
- Device tree.
- Recovery environment.

Applying a firmware image or bootloader procedure intended for another variant can render the device unusable.

The project emphasized verifying hardware identifiers and obtaining a reliable backup before attempting low-level modifications.

---

## Root Access Versus ADB Access

ADB and root access are not equivalent.

ADB may allow an authorized user to install applications and execute commands within Android's normal security restrictions.

Root access provides elevated privileges and may permit modification of protected system components.

The original project focused on establishing ADB connectivity.

Root access was a possible future direction, not a confirmed achievement.

The project emphasized attempting the least invasive solution that could satisfy the functional requirements.

---

## Preserving the Original Skylight Functionality

A major consideration was maintaining the ability to use the original Skylight interface.

Potential approaches included:

- Installing a secondary launcher.
- Using a browser without replacing system applications.
- Retaining the original Skylight package.
- Avoiding modifications to boot partitions.
- Backing up important configuration.
- Documenting changes before applying them.

The project emphasized that a reversible application-level modification is preferable to firmware replacement when both can accomplish the same goal.

---

## Troubleshooting Methodology

The investigation can be organized into the following stages.

### Stage 1: Identify the Hardware

Record the model, firmware, Android version, and available ports.

### Stage 2: Enable Developer Options

Confirm access to Android settings.

### Stage 3: Enable USB Debugging

Verify that debugging is enabled.

### Stage 4: Verify ADB Installation

Confirm that the Bazzite workstation can execute ADB.

### Stage 5: Inspect USB Enumeration

Use `lsusb` and kernel logs to determine whether the device is visible.

### Stage 6: Test a Known-Good Data Cable

Eliminate charge-only cables and defective connections.

### Stage 7: Verify USB Port Functionality

Determine whether the selected port supports the required device role.

### Stage 8: Restart the ADB Server

Clear stale server state.

### Stage 9: Check Linux Permissions

Investigate udev rules and user access if the device enumerates but remains inaccessible.

### Stage 10: Investigate Wireless Debugging

Determine whether the Android firmware exposes a supported wireless debugging interface.

### Stage 11: Establish an Authorized ADB Connection

Confirm that `adb devices` reports the device as available.

### Stage 12: Test a Non-Destructive Command

For example:

```bash
adb shell getprop ro.build.version.release
```

### Stage 13: Evaluate Application Installation

Test compatible applications without modifying protected system partitions.

### Stage 14: Configure the Dashboard

Integrate Home Assistant or another calendar frontend.

### Stage 15: Validate Recovery

Confirm that the original Skylight interface remains accessible.

This methodology prioritizes hardware identification and transport diagnostics before attempting software modifications.

---

## Diagnostic Decision Matrix

| Symptom | Diagnostic Area |
|---|---|
| `adb` command not found | Host tool installation |
| Empty `adb devices` output | USB transport or device discovery |
| Device appears as `unauthorized` | Android authorization prompt |
| Device appears as `offline` | ADB transport or daemon state |
| No new entry in `lsusb` | Cable, port role, or hardware connection |
| USB device detected but ADB absent | Interface configuration or firmware restrictions |
| Permission denied | Linux permissions or udev rules |
| APK installation rejected | Android package restrictions |
| Launcher cannot be selected | Device policy or system configuration |
| Home Assistant dashboard fails to load | Network or browser compatibility |
| Calendar events do not synchronize | Calendar integration or authentication |

The matrix provides a way to identify the failing layer without changing unrelated settings.

---

## Security Considerations

The project involved modifying a proprietary Android device and connecting it to the home network.

Important considerations included:

### Debugging Access

ADB should be enabled only when required.

### Application Trust

APK files should come from reputable sources and be checked for compatibility.

### Network Isolation

The display should have access only to services required for its intended functionality.

### Credential Protection

Calendar and Home Assistant credentials should not be embedded in publicly accessible configuration files.

### Firmware Integrity

Unverified firmware images should not be installed.

### Backups

Device configuration and relevant application data should be preserved before significant changes.

### Cloud Dependencies

The original Skylight application and external calendar providers may continue to depend on cloud services.

### Administrative Recovery

A recovery procedure should be established before replacing the default launcher or modifying system software.

The project emphasized that gaining additional control over the device also increases responsibility for its security and maintenance.

---

## Technical Challenges

### Proprietary Android Interface

The original Skylight software restricted access to conventional Android functionality.

### Model-Specific Documentation

The referenced ADB walkthrough targeted the Calendar Max rather than the 15-inch model.

### Bazzite Development Environment

Android development tools needed to operate within Bazzite's atomic Linux architecture.

### USB Debugging Without Device Detection

USB debugging was enabled, but ADB continued to report an empty device list.

### Unknown USB Port Behavior

The available project record did not establish whether the connected port exposed an ADB-compatible USB interface.

### Application Compatibility

The device's Android version and system restrictions could limit which applications would run.

### Calendar Synchronization

The desired interface needed to combine Google Calendar and Apple Calendar data.

### Home Assistant Integration

The device needed a usable frontend for the existing Home Assistant instance.

### Recovery and Reversibility

Modifications needed to preserve the ability to return to the original Skylight environment.

These challenges demonstrated the difference between a device running Android and a device functioning as an unrestricted Android tablet.

---

## Evaluation Outcome

The project established that the Skylight Calendar provided access to Android Developer Options and allowed USB debugging to be enabled.

The Bazzite workstation had access to the ADB command-line tool.

However, repeated connection attempts returned:

```text
List of devices attached
```

with no detected device.

An attempt to launch Android Settings through ADB failed because no device or emulator was available.

The project also identified community documentation demonstrating application installation on certain Skylight models, but compatibility with the target 15-inch device remained unverified.

No successful APK sideloading, root access, launcher replacement, or complete Home Assistant dashboard deployment was documented.

The confirmed outcome was therefore the initial Android access investigation and the identification of USB/ADB device discovery as the immediate technical obstacle.

---

## Lessons Learned

Proprietary smart displays may contain capable general-purpose hardware beneath a restricted application interface.

An Android operating system does not guarantee unrestricted access to applications or debugging tools.

Enabling USB debugging is not sufficient to establish an ADB connection.

An empty ADB device list should be investigated at the USB transport layer before troubleshooting Android authorization.

Linux USB enumeration and ADB discovery are separate diagnostic stages.

Bazzite's atomic architecture requires consideration when installing development tools.

Community hardware modifications must be matched to the exact device model and firmware.

ADB access and root access provide different levels of system control.

Application-level customization is generally less risky than replacing firmware.

A locally hosted dashboard can separate application functionality from the display hardware.

Calendar synchronization, Android customization, and Home Assistant integration are distinct technical components.

The project strengthened understanding of Android debugging, Linux USB diagnostics, ADB, Bazzite, device firmware compatibility, and self-hosted dashboard architecture.

---

## Future Improvements

Future work may include:

- Confirming the Skylight's exact hardware revision.
- Recording the Android build fingerprint.
- Testing additional USB data cables.
- Identifying which physical ports support debugging.
- Capturing USB enumeration logs.
- Verifying host ADB permissions.
- Investigating wireless debugging support.
- Establishing an authorized ADB connection.
- Creating a non-destructive device configuration backup.
- Testing a compatible Android launcher.
- Installing a kiosk browser.
- Displaying the existing Home Assistant dashboard.
- Integrating Google Calendar events.
- Integrating Apple Calendar events.
- Evaluating a self-hosted CalDAV server.
- Developing a dedicated household dashboard.
- Testing touchscreen behavior and display sleep management.
- Implementing automatic dashboard startup.
- Documenting a reliable recovery procedure.
- Evaluating root access only if application-level methods prove insufficient.

These improvements would move the project from hardware investigation toward a functional locally managed household information terminal.

---

## Summary

The Skylight Calendar Android Integration project investigated converting a proprietary smart calendar display into a flexible Android-based Home Assistant and calendar dashboard.

The target device was a Skylight Calendar 15-inch model running a customized Android environment.

The project used a Bazzite Linux workstation and Android Debug Bridge to investigate system access.

Developer Options and USB debugging were enabled, but the workstation did not detect the device through ADB.

The investigation therefore focused on USB connectivity, Android debugging, Linux permissions, and device-specific firmware restrictions.

The project also evaluated community ADB documentation, alternative Android launchers, application sideloading, and self-hosted calendar architectures.

Although the original project did not reach a verified custom-dashboard deployment, it established a structured approach for evaluating and troubleshooting proprietary Android hardware.

The work demonstrates practical experience with Android system administration, Linux debugging, USB device communication, embedded hardware evaluation, and self-hosted smart home integration.

---

## Technical References

- [Android Developers — Android Debug Bridge](https://developer.android.com/tools/adb)
- [Android Developers — Run Apps on a Hardware Device](https://developer.android.com/studio/run/device)
- [Android Developers — SDK Platform Tools](https://developer.android.com/tools/releases/platform-tools)
- [SkylightMaxCalendarADB — Community Walkthrough](https://github.com/dinewby88/SkylightMaxCalendarADB)
- [Photo Frame Dashboard — Community Hardware Research](https://github.com/wesbos/photo-frame-dashboard)
- [Home Assistant Community — Skylight ADB Discussion](https://community.home-assistant.io/t/adb-skylight-calendar/791807)
- [Home Assistant — Official Documentation](https://www.home-assistant.io/docs/)
- [Home Assistant — Calendar Integration](https://www.home-assistant.io/integrations/calendar/)
- [Bazzite — Official Website](https://bazzite.gg/)
- [Bazzite — Official Documentation](https://docs.bazzite.gg/)
- [Android Studio — Official Documentation](https://developer.android.com/studio)
- [NetworkManager — Official Documentation](https://networkmanager.dev/docs/)
- [Unraid — Official Documentation](https://docs.unraid.net/)
