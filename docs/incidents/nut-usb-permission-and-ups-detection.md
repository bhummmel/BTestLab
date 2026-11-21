
# NUT USB Permission and UPS Detection Troubleshooting

## Overview

The NUT USB Permission and UPS Detection Troubleshooting project focused on investigating communication problems between a Linux-based NAS server and a USB-connected uninterruptible power supply (UPS).

The objective was to establish reliable UPS detection, resolve potential USB device permission conflicts and understand how Network UPS Tools (NUT) communicates with power protection hardware.

The project formed part of the homelab's broader effort to improve infrastructure reliability, particularly during electrical disturbances and utility power outages.

The environment included an APC Smart-UPS SMC1500-2UC, which provides battery backup and power protection for connected equipment.

The UPS supported the goal of allowing the storage server to monitor power conditions and initiate an orderly shutdown when battery capacity or runtime became critically low.

This is particularly important for a NAS because an abrupt power interruption can disrupt active file operations, interrupt applications and complicate storage recovery.

The investigation involved Linux USB enumeration, device permissions, NUT driver selection, USB HID communication, service configuration and UPS monitoring.

A central troubleshooting consideration was that detecting a UPS at the USB subsystem level does not automatically mean that NUT can communicate with it.

The USB device must be recognized, a compatible driver must be selected, the NUT process must have the necessary permissions and the monitoring services must be configured correctly.

The broader homelab subsequently used Unraid as its primary NAS platform, with UPS monitoring forming part of the infrastructure reliability strategy.

The available project history confirms the APC SMC1500-2UC hardware and the homelab's interest in UPS monitoring. However, it does not preserve the exact original USB permission errors, device identifiers or final NUT configuration.

This document therefore presents the troubleshooting methodology and technical architecture without claiming an unverified repair or deployment outcome.

---

## Objectives

The primary objective was to investigate USB detection and communication problems affecting NUT integration with an APC UPS.

Specific goals included:

- Understanding Network UPS Tools architecture.
- Identifying USB-connected UPS hardware.
- Verifying Linux USB device enumeration.
- Selecting an appropriate NUT driver.
- Investigating USB HID compatibility.
- Diagnosing device permission errors.
- Understanding Linux udev rules.
- Verifying NUT driver initialization.
- Troubleshooting UPS communication failures.
- Understanding NUT service dependencies.
- Configuring local UPS monitoring.
- Evaluating network-based UPS monitoring.
- Investigating automatic shutdown behavior.
- Improving NAS power protection.
- Developing repeatable UPS diagnostics.

An additional objective was to ensure that UPS communication problems could be diagnosed without compromising the availability of the storage server.

---

## Infrastructure Environment

The project was associated with the homelab's Linux-based storage and application infrastructure.

The primary NAS platform eventually transitioned from TrueNAS SCALE to Unraid.

The UPS was intended to protect important infrastructure components and provide information about electrical conditions.

| Component | Role |
|---|---|
| APC SMC1500-2UC | Battery backup and power protection |
| Unraid | Primary NAS and application host |
| Linux kernel | USB device and driver management |
| Network UPS Tools | UPS communication and monitoring |
| `usbhid-ups` | NUT driver for compatible USB HID UPS devices |
| udev | Linux device event and permission management |
| `upsd` | NUT data server |
| `upsmon` | UPS monitoring and shutdown coordination |
| USB | Direct UPS communication |
| Ethernet | Alternative management connectivity |
| Docker | Application hosting infrastructure |
| Home Assistant | Potential UPS status visualization and automation |

The APC SMC1500-2UC supports management connectivity, but the available interfaces and supported protocols must be verified against the installed hardware and firmware.

The project emphasized selecting the communication method based on the actual UPS capabilities rather than assuming that every APC UPS uses the same interface.

---

## System Architecture

A typical NUT deployment consists of a UPS driver, a data server and one or more monitoring clients.

