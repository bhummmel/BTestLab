
# Privacy-Focused Smart Home Terminals

## Overview

The Privacy-Focused Smart Home Terminals project focused on evaluating and adapting consumer electronics for use as dedicated, locally managed Home Assistant control interfaces.

The primary objective was to create convenient smart home terminals without relying on manufacturer cloud services, subscription-based dashboards or unnecessary third-party data collection.

The project explored repurposing Android-based devices, including Amazon Fire tablets and a Skylight smart calendar, as dedicated interfaces for the existing Home Assistant infrastructure.

A major consideration was the balance between convenience, privacy, security and hardware longevity.

Instead of treating consumer tablets as disposable devices tied to their original software ecosystems, the project investigated how their displays, touchscreens and network capabilities could be reused within a self-hosted environment.

The work included Android debugging, application management, kiosk-style interfaces, network isolation and local dashboard integration.

The project combined hands-on Android device experimentation with the evaluation of additional terminal configurations.

---

## Objectives

The primary objective was to establish a practical architecture for dedicated smart home terminals that could operate independently of unnecessary cloud services.

Specific goals included:

- Repurposing existing Android-based consumer devices.
- Providing dedicated Home Assistant dashboard access.
- Reducing dependence on manufacturer applications and cloud accounts.
- Investigating Android Debug Bridge (ADB) access.
- Evaluating kiosk-mode application configurations.
- Limiting unnecessary background services.
- Restricting network access to trusted infrastructure.
- Supporting touch-based smart home control.
- Evaluating local calendar and scheduling interfaces.
- Maintaining compatibility with existing Home Assistant automations.
- Developing a repeatable device configuration process.
- Extending the useful life of existing hardware.

An additional objective was to determine how much control could be recovered from consumer devices designed around proprietary software ecosystems.

---

## Infrastructure Environment

The project was designed around an existing self-hosted Home Assistant installation running on Unraid.

The environment includes a variety of smart home devices and local network services.

| Component | Role |
|---|---|
| Unraid | Primary server and container hosting platform |
| Home Assistant | Centralized smart home management |
| Docker | Application deployment and isolation |
| OPNsense | Firewall and network segmentation |
| UniFi networking | Wireless and wired network infrastructure |
| Amazon Fire tablets | Repurposed dashboard terminals |
| Skylight smart calendar | Android-based terminal evaluation |
| Linux desktop | ADB and device configuration workstation |
| Android Debug Bridge | Device administration and diagnostics |
| MQTT | Local device communication |
| Zigbee and Z-Wave | Smart home device connectivity |
| Home Assistant dashboards | User-facing control interface |

The existing environment was well suited to dedicated control terminals because Home Assistant already provided a centralized interface for devices from multiple manufacturers.

The terminals would primarily function as clients rather than independently managing smart home devices.

---

## System Architecture

The proposed architecture separates smart home control logic from the physical display devices.

Home Assistant remains responsible for device state, automation execution and integrations.

The terminals provide user interfaces through a local network connection.

```text
                 Smart Home Devices
                         |
           +-------------+-------------+
           |             |             |
           v             v             v
        Zigbee         Z-Wave         MQTT
           |             |             |
           +-------------+-------------+
                         |
                         v
                 +---------------+
                 | Home Assistant|
                 | Unraid Server |
                 +---------------+
                         |
                    Local Network
                         |
                 +-------+-------+
                 |               |
                 v               v
          +-------------+  +-------------+
          | Fire Tablet |  | Android     |
          | Dashboard   |  | Calendar    |
          +-------------+  +-------------+
                 |               |
                 v               v
             Touch UI        Touch UI
```

This architecture reduces the amount of processing and configuration required on each terminal.

A dashboard can be modified centrally without requiring extensive changes to every device.

---

## Privacy-First Design Philosophy

Privacy was a central requirement of the project.

Many consumer smart home displays are designed to operate through manufacturer accounts and remote services.

These dependencies can introduce additional data collection, external connectivity requirements and limitations on local control.

The project evaluated an alternative model in which the terminal acts primarily as a display for locally hosted services.

Important design principles included:

- Local-first communication.
- Minimal cloud dependencies.
- Restricted application permissions.
- Limited background network activity.
- Centralized control through Home Assistant.
- Separation of user interface and automation logic.
- Reuse of existing hardware.
- Administrative access through trusted systems.