```text
                 Utility Power
                       |
                       v
              +----------------+
              | APC Smart-UPS  |
              | SMC1500-2UC    |
              +----------------+
                       |
                  USB Connection
                       |
                       v
              +----------------+
              | Linux USB      |
              | Subsystem      |
              +----------------+
                       |
                       v
              +----------------+
              | NUT Driver     |
              | usbhid-ups     |
              +----------------+
                       |
                       v
              +----------------+
              | NUT Server     |
              | upsd           |
              +----------------+
                       |
              +--------+--------+
              |                 |
              v                 v
          upsmon          Monitoring Client
              |                 |
              v                 v
        NAS Shutdown      Home Assistant /
        Coordination      Network Monitoring
```

This architecture separates hardware communication from power monitoring and shutdown decisions.

The project emphasized that each component can fail independently.

For example, a functioning USB driver does not guarantee that the NUT server is reachable or that the shutdown monitor is configured correctly.

---

## Understanding Network UPS Tools

Network UPS Tools is an open-source software suite designed to communicate with UPS hardware and expose power status information to local or remote clients.

NUT supports a variety of manufacturers and communication protocols.

Its architecture generally includes three primary components.

### UPS Driver

The driver communicates directly with the physical UPS.

### NUT Server

The `upsd` service exposes UPS status information to authorized clients.

### UPS Monitor

The `upsmon` service monitors UPS state and coordinates shutdown behavior.

Separating these responsibilities allows one system to communicate directly with a UPS while other systems receive status information over the network.

This is particularly useful in a homelab where several servers may depend on the same UPS.

---

## APC UPS Communication

APC UPS models support different communication interfaces depending on their product family and generation.

Possible interfaces include:

- USB.
- Serial communication.
- Ethernet management.
- SNMP through compatible management hardware.
- Manufacturer-specific protocols.

The project focused primarily on USB detection and permissions.

For a compatible USB HID UPS, NUT commonly uses the `usbhid-ups` driver.

However, the correct driver must be determined from the device's actual USB identity and supported protocol.

The project emphasized checking hardware compatibility before assuming that a driver is appropriate.

---

## Linux USB Enumeration

The first diagnostic step is determining whether the operating system can detect the connected UPS.

Linux provides the `lsusb` utility for listing USB devices.

```bash
lsusb
```

A recognized APC device may appear with the manufacturer's USB vendor identifier.

APC commonly uses vendor ID:

```text
051d
```

The product identifier varies by device.

A targeted command is:

```bash
lsusb -d 051d:
```

The project emphasized recording the actual vendor and product identifiers.

These values can help distinguish an unsupported device from a permissions or driver configuration problem.

---

## USB Detection Versus Driver Communication

A UPS appearing in `lsusb` confirms that the host has enumerated a USB device.

It does not establish that NUT can successfully communicate with it.

The device may still be inaccessible because of:

- Incorrect permissions.
- Unsupported USB interface.
- Wrong NUT driver.
- Another process holding the device.
- USB communication errors.
- Firmware compatibility problems.
- Incomplete service configuration.

The project emphasized distinguishing hardware enumeration from application-level communication.

This prevents unnecessary driver changes when the underlying issue is access control.

---

## USB Device Files

Linux exposes USB devices through device nodes.

For example:

```text
/dev/bus/usb/001/004
```

The numbers identify the USB bus and device address.

These values are assigned dynamically and may change when the device is disconnected or reconnected.

The current device path can be determined using:

```bash
lsusb
```

Permissions can then be inspected using:

```bash
ls -l /dev/bus/usb/001/004
```

The path is illustrative and must be replaced with the actual device location.

The project emphasized that hardcoding dynamic USB bus and device numbers is generally undesirable.

---

## USB Device Permissions

Linux restricts access to USB device nodes through filesystem permissions and device access policies.

A process attempting to communicate with the UPS must have sufficient permissions to open the relevant USB device.

NUT drivers may run under a dedicated service account rather than as root.

If that account lacks access, driver initialization may fail.

Potential symptoms include:

```text
Permission denied
```