The objective was not necessarily to eliminate every external connection.

Instead, it was to identify which connections were necessary and prevent unnecessary services from becoming dependencies for basic smart home control.

---

## Amazon Fire Tablet Evaluation

Amazon Fire tablets were investigated as low-cost smart home control terminals.

These devices provide touchscreens, Wi-Fi connectivity and sufficient computing capability for browser-based dashboards.

However, Fire OS includes manufacturer-specific services and application restrictions.

The project examined how these devices could be adapted for dedicated Home Assistant use.

Potential configuration tasks included:

- Installing a compatible browser or dashboard application.
- Configuring automatic dashboard launch.
- Preventing unwanted screen interruptions.
- Adjusting display timeout behavior.
- Restricting unnecessary applications.
- Managing system updates.
- Reducing unwanted background services.
- Establishing reliable Wi-Fi connectivity.

The evaluation emphasized retaining enough Android functionality for stable operation while minimizing software unrelated to the dashboard.

---

## Android Debug Bridge

Android Debug Bridge was a central tool for investigating device administration.

ADB provides a command-line interface for communicating with compatible Android devices.

It can be used to inspect device information, manage applications and perform supported administrative operations.

On a Linux workstation, basic device discovery can be performed using:

```bash
adb devices
```

Additional device information can be inspected through:

```bash
adb shell getprop
```

Installed Android packages can be listed using:

```bash
adb shell pm list packages
```

These commands provide useful information when evaluating unfamiliar Android hardware.

ADB access depends on the device exposing an appropriate debugging interface and authorizing the connected computer.

Enabling USB debugging alone does not guarantee that the device will appear in the ADB device list.

The project reinforced the importance of verifying physical connectivity, USB operating mode and debugging authorization before attempting further configuration.

---

## Fire OS Application Management

The project considered reducing unwanted Fire OS functionality through application management.

Android provides mechanisms for disabling certain packages for a particular user profile.

For example, the general command structure is:

```bash
adb shell pm disable-user --user 0 PACKAGE_NAME
```

This operation is different from deleting an application from the underlying system partition.

Disabling a package may prevent it from running for the selected Android user.

However, not every system component can be safely disabled.

Removing or disabling dependencies used by the launcher, system settings or other core services may cause instability.

The evaluation emphasized identifying package functions before changing their state.

A reversible configuration process was preferred over aggressive modification of system software.

---

## Kiosk-Mode Dashboard Design

A dedicated smart home terminal benefits from a simplified interface.

Kiosk mode is a configuration in which a device is primarily restricted to one application or interface.

For Home Assistant, this commonly involves displaying a dashboard through a browser or compatible application.

A proposed kiosk configuration would provide:

- Automatic dashboard launch.
- Full-screen display.
- Minimal navigation controls.
- Restricted access to unrelated applications.
- Predictable behavior after reboot.
- Automatic recovery after connection failures.

The design goal was to make the device behave like a purpose-built control panel rather than a general-purpose tablet.

However, kiosk mode must be implemented carefully.

A reliable administrative recovery method should remain available so that the device can be updated or repaired.

---

## Home Assistant Dashboard Integration

Home Assistant provides the central interface for smart home control.

The terminals would display dashboards tailored to their physical location and intended purpose.

Potential dashboard functions include:

- Lighting control.
- Thermostat adjustment.
- Room temperature and humidity.
- Motion and presence information.
- Media playback controls.
- Guest mode.
- Security and sensor status.
- Household routines.
- Device availability indicators.

The existing Home Assistant environment already includes automations for lighting, environmental sensors, media equipment and household appliances.

Dedicated terminals could provide a convenient interface for interacting with these systems.

The project emphasized that dashboards should expose useful controls without overwhelming users with unnecessary technical information.

---

## Room-Specific Dashboard Architecture

Different rooms have different control requirements.

A single universal dashboard may not provide the most efficient interface for every location.

The project considered room-specific terminal configurations.

For example, a living room terminal could prioritize lighting and media controls.

A hallway terminal could display household status, weather information and common automations.

A bedroom terminal could prioritize lighting, climate control and nighttime routines.