```text
Can't claim USB device
```

```text
No matching HID UPS found
```

These are representative diagnostic messages, not preserved output from the original deployment.

The project emphasized determining whether the device is absent, inaccessible or incompatible before modifying permissions.

---

## Understanding udev

udev is the Linux userspace device manager.

It processes device events and applies rules that can influence device naming, ownership and permissions.

When a USB UPS is connected, udev may apply rules based on attributes such as:

- Vendor ID.
- Product ID.
- Device subsystem.
- Device class.
- Device path.

These rules can allow a dedicated NUT service account to access the UPS without granting broad permissions to all USB devices.

The project emphasized using targeted device permissions rather than weakening access controls globally.

---

## Inspecting udev Device Attributes

The attributes associated with a USB device can be inspected using `udevadm`.

For example:

```bash
udevadm info \
  --attribute-walk \
  --name=/dev/bus/usb/001/004
```

The device path must be replaced with the actual UPS node.

This command can expose information useful for constructing device matching rules.

The project emphasized verifying the correct USB attributes before creating a rule.

A rule that matches the wrong device may have no effect or may unintentionally change permissions on unrelated hardware.

---

## Example udev Permission Rule

On a conventional Linux installation, a device-specific udev rule may be used to grant access to a dedicated UPS service group.

For example:

```text
SUBSYSTEM=="usb", ATTR{idVendor}=="051d", ATTR{idProduct}=="XXXX", GROUP="nut", MODE="0660"
```

The product identifier must be replaced with the actual value.

The `nut` group must exist, and the driver process must have the appropriate group membership.

This is an illustrative configuration rather than a confirmed rule from the original server.

On appliance-oriented operating systems such as Unraid or TrueNAS SCALE, manually created udev rules may not persist across updates or reboots.

The project emphasized using platform-supported UPS configuration mechanisms before introducing custom device rules.

---

## Avoiding World-Writable USB Devices

A common troubleshooting shortcut is granting unrestricted access to a USB device.

For example:

```bash
chmod 666 /dev/bus/usb/001/004
```

Although such a change may temporarily alter access behavior, it is not an appropriate permanent solution.

The permissions may be reset when the device reconnects.

It also grants unnecessary access to other local processes.

The project emphasized implementing persistent, narrowly scoped permissions through supported configuration methods.

A reliable solution should grant access only to the service that requires it.

---

## NUT Driver Selection

NUT supports multiple UPS communication drivers.

The correct driver depends on the hardware.

For compatible APC USB HID devices, the expected driver is commonly:

```text
usbhid-ups
```

A typical NUT configuration may resemble:

```ini
[apc]
    driver = usbhid-ups
    port = auto
    desc = "APC Smart-UPS"
```

This is a reference configuration.

The actual driver and device parameters must be verified against the installed UPS.

The project emphasized using the correct driver before investigating more complex communication problems.

---

## NUT Configuration Files

A conventional NUT installation commonly uses several configuration files.

| File | Purpose |
|---|---|
| `nut.conf` | NUT operating mode |
| `ups.conf` | UPS device and driver configuration |
| `upsd.conf` | NUT server listening configuration |
| `upsd.users` | NUT client authentication and permissions |
| `upsmon.conf` | UPS monitoring and shutdown configuration |

The exact file locations and management procedures depend on the Linux distribution.

Unraid may manage UPS settings through its own interface or installed plugins.

The project emphasized avoiding unsupported direct modifications to appliance-managed configuration files.

---

## NUT Operating Modes

NUT supports different deployment models.

### Standalone

The UPS and monitoring service operate on the same host.

### Network Server

One system communicates with the UPS and exposes its status to network clients.

### Network Client

A system receives UPS information from another NUT server.

The project considered the network server model particularly relevant to a homelab with multiple devices.

However, the available history does not establish that a multi-host NUT shutdown deployment was completed.

---

## Testing UPS Communication

Once the driver and server are configured, NUT status can be queried using:

```bash
upsc apc@localhost
```

The UPS name must match the configured device identifier.

A functioning connection may return values such as:

```text
battery.charge
battery.runtime
input.voltage
ups.load
ups.status
```

The exact variables depend on the UPS model and driver.

Not every device exposes every measurement.

The project emphasized using actual returned values to verify communication.

A configured UPS entry alone does not establish that the device is responding.

---

## Understanding UPS Status Values

NUT commonly represents UPS state through status flags.

Examples include:

| Status | Meaning |
|---|---|
| `OL` | Operating on line power |
| `OB` | Operating on battery |
| `LB` | Low battery |
| `CHRG` | Battery charging |
| `DISCHRG` | Battery discharging |
| `FSD` | Forced shutdown condition |

Status flags may appear in combination.

For example, a UPS can be operating on battery while its charge is decreasing.

The project emphasized interpreting the reported status in context rather than assuming a single flag describes the entire operating condition.

---

## NUT Driver Debugging

NUT drivers support diagnostic output that can help identify communication problems.

On a conventional installation, a driver may be launched in a foreground debug mode.

For example:

```bash
usbhid-ups -a apc -DDD
```

The configured UPS identifier must match the relevant NUT configuration.

Before running a driver manually, the existing managed driver instance should be stopped using the platform's supported service controls.

Two processes attempting to communicate with the same USB UPS can interfere with one another.

The project emphasized preserving the managed service configuration during diagnostics.

---

## Driver Startup Failures

A NUT driver may fail during initialization for several reasons.

Potential causes include:

- USB device not detected.
- Incorrect driver selection.
- Missing permissions.
- Unsupported USB interface.
- Conflicting monitoring software.
- Invalid configuration.
- USB communication timeout.
- Device firmware behavior.

The project emphasized reviewing driver logs before changing the NUT server or shutdown configuration.

A driver that cannot communicate with the UPS will not provide useful status data to downstream monitoring services.

---

## USB Device Contention

Some UPS communication interfaces do not tolerate multiple independent processes accessing the device simultaneously.

Potential conflicting software includes:

- Another NUT driver instance.
- APC-specific monitoring software.
- A previously started diagnostic process.
- An operating system UPS management service.
- A container with direct USB access.

The project emphasized verifying that only the intended driver process controls the UPS.

Restarting multiple monitoring applications without identifying the active device owner can make troubleshooting more difficult.

---

## USB Power Management

Linux USB power management can affect some devices.

Certain USB peripherals may behave unexpectedly after suspend, resume or power state transitions.

For a server intended to run continuously, reliable USB communication is particularly important.

The investigation considered whether device disconnects or communication timeouts correlated with power management events.

However, the available history does not establish USB autosuspend as the cause of the original UPS issue.

The project emphasized examining kernel logs before attributing communication failures to power management.

---

## Kernel Log Diagnostics

Linux kernel logs can provide evidence of USB communication problems.

Useful commands include:

```bash
dmesg | grep -Ei 'usb|hid|ups'
```

And, on systems using systemd:

```bash
journalctl -k | grep -Ei 'usb|hid|ups'
```

Potential messages may identify:

- USB connection events.
- Device descriptor failures.
- USB resets.
- HID interface registration.
- Communication timeouts.
- Device disconnections.

The project emphasized correlating these messages with NUT driver failures.

A USB reset immediately preceding a driver disconnect may indicate a different problem from a persistent permission error.

---

## Physical USB Troubleshooting

Software configuration cannot compensate for an unreliable physical connection.

The investigation considered:

- USB cable condition.
- Connector seating.
- USB port compatibility.
- Direct versus hub connections.
- Device power state.
- USB controller behavior.
- Cable length.

A useful diagnostic technique is testing the UPS through a direct USB connection using a known-good cable.

This helps distinguish physical communication failures from operating system configuration problems.

---

## Serial Versus USB Communication

Some UPS models support serial-style management connections.

However, connector appearance alone does not establish protocol compatibility.

A USB connector may expose a USB HID interface rather than a conventional serial port.