The architecture would allow each terminal to load a designated Home Assistant dashboard.

This approach provides centralized administration while preserving location-specific functionality.

It also reduces the need for users to navigate through unrelated devices.

---

## Guest Mode Integration

The existing Home Assistant environment includes a guest mode controlled through an input boolean.

Guest mode provides an opportunity to simplify the terminal interface for visitors.

A guest-facing dashboard could expose selected controls without providing access to administrative functions.

Potential features include:

- Basic lighting controls.
- Common media functions.
- Thermostat adjustments within permitted limits.
- Household information.
- Simple room navigation.

The evaluation considered the distinction between changing a dashboard's appearance and enforcing actual authorization.

Hiding controls does not necessarily prevent a user from accessing the underlying service.

Appropriate Home Assistant accounts, permissions and network controls remain important.

The project reinforced the principle that user interface restrictions should not be treated as complete security boundaries.

---

## Skylight Calendar Hardware Evaluation

The project also investigated repurposing a Skylight smart calendar.

The device was of interest because it provides a relatively large touchscreen display designed for persistent household use.

The evaluated hardware included a 15-inch Android-based calendar platform associated with the RK3562 hardware family.

The original software environment was designed around the manufacturer's calendar application and services.

The project investigated whether the device could be used for a more flexible local dashboard.

Potential applications included:

- Home Assistant dashboards.
- Household calendar displays.
- Locally hosted scheduling applications.
- Shared task lists.
- Smart home status panels.
- Family information dashboards.

The evaluation focused on recovering general-purpose Android functionality without unnecessarily depending on the original cloud platform.

---

## Skylight ADB Troubleshooting

The Skylight evaluation included enabling USB debugging and attempting to establish an ADB connection from a Linux workstation.

The device did not initially appear in the output of:

```bash
adb devices
```

This occurred despite USB debugging being enabled in Android settings.

The problem demonstrated that Android debugging involves several independent requirements.

The operating system must expose a functioning ADB interface.

The USB connection must support data transfer.

The host must recognize the USB device.

The Android device must authorize the debugging session.

Some embedded Android devices may expose debugging only through specific interfaces or require additional configuration.

The project therefore treated ADB connectivity as a hardware and operating system investigation rather than assuming that a visible debugging toggle guaranteed access.

---

## Linux Workstation Integration

The device configuration work was performed in the context of a Linux desktop environment.

The workstation used Bazzite, a Fedora-based operating system.

This introduced additional considerations for installing and accessing Android development tools.

ADB was used as the primary command-line interface for investigating connected Android devices.

The project included examining USB device detection and troubleshooting the absence of expected ADB connections.

A systematic diagnostic workflow would include:

```bash
lsusb
```

```bash
adb kill-server
adb start-server
adb devices -l
```

These commands help distinguish between USB enumeration problems and ADB authorization problems.

If a device appears in `lsusb` but not in ADB, additional investigation is required into its exposed USB interfaces and debugging configuration.

The project strengthened practical experience with Linux device administration and Android connectivity troubleshooting.

---

## Local Calendar Integration

The Skylight evaluation also considered replacing proprietary calendar functionality with locally controlled services.

The objective was to display household schedules without making the display dependent on a particular manufacturer's application.

Potential approaches included:

- A Home Assistant calendar dashboard.
- A locally hosted web calendar.
- A CalDAV-compatible calendar interface.
- A self-hosted household scheduling application.
- A custom dashboard combining calendar and smart home information.

Google Calendar and Apple Calendar integration were also considered.

However, synchronization with external calendar providers introduces additional service dependencies.

The project distinguished between locally hosting the user interface and maintaining a completely cloud-independent calendar system.

A local dashboard can still display information synchronized from external services.

---

## Network Segmentation

Dedicated smart home terminals should not require unrestricted access to the entire network.

The existing OPNsense and UniFi environment provides opportunities for network segmentation.

A proposed configuration would place terminals on an appropriate network segment with controlled access to Home Assistant.

The terminals would require connectivity to the dashboard service and any necessary supporting infrastructure.

They would not generally require administrative access to Unraid, firewall management or unrelated server applications.

The evaluation emphasized limiting traffic based on actual application requirements.

This approach reduces the consequences of a compromised or outdated Android device.