Similarly, an RJ-style management connector may use a manufacturer-specific pinout.

The project emphasized verifying the UPS documentation before selecting a driver or connecting alternative cables.

Using an incorrect cable or protocol can prevent communication even when the hardware is otherwise functional.

---

## Ethernet-Based UPS Monitoring

The APC SMC1500-2UC also made network management an architectural consideration.

Network-based monitoring can eliminate dependence on direct USB access from a particular host when the UPS exposes a supported network management protocol.

Potential benefits include:

- Centralized monitoring.
- Reduced USB dependency.
- Multiple monitoring clients.
- Easier integration with network management systems.
- Remote UPS status access.

However, the exact network management capabilities depend on the UPS hardware and firmware.

The project emphasized verifying whether the device exposes a supported protocol such as SNMP or another management interface.

An Ethernet connector alone does not guarantee that NUT can communicate with the UPS over the network.

---

## NUT Network Server Architecture

A NUT server can expose UPS information to multiple network clients.

```text
                APC UPS
                   |
                   v
             NUT Driver Host
                   |
                   v
                 upsd
                   |
          +--------+--------+
          |                 |
          v                 v
      NAS Monitor      Secondary Server
          |                 |
          v                 v
      Shutdown          Shutdown
      Decision          Decision
```

This architecture can coordinate shutdown behavior across multiple systems.

The project considered this model for future infrastructure improvements.

However, network monitoring clients must be configured carefully to avoid unauthorized access or unintended shutdown commands.

---

## NUT Authentication

NUT supports authenticated client access.

The `upsd.users` configuration defines users and associated permissions.

Monitoring clients can authenticate to the server using configured credentials.

The project emphasized:

- Using dedicated monitoring accounts.
- Restricting administrative privileges.
- Protecting credentials.
- Limiting network access to the NUT server.
- Avoiding unnecessary exposure to the internet.

NUT commonly uses TCP port 3493 for its network protocol.

The service should generally be restricted to trusted networks.

---

## UPS Shutdown Coordination

One of the most important functions of NUT is coordinating orderly shutdown during power failures.

A typical shutdown process may involve:

1. UPS detects loss of utility power.
2. NUT reports battery operation.
3. Monitoring software tracks battery state.
4. Shutdown criteria are reached.
5. The host begins an orderly shutdown.
6. Applications and storage services stop.
7. The operating system powers down.

The exact behavior depends on the configured monitoring and shutdown policies.

The project emphasized that UPS detection alone does not establish that shutdown coordination is functioning.

The shutdown process must be tested separately.

---

## Battery Runtime Considerations

UPS runtime depends on several factors.

These include:

- Connected electrical load.
- Battery condition.
- Battery age.
- UPS efficiency.
- Power factor.
- Environmental conditions.

The SMC1500-2UC's rated capacity does not imply a fixed runtime for every load.

A server drawing substantial power may have significantly less runtime than a lightly loaded network appliance.

The project emphasized using actual UPS telemetry and controlled testing rather than relying exclusively on nominal capacity.

---

## Shutdown Thresholds

NUT shutdown behavior can be based on UPS status and configured monitoring logic.

A UPS may report a low-battery condition when its internal thresholds are reached.

Some environments also use additional logic based on remaining runtime or battery charge.

The project emphasized avoiding shutdown thresholds that leave insufficient time for the NAS to stop its services.

Storage arrays, virtual machines and containers may require additional shutdown time.

A reliable shutdown policy should account for the actual system workload.

---

## Protecting Storage Integrity

The UPS was particularly important because the homelab hosts persistent storage.

Unexpected power loss can interrupt active writes and application transactions.

Although modern filesystems provide mechanisms for maintaining consistency, battery-backed shutdown remains valuable.

The project emphasized protecting:

- NAS filesystem operations.
- Application databases.
- Docker containers.
- Virtual machines.
- Media services.
- Network infrastructure.
- Configuration files.