---

## DNS and Local Service Discovery

A locally hosted dashboard requires a reliable method of locating the Home Assistant server.

The existing environment includes local DNS infrastructure through Unbound.

Using a consistent internal hostname would simplify terminal configuration.

Instead of configuring each device with a changing IP address, the terminal could access a stable local service name.

However, the DNS configuration must remain functional during internet outages if local control is a requirement.

The project reinforced the importance of distinguishing internal name resolution from external cloud-based DNS dependencies.

Reliable local service discovery is essential for a genuinely local-first smart home interface.

---

## Authentication and Access Control

Dedicated terminals introduce unique authentication considerations.

A wall-mounted display may be physically accessible to household members or visitors.

The terminal should therefore use an account appropriate for its intended functions.

Administrative Home Assistant credentials should not be used unnecessarily.

Important considerations include:

- Dedicated dashboard accounts.
- Restricted access to sensitive controls.
- Session management.
- Secure credential storage.
- Device lock and recovery procedures.
- Physical access to the terminal.
- Separation of guest and administrative interfaces.

The evaluation emphasized that a device's physical location affects its security requirements.

A publicly accessible household terminal should not expose the same administrative functionality as a trusted personal workstation.

---

## Display Power Management

Always-on displays introduce power consumption and hardware longevity considerations.

A terminal may need to remain visible throughout the day while reducing brightness or turning off its screen when not in use.

Potential approaches include:

- Scheduled brightness adjustments.
- Screen timeout configuration.
- Motion-triggered wake behavior.
- Presence-aware display control.
- Nighttime dimming.
- Automatic screen recovery after reboot.

The existing Home Assistant environment includes motion and presence sensors that could potentially support these functions.

However, implementation depends on the Android device and available control interfaces.

The evaluation treated display power management as an important usability and reliability requirement.

---

## Device Recovery and Maintenance

Repurposed consumer devices may have limited long-term software support.

This creates operational risks involving outdated Android versions, application compatibility and security updates.

The project considered the importance of maintaining a recoverable configuration.

A practical maintenance process should include:

- Recording device model and firmware information.
- Documenting installed applications.
- Maintaining ADB access where supported.
- Preserving original configuration details.
- Testing changes incrementally.
- Retaining a recovery method.
- Monitoring device connectivity.
- Planning for eventual hardware replacement.

The project emphasized avoiding irreversible modifications unless the recovery implications are understood.

---

## Troubleshooting Methodology

The project used a layered approach to investigating terminal configuration problems.

### Stage 1: Verify Hardware Operation

Confirm that the device powers on and that the touchscreen and display function correctly.

### Stage 2: Identify the Operating System

Determine the Android version, device model and available management interfaces.

### Stage 3: Establish Administrative Access

Test ADB or other supported device management methods.

### Stage 4: Validate Network Connectivity

Confirm Wi-Fi connectivity, local addressing and DNS resolution.

### Stage 5: Test Home Assistant Access

Verify that the device can load the intended dashboard.

### Stage 6: Configure Kiosk Behavior

Evaluate automatic launch, screen behavior and navigation restrictions.

### Stage 7: Restrict Unnecessary Services

Disable only identified services that are not required for stable operation.

### Stage 8: Validate Recovery

Restart the device and confirm that the dashboard returns to the expected state.

This approach reduces the likelihood of making changes that prevent the terminal from operating reliably.

---

## Technical Challenges Encountered

### Proprietary Android Distributions

Consumer devices may use customized Android builds with restricted functionality.

### ADB Connectivity

Enabling USB debugging does not guarantee that the device exposes an accessible ADB interface.

### Application Compatibility

Older Android versions may not support current browser or dashboard applications.

### Cloud Dependencies

Manufacturer software may depend on external services for basic functionality.

### Kiosk Reliability

A dedicated terminal must recover automatically after reboots or application failures.

### Display Management

Always-on screens require appropriate brightness and power management.

### Security Updates

Repurposed hardware may no longer receive regular manufacturer security patches.

### Administrative Recovery

Aggressive system modification can make devices difficult to maintain or restore.

These challenges informed the project's preference for incremental, reversible configuration changes.

---

## Security Considerations

Privacy-focused terminals still require conventional endpoint security.