The objective was not merely to keep the server running temporarily, but to provide enough time for a controlled shutdown.

---

## Home Assistant Integration

The broader homelab uses Home Assistant for infrastructure monitoring and automation.

NUT can expose UPS status information that Home Assistant may consume through a compatible integration.

Potential monitored values include:

- Battery charge.
- Remaining runtime.
- UPS load.
- Input voltage.
- Output voltage.
- Line power status.
- Battery operation.
- Low-battery conditions.

These measurements could support dashboards and notifications.

For example, Home Assistant could notify an administrator when the UPS switches to battery power.

The available history does not establish that every proposed UPS entity or automation was deployed.

The project considered these capabilities as potential monitoring improvements.

---

## Troubleshooting Methodology

The investigation can be organized into a layered diagnostic process.

### Stage 1: Verify Physical Connectivity

Confirm that the UPS is powered and connected through a suitable cable.

### Stage 2: Confirm USB Enumeration

Use `lsusb` to determine whether Linux detects the UPS.

### Stage 3: Record Device Identifiers

Document the USB vendor and product identifiers.

### Stage 4: Inspect Device Permissions

Determine whether the NUT driver process can access the USB device.

### Stage 5: Verify Driver Compatibility

Confirm that the selected NUT driver supports the UPS communication interface.

### Stage 6: Check for Conflicting Processes

Ensure that another monitoring application is not controlling the device.

### Stage 7: Inspect Driver Logs

Identify permission errors, communication failures or unsupported device behavior.

### Stage 8: Verify NUT Server Operation

Confirm that `upsd` can expose the driver's status information.

### Stage 9: Test UPS Status Queries

Use `upsc` to verify actual telemetry.

### Stage 10: Validate Shutdown Behavior

Test the configured shutdown policy using a controlled procedure that protects active workloads.

This methodology helps isolate problems without making unnecessary changes to unrelated system services.

---

## Technical Challenges

### USB Enumeration

A device may be physically connected but absent from the Linux USB subsystem.

### Device Permissions

The NUT service account may lack access to the USB device node.

### Driver Compatibility

Different UPS models require different communication drivers.

### Dynamic Device Paths

USB bus and device numbers may change after reconnection.

### Conflicting Processes

Multiple monitoring applications may interfere with device access.

### Appliance Configuration

Unraid and TrueNAS SCALE may manage UPS services differently from conventional Linux distributions.

### Network Monitoring

Ethernet connectivity does not necessarily imply support for a NUT-compatible network protocol.

### Shutdown Reliability

A functioning UPS status display does not guarantee that automatic shutdown will work.

These challenges reinforced the importance of validating each layer independently.

---

## Security Considerations

UPS monitoring software can influence server shutdown behavior.

Consequently, access to its administrative functions should be restricted.

The project emphasized:

- Using least-privilege service accounts.
- Restricting USB device permissions.
- Protecting NUT authentication credentials.
- Limiting network access to monitoring services.
- Avoiding unnecessary privileged containers.
- Keeping monitoring software updated.
- Securing remote management interfaces.
- Restricting shutdown command privileges.
- Maintaining configuration backups.

The evaluation reinforced that power management infrastructure is part of the broader security and availability architecture.

---

## Reliability and Recovery

A reliable UPS monitoring deployment should continue functioning after routine system events.

These include:

- Server reboot.
- USB reconnection.
- NUT service restart.
- UPS power cycling.
- Operating system updates.
- Network interruptions.
- Battery replacement.

The project emphasized persistent configuration and predictable service startup.

A temporary permission change that disappears after reboot is not a reliable long-term solution.

Likewise, a UPS that responds only after manual driver intervention is not suitable for unattended power protection without further remediation.

---

## Monitoring and Diagnostics

Useful operational indicators include:

- USB device presence.
- NUT driver status.
- UPS communication health.
- Battery charge.
- Battery runtime.
- UPS load.
- Input voltage.
- Output voltage.
- Utility power availability.
- Low-battery status.
- Monitoring client connectivity.
- Shutdown events.