Removing manufacturer applications does not automatically make an Android device secure.

The evaluation identified several important controls:

- Restrict access to internal services.
- Use dedicated non-administrative accounts.
- Keep supported software updated.
- Disable unnecessary remote access features.
- Protect ADB access.
- Avoid exposing debugging interfaces to untrusted networks.
- Limit installed applications.
- Monitor unexpected network activity.
- Maintain secure local authentication.

The project also emphasized that outdated firmware remains a risk even when the device communicates only with locally hosted services.

Network isolation and least-privilege accounts help reduce this exposure.

---

## Reliability and Recovery

A smart home terminal should remain usable during ordinary infrastructure interruptions.

The proposed architecture keeps automation logic on the Home Assistant server rather than on the terminal.

This means that a tablet failure does not necessarily prevent existing automations from continuing to operate.

However, the terminal still depends on local network connectivity and Home Assistant availability.

A reliable deployment would require:

- Stable Wi-Fi coverage.
- Predictable dashboard startup.
- Automatic application recovery.
- Appropriate power delivery.
- Local DNS availability.
- Documented device configuration.
- A practical replacement procedure.

The evaluation reinforced the value of keeping user interfaces replaceable and independent of core automation logic.

---

## Evaluation Outcome

The project identified repurposed Android devices as a practical category of hardware for dedicated smart home terminals.

Amazon Fire tablets provide an accessible platform for dashboard experimentation.

The Skylight calendar offers a larger touchscreen form factor but introduces additional challenges associated with proprietary firmware and device management.

The project included hands-on investigation of Android debugging and device connectivity, including an unsuccessful initial ADB connection to the Skylight hardware.

The broader architecture supports local Home Assistant dashboards, restricted network access and centralized interface management.

However, the available project information does not establish that every proposed privacy modification, kiosk configuration or calendar replacement was successfully deployed.

The work should therefore be understood as a combination of device experimentation, troubleshooting and infrastructure design.

---

## Lessons Learned

Consumer Android devices can often be repurposed for applications beyond their original intended purpose.

Local-first smart home interfaces can reduce dependence on proprietary cloud services.

Android debugging requires cooperation between device firmware, USB interfaces and the host operating system.

Kiosk mode improves usability but does not replace proper authentication and network security.

A dedicated terminal should use restricted credentials rather than administrative accounts.

Centralizing automation logic in Home Assistant allows display devices to remain replaceable.

Proprietary firmware can limit hardware reuse even when the underlying device runs Android.

The project strengthened understanding of Android administration, Linux device troubleshooting, network segmentation and privacy-focused infrastructure design.

---

## Future Improvements

Future development may include:

- Establishing reliable ADB connectivity with the Skylight calendar.
- Evaluating alternative Android launchers.
- Deploying a dedicated Home Assistant kiosk interface.
- Implementing room-specific dashboards.
- Configuring automatic dashboard startup.
- Testing screen wake and brightness automation.
- Investigating local calendar applications.
- Evaluating CalDAV synchronization.
- Monitoring terminal network traffic.
- Restricting unnecessary manufacturer services.
- Documenting repeatable tablet configuration procedures.
- Evaluating long-term device security and maintenance.
- Testing alternative low-power touchscreen hardware.
- Developing a standardized smart home terminal deployment process.

These improvements would support progression from individual device experimentation to a consistent fleet of locally managed household terminals.

---

## Summary

The Privacy-Focused Smart Home Terminals project investigated how consumer Android devices could be repurposed as dedicated Home Assistant control interfaces.

The work focused on Amazon Fire tablets, Skylight calendar hardware, Android Debug Bridge, kiosk-mode applications and locally hosted dashboards.

The evaluation examined privacy, cloud dependencies, authentication, network segmentation, device recovery and long-term maintainability.

Hands-on troubleshooting provided experience with Linux-based Android administration and device connectivity.

A central architectural principle was keeping smart home automation and device management on the local Home Assistant server while using inexpensive touchscreen devices as replaceable interfaces.

Although some hardware modifications and integrations remain under evaluation, the project established a practical foundation for privacy-conscious smart home terminals.

The work demonstrates skills in Linux systems administration, Android device management, networking, endpoint security, Home Assistant integration, troubleshooting and infrastructure design.