The project emphasized distinguishing communication health from actual electrical conditions.

A monitoring failure should not automatically be interpreted as a utility power failure.

---

## Evaluation Outcome

The project examined the requirements for reliable UPS detection and monitoring through Network UPS Tools.

The investigation focused on Linux USB enumeration, device permissions, NUT driver selection and communication between UPS hardware and monitoring services.

The broader homelab included an APC SMC1500-2UC UPS intended to protect the NAS and associated infrastructure.

The project established a structured methodology for distinguishing USB detection failures, permission problems, driver incompatibility and NUT service configuration issues.

The available project history does not preserve the exact original permission error or establish a verified final driver configuration.

Accordingly, the document does not claim that a specific udev rule or service modification resolved the incident.

The documented outcome is the troubleshooting experience and the resulting understanding of UPS communication and power management architecture.

---

## Lessons Learned

USB device detection does not guarantee successful application communication.

NUT requires a compatible driver and appropriate device permissions.

Dynamic USB device paths should not be treated as permanent identifiers.

udev rules can provide persistent, targeted device permissions on compatible Linux installations.

Broad permission changes are not appropriate long-term solutions.

Multiple processes attempting to access the same UPS can cause communication problems.

NUT drivers, servers and monitoring clients perform different functions.

A functioning UPS status query does not establish that automatic shutdown is correctly configured.

UPS runtime must be evaluated against actual electrical load.

The project strengthened understanding of Linux USB management, NUT, service permissions, power monitoring and infrastructure reliability.

---

## Future Improvements

Future work may include:

- Recording the UPS USB vendor and product identifiers.
- Documenting the exact NUT driver configuration.
- Establishing persistent device permissions.
- Monitoring UPS communication health.
- Integrating UPS telemetry into Home Assistant.
- Creating battery-operation notifications.
- Measuring runtime under representative server loads.
- Testing controlled NAS shutdown.
- Coordinating shutdown across multiple hosts.
- Evaluating network-based UPS monitoring.
- Documenting battery replacement procedures.
- Maintaining NUT configuration backups.
- Testing recovery after USB reconnection.
- Implementing alerts for UPS communication loss.
- Developing a repeatable UPS validation checklist.

These improvements would support more reliable power protection across the homelab.

---

## Summary

The NUT USB Permission and UPS Detection Troubleshooting project focused on investigating communication between Linux-based server infrastructure and an APC uninterruptible power supply.

The work examined USB enumeration, device permissions, udev rules, NUT driver selection, monitoring services and automatic shutdown architecture.

A central finding was that reliable UPS integration depends on several independent layers, from physical connectivity through kernel device detection and service permissions to monitoring and shutdown coordination.

The project also highlighted the importance of preserving NAS availability during power disturbances and ensuring that monitoring configurations remain functional after reboots and updates.

Although the exact original USB permission errors and final repair outcome were not established in the available project record, the work provided a structured foundation for diagnosing UPS communication failures.

The project demonstrates knowledge of Linux systems administration, USB hardware, udev, Network UPS Tools, service permissions, UPS monitoring, power management and infrastructure reliability engineering.

---

## Technical References

- [Network UPS Tools — Official Documentation](https://networkupstools.org/docs/)
- [Network UPS Tools — Driver Compatibility List](https://networkupstools.org/stable-hcl.html)
- [Network UPS Tools — USB HID Driver](https://networkupstools.org/docs/man/usbhid-ups.html)
- [Network UPS Tools — upsc](https://networkupstools.org/docs/man/upsc.html)
- [Network UPS Tools — upsmon](https://networkupstools.org/docs/man/upsmon.html)
- [APC / Schneider Electric — UPS Support](https://www.apc.com/)
- [Home Assistant — Network UPS Tools Integration](https://www.home-assistant.io/integrations/nut/)
- [Unraid — Official Documentation](https://docs.unraid.net/)
- [Linux Kernel — USB Documentation](https://docs.kernel.org/usb/index.html)
