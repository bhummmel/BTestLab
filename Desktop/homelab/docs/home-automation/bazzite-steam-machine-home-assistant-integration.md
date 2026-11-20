# Bazzite Steam Machine and Home Assistant Integration

## Overview

The Bazzite Steam Machine and Home Assistant Integration project focused on integrating a Linux-based gaming computer into a self-hosted smart home infrastructure.

The system used Bazzite, a Fedora Atomic-based Linux distribution optimized for gaming, as the operating system for a dedicated Steam Machine.

The primary objective was to control the computer through Home Assistant, allowing it to participate in smart home automations alongside lighting, entertainment equipment, and network-connected power monitoring devices.

The project involved several interconnected technologies:

- Bazzite Linux.
- Home Assistant.
- Wake-on-LAN.
- SSH remote administration.
- Smart plug power monitoring.
- Logitech Harmony Hub.
- Steam activity detection.
- Automated lighting control.
- Linux system recovery.

An existing Home Assistant automation controlled the brightness of a living room lamp when the Steam Machine was in use.

The initial implementation relied on a combination of the Harmony Hub's active entertainment activity and the Steam account's online status.

This was subsequently redesigned to use electrical power consumption from the Steam Machine's smart switch.

The revised automation dimmed the living room lamp when power consumption exceeded 145 watts and restored full brightness when consumption fell below 125 watts.

This provided a more direct indication of the computer's activity without relying on Steam presence reporting.

The project also investigated enabling Wake-on-LAN to start the computer remotely and SSH to support controlled shutdown and system administration.

During this process, an attempted Bazzite `ujust` command failed because the requested recipe was not available in the installed configuration.

Additional troubleshooting involved recovering from a Linux emergency shell and identifying alternatives to directly editing Home Assistant's `configuration.yaml`.

The available project history confirms the lighting automation requirements, the attempted SSH configuration, and the system recovery investigation. It does not establish that remote power-on, remote shutdown, and all recovery procedures were successfully completed.

---

## Objectives

The primary objective was to integrate the Steam Machine into Home Assistant as a controllable and observable device.

Specific objectives included:

- Monitoring the Steam Machine's electrical power consumption.
- Detecting active gaming sessions.
- Automating living room lighting.
- Replacing unreliable presence-based detection.
- Integrating the Logitech Harmony Hub.
- Enabling Wake-on-LAN.
- Investigating remote shutdown over SSH.
- Configuring SSH on Bazzite.
- Understanding Bazzite's immutable operating system architecture.
- Avoiding unnecessary system modifications.
- Recovering from boot and emergency-shell problems.
- Creating Home Assistant dashboard controls.
- Reducing dependence on cloud-based presence detection.
- Maintaining safe and predictable power control.
- Improving the reliability of gaming-related automations.

The broader goal was to make the Steam Machine function as a native component of the smart home rather than an independently managed computer.

---

## Infrastructure Environment

The project operated within the existing Unraid and Home Assistant infrastructure.

| Component | Function |
|---|---|
| Bazzite Linux | Steam Machine operating system |
| Steam | Gaming platform |
| Home Assistant | Automation and device management |
| Unraid | Home Assistant container host |
| Logitech Harmony Hub | Entertainment activity control |
| Smart plug | Steam Machine power monitoring |
| Wake-on-LAN | Remote computer startup |
| SSH | Remote Linux administration |
| NetworkManager | Linux network configuration |
| systemd | Linux service management |
| `ujust` | Bazzite configuration utility |
| OPNsense | Network firewall and routing |
| Ethernet | Preferred Wake-on-LAN connectivity |
| Living Room Lamp 2 | Automated lighting endpoint |

Home Assistant was hosted on the Unraid server, while the Steam Machine operated as a separate computer on the local network.

The Steam Machine was also integrated into the living room entertainment system through the Harmony Hub.

---

## System Architecture

The architecture combined network-based computer control with smart plug telemetry and entertainment system state.

```text
                     Home Assistant
                           |
              +------------+------------+
              |            |            |
              v            v            v
          Smart Plug   Harmony Hub   Wake-on-LAN
              |            |            |
              v            v            v
         Power Sensor  Steam Machine   Ethernet
              |          Activity        |
              |                          |
              +------------+-------------+
                           |
                           v
                    Bazzite Steam
                       Machine
                           |
                    Gaming Activity
                           |
                           v
                    Power Consumption
                           |
                           v
                    Home Assistant
                           |
                  Threshold Automation
                           |
                           v
                   Living Room Lamp 2
```

The architecture separates three functions:

1. **State detection:** Determining whether the Steam Machine is actively being used.
2. **Power management:** Starting or shutting down the computer remotely.
3. **Environmental automation:** Adjusting lighting based on gaming activity.

These functions can operate independently.

For example, power-based lighting automation does not require SSH access to the Steam Machine.

---

## Why Bazzite Was Selected

Bazzite is a gaming-oriented Linux distribution built using Fedora Atomic technologies.

It provides an operating system designed for gaming hardware, Steam integration, and modern graphics support.

Relevant characteristics include:

- Fedora-based system architecture.
- Atomic operating system deployments.
- Steam and gaming-oriented configuration.
- Linux graphics driver integration.
- systemd service management.
- NetworkManager networking.
- Container-oriented software installation.
- `ujust` configuration helpers.
- Support for remote administration.
- Compatibility with conventional Linux networking tools.

The project used Bazzite as an alternative to a conventional Windows-based gaming computer.

This introduced additional flexibility for automation and remote management through standard Linux services.

However, Bazzite's atomic system architecture also required understanding how its configuration and update mechanisms differ from those of a traditional Linux distribution.

---

## Understanding Bazzite's Atomic Architecture

Bazzite uses an image-based operating system model.

Rather than relying primarily on traditional package installation directly into the running system, Bazzite manages its base operating system through atomic deployments.

This provides advantages such as:

- Reproducible system images.
- Controlled operating system updates.
- Deployment rollback capabilities.
- Reduced modification of the base system.
- Separation of applications from system components.

Bazzite also supports package layering through `rpm-ostree`.

However, unnecessary package layering can complicate upgrades and introduce dependency problems.

The project emphasized using existing system services and supported configuration tools whenever possible.

Wake-on-LAN and SSH generally do not require replacing the operating system or installing a separate remote management platform.

---

## Existing Steam Machine Automation

The original Home Assistant automation used Steam presence information to infer whether the computer was in use.

The logic combined two conditions:

- The Logitech Harmony Hub was set to the Steam Machine activity.
- The Steam account was online.

When both conditions were satisfied, Home Assistant dimmed the living room lamp.

When the Steam account went offline, the lamp returned to full brightness.

The intended lighting behavior was:

| Condition | Lamp Brightness |
|---|---|
| Steam Machine active | 15% |
| Steam Machine inactive | 100% |

This provided automatic lighting adjustment when using the living room gaming system.

---

## Limitations of Steam Presence Detection

Steam online status does not necessarily correspond to actual computer activity.

Potential discrepancies include:

- Steam remaining online while the computer is idle.
- Steam entering an offline state during a gaming session.
- Network interruptions.
- Steam client restarts.
- Presence updates being delayed.
- The computer being powered on without Steam running.
- Games being launched outside the Steam client.

The Harmony Hub activity provided additional context but did not guarantee that the Steam Machine was actually running a game.

The project therefore investigated a more direct method of detecting system activity.

---

## Power-Based Activity Detection

The revised automation used the electrical power consumption reported by the Steam Machine's smart plug.

A gaming computer typically consumes more power under substantial CPU and GPU load than it does while idle.

Home Assistant can monitor this consumption through a power sensor.

The revised logic was:

- Dim the living room lamp when power consumption exceeds 145 watts.
- Restore full brightness when power consumption falls below 125 watts.

This replaced Steam presence as the primary trigger.

The approach provided a hardware-level signal that did not depend on Steam account status.

---

## Understanding Power Thresholds

The automation used two different thresholds rather than a single threshold.

This technique is known as **hysteresis**.

Hysteresis prevents rapid switching when a measured value fluctuates near a boundary.

For example, using the same threshold for both activation and deactivation could cause repeated brightness changes if power consumption oscillated around that value.

The revised thresholds created a 20-watt separation between the two states.

```text
Power Consumption

       Active Gaming
             |
             v
        Above 145 W
             |
             v
       Dim Lamp to 15%


        Between 125 W
          and 145 W
             |
             v
       Preserve Current
         Lamp State


        Below 125 W
             |
             v
       Restore Lamp
          to 100%
```

The project emphasized that threshold selection should be based on the actual power characteristics of the computer.

The 145-watt and 125-watt values were the selected automation thresholds, not documented measurements of idle or gaming consumption.

---

## Original Threshold Experimentation

Before selecting the revised thresholds, the project considered lower power thresholds around 50 watts.

These values were subsequently replaced.

The higher thresholds were intended to distinguish more substantial system activity from lower-power operating states.

The final documented requirements were:

| Event | Threshold | Action |
|---|---|---|
| Power rises | Above 145 W | Dim lamp to 15% |
| Power falls | Below 125 W | Restore lamp to 100% |

The project emphasized that threshold-based automation is more reliable when the activation and deactivation conditions are deliberately separated.

---

## Home Assistant Power Sensor

The automation required a Home Assistant sensor representing the Steam Machine's electrical power consumption.

A typical entity might resemble:

```text
sensor.steam_machine_power
```

The actual entity ID depends on the installed smart plug and its integration.

The sensor should report power in watts.

Before creating the automation, the sensor can be inspected in:

**Developer Tools → States**

The project emphasized verifying that the sensor reports numerical values and updates reliably.

A sensor that becomes unavailable or stops reporting should not automatically be interpreted as zero power consumption.

---

## Automation: Dim Lamp Above 145 Watts

The following automation represents the documented behavior using current Home Assistant YAML syntax.

The power sensor entity ID is a placeholder and must be replaced with the actual smart plug sensor.

```yaml
alias: Dim Living Room Lamp When Steam Machine Is Active

description: >
  Dim Living Room Lamp 2 to 15 percent when
  the Steam Machine draws more than 145 watts.

triggers:
  - trigger: numeric_state
    entity_id: sensor.steam_machine_power
    above: 145

conditions: []

actions:
  - action: light.turn_on
    target:
      entity_id: light.living_room_lamp_2
    data:
      brightness_pct: 15

mode: single
```

The `numeric_state` trigger fires when the sensor crosses the configured threshold.

It does not continuously execute while the value remains above the threshold.

This behavior reduces unnecessary service calls.

---

## Automation: Restore Lamp Below 125 Watts

A second automation restores the lamp to full brightness when the Steam Machine's power consumption falls below 125 watts.

```yaml
alias: Restore Living Room Lamp When Steam Machine Is Idle

description: >
  Restore Living Room Lamp 2 to full brightness
  when Steam Machine power falls below 125 watts.

triggers:
  - trigger: numeric_state
    entity_id: sensor.steam_machine_power
    below: 125

conditions: []

actions:
  - action: light.turn_on
    target:
      entity_id: light.living_room_lamp_2
    data:
      brightness_pct: 100

mode: single
```

This implementation restores the lamp to 100% brightness.

It does not restore a previously saved brightness value.

The project used full brightness as the intended inactive state.

---

## Threshold Crossing Behavior

Home Assistant's numeric state triggers operate on threshold crossings.

For example:

1. Power consumption starts at 90 watts.
2. Consumption rises to 150 watts.
3. The activation threshold is crossed.
4. Home Assistant dims the lamp.
5. Consumption decreases to 135 watts.
6. No additional action occurs.
7. Consumption decreases to 120 watts.
8. The deactivation threshold is crossed.
9. Home Assistant restores the lamp.

This provides predictable behavior without requiring continuous polling inside the automation.

The project emphasized that threshold triggers must be understood as state transitions rather than continuously evaluated conditions.

---

## Optional Debouncing

Electrical power consumption can fluctuate rapidly.

A brief CPU or GPU workload may cause a temporary power spike.

Home Assistant supports a `for` condition on numeric state triggers to require a threshold to remain satisfied for a specified duration.

For example:

```yaml
triggers:
  - trigger: numeric_state
    entity_id: sensor.steam_machine_power
    above: 145
    for:
      seconds: 10
```

This requires the measured power to remain above 145 watts for ten seconds before the automation triggers.

The same approach can be applied to the lower threshold.

Debouncing was considered a possible reliability improvement, not a confirmed part of the original automation.

---

## Harmony Hub Integration

The Steam Machine was also represented as an entertainment activity within the Logitech Harmony Hub.

The Home Assistant installation included a Harmony remote entity associated with the living room entertainment system.

An activity named:

```text
Steam Machine
```

was used to select the appropriate entertainment configuration.

Harmony activity information could be read through Home Assistant and incorporated into automation conditions.

The original automation used this activity alongside Steam presence.

---

## Combining Harmony and Power Monitoring

Although power consumption became the primary activity indicator, Harmony information remained useful.

For example, an automation could require both:

- The Steam Machine's power consumption to exceed 145 watts.
- The Harmony Hub to be using the Steam Machine activity.

This would reduce the likelihood of changing living room lighting when the computer was running a background task but the television was displaying something else.

An illustrative condition is:

```yaml
conditions:
  - condition: state
    entity_id: remote.cornhub
    attribute: current_activity
    state: "Steam Machine"
```

The exact Harmony entity and activity attribute must be verified in the installed integration.

This condition was part of the earlier automation architecture; the documented revised request focused on power thresholds.

---

## Remote Power Management

The next major objective was to control the Steam Machine's power state through Home Assistant.

The desired functionality included:

- Turning the computer on remotely.
- Shutting it down cleanly.
- Monitoring whether it was online.
- Providing dashboard controls.
- Integrating power control into automations.

Two technologies were considered:

1. Wake-on-LAN for starting the computer.
2. SSH for issuing operating system shutdown commands.

These technologies address different stages of the computer's power lifecycle.

---

## Understanding Wake-on-LAN

Wake-on-LAN is a networking mechanism that allows a compatible computer to wake when its network adapter receives a specially formatted packet.

This packet is commonly called a **magic packet**.

The packet contains the destination network adapter's MAC address in a repeated sequence.

The network interface and motherboard firmware must support the relevant wake state.

Wake-on-LAN is commonly used for:

- Remote workstation startup.
- Server management.
- Gaming computer activation.
- Automated backup systems.
- Home automation.

The project considered Wake-on-LAN as the preferred method for remotely powering on the Steam Machine.

---

## Wake-on-LAN Requirements

Wake-on-LAN depends on both hardware and software configuration.

Typical requirements include:

- A compatible network adapter.
- Motherboard firmware support.
- Wake-on-LAN enabled in BIOS or UEFI.
- Appropriate network adapter settings.
- Standby power available to the network interface.
- A supported power state.
- Network delivery of the magic packet.

Wired Ethernet is generally preferable to Wi-Fi for this use case.

Wake behavior can differ between sleep, hibernation, and full shutdown.

The project emphasized verifying the hardware's actual wake capabilities rather than assuming that every computer supports Wake-on-LAN from every power state.

---

## Inspecting Network Interfaces on Bazzite

The Steam Machine's network interfaces can be inspected using:

```bash
ip link
```

For a more detailed view:

```bash
ip addr
```

NetworkManager connections can be listed using:

```bash
nmcli connection show
```

The physical Ethernet adapter may have an interface name such as:

```text
enp3s0
```

The actual name depends on the hardware.

The MAC address of the relevant interface is required for Home Assistant's Wake-on-LAN integration.

---

## Checking Wake-on-LAN Support

On Linux, Wake-on-LAN capabilities can be inspected using `ethtool` when the utility is available.

```bash
sudo ethtool enp3s0
```

Relevant output may resemble:

```text
Supports Wake-on: pumbg
Wake-on: g
```

The `g` flag indicates wake support using a magic packet.

If the current setting does not include `g`, Wake-on-LAN may not be enabled.

The example is illustrative and is not a recorded output from the Steam Machine.

---

## Configuring Wake-on-LAN with NetworkManager

Bazzite uses NetworkManager for network configuration.

Wake-on-LAN can be enabled for a wired connection using:

```bash
sudo nmcli connection modify \
  "Wired connection 1" \
  802-3-ethernet.wake-on-lan magic
```

The connection name must be replaced with the actual NetworkManager connection.

The setting can be inspected using:

```bash
nmcli connection show "Wired connection 1" \
  | grep wake-on-lan
```

The project emphasized using NetworkManager's persistent connection settings rather than relying exclusively on temporary interface changes.

---

## Bazzite Wake-on-LAN Helpers

Bazzite provides `ujust` commands for common system configuration tasks.

Depending on the installed Bazzite version, Wake-on-LAN management may be available through a helper recipe.

Available recipes can be listed using:

```bash
ujust
```

Or searched using:

```bash
ujust | grep -i wol
```

The exact command names may vary between Bazzite releases.

The project emphasized checking the installed recipe list before relying on commands found in older documentation.

---

## Home Assistant Wake-on-LAN Integration

Home Assistant includes a native Wake-on-LAN integration.

The integration can send magic packets to compatible devices.

It can be configured through:

**Settings → Devices & Services → Add Integration → Wake on LAN**

The configuration requires the target network adapter's MAC address.

Home Assistant can then expose a button for sending the wake packet.

This provides a graphical alternative to manually editing `configuration.yaml`.

---

## Example Wake-on-LAN Automation

The following example demonstrates how Home Assistant can send a magic packet.

```yaml
alias: Wake Steam Machine

triggers:
  - trigger: state
    entity_id: input_button.wake_steam_machine

actions:
  - action: wake_on_lan.send_magic_packet
    data:
      mac: "AA:BB:CC:DD:EE:FF"

mode: single
```

The MAC address is a placeholder.

The automation assumes that a helper button named `input_button.wake_steam_machine` has already been created.

The Wake-on-LAN integration must also be installed.

---

## Wake-on-LAN Limitations

Wake-on-LAN sends a packet but does not inherently confirm that the computer has started.

A successful service call indicates that Home Assistant attempted to send the packet.

It does not guarantee that:

- The network adapter received the packet.
- The motherboard accepted the wake request.
- The operating system booted.
- The network connection became available.
- Steam started successfully.

The project emphasized separating the wake command from online-state monitoring.

---

## Monitoring the Steam Machine's Online State

A reliable dashboard should indicate whether the Steam Machine is actually reachable.

Possible monitoring methods include:

- ICMP ping.
- Network device tracking.
- SSH connectivity.
- Smart plug power consumption.
- Steam client presence.
- Router-based device tracking.

Each method has limitations.

For example, a computer may respond to ping while Steam is not running.

Similarly, power consumption can indicate that the computer is active without confirming that its operating system is functioning correctly.

The project emphasized choosing monitoring methods according to the specific state being measured.

---

## Remote Shutdown with SSH

Wake-on-LAN can start a compatible computer, but it does not provide a standard method for shutting it down.

For a Linux-based Steam Machine, SSH provides a practical remote administration mechanism.

Home Assistant can connect to the computer and issue a controlled shutdown command.

A typical Linux shutdown command is:

```bash
sudo systemctl poweroff
```

This requests a clean operating system shutdown.

It is preferable to abruptly disconnecting power through a smart plug.

---

## Configuring SSH on Bazzite

The project attempted to enable SSH using a Bazzite `ujust` command.

The command used was:

```bash
ujust ssh enable
```

The system returned an error indicating that the requested recipe was not available.

```text
error: Justfile does not contain recipe `ssh`
```

This was a confirmed troubleshooting event.

The error indicated that the installed Bazzite configuration did not provide the requested recipe under that name.

It did not necessarily mean that the SSH server was unavailable.

---

## Understanding `ujust`

Bazzite provides the `ujust` command as a convenient interface to predefined configuration recipes.

These recipes automate common maintenance and setup tasks.

However, the available commands can differ between releases and system images.

The project emphasized checking the available commands directly:

```bash
ujust
```

To search for SSH-related recipes:

```bash
ujust | grep -i ssh
```

Current Bazzite source code includes an SSH management recipe that can enable the `sshd` service.

However, the original system's error demonstrates why administrators should not assume that a particular recipe exists on every installed image.

---

## Enabling SSH Directly with systemd

Bazzite uses systemd for service management.

Where the OpenSSH server is installed, SSH can be enabled directly:

```bash
sudo systemctl enable --now sshd
```

The command enables the service at startup and starts it immediately.

The service state can be checked using:

```bash
systemctl status sshd
```

To verify whether SSH is listening:

```bash
sudo ss -tlnp | grep ':22'
```

This provides a direct alternative to relying on a `ujust` helper.

---

## Testing SSH Connectivity

From another computer on the local network, SSH can be tested using:

```bash
ssh USERNAME@STEAM_MACHINE_IP
```

The username and IP address must match the actual Bazzite installation.

Successful authentication establishes that the SSH server is reachable and accepting the configured credentials.

The project emphasized testing SSH independently before attempting to integrate it with Home Assistant.

---

## SSH Key Authentication

For automated remote management, SSH key authentication is preferable to embedding a reusable account password in scripts.

A dedicated SSH key can be generated using:

```bash
ssh-keygen -t ed25519
```

The public key can then be installed on the Steam Machine.

For example:

```bash
ssh-copy-id USERNAME@STEAM_MACHINE_IP
```

This command assumes the SSH client has access to `ssh-copy-id`.

The project emphasized that Home Assistant's runtime environment must have access to the private key used for authentication.

The corresponding public key belongs on the Bazzite machine.

---

## Home Assistant Container Considerations

Home Assistant was deployed as a Docker container on Unraid.

This affected how remote commands could be executed.

A shell command running inside Home Assistant executes in the container's environment, not directly on the Unraid host.

Consequently, SSH integration requires considering:

- Availability of an SSH client.
- Persistent access to SSH keys.
- Container filesystem permissions.
- Network connectivity.
- Host key verification.
- Command execution permissions.

The project emphasized that commands working in an Unraid terminal do not necessarily work inside the Home Assistant container.

---

## Remote Shutdown Permissions

A dedicated automation account should have only the permissions required for the intended task.

For example, an SSH account might be permitted to execute a specific shutdown command without requiring an interactive password.

The command path should be verified using:

```bash
command -v systemctl
```

A narrowly scoped sudoers rule can then be created using:

```bash
sudo visudo
```

An illustrative rule is:

```text
ha-remote ALL=(root) NOPASSWD: /usr/bin/systemctl poweroff
```

This example assumes a dedicated account named `ha-remote` and that the command path is `/usr/bin/systemctl`.

The exact configuration must be verified against the installed operating system.

The project emphasized avoiding unrestricted passwordless sudo access.

---

## Example Home Assistant Shutdown Command

For installations that support shell commands, an illustrative configuration is:

```yaml
shell_command:
  shutdown_steam_machine: >-
    ssh -i /config/ssh/steam_machine_key
    -o BatchMode=yes
    -o StrictHostKeyChecking=yes
    ha-remote@STEAM_MACHINE_IP
    'sudo /usr/bin/systemctl poweroff'
```

This example requires:

- A configured SSH client.
- A persistent private key.
- A verified SSH host key.
- A corresponding account on Bazzite.
- A narrowly scoped sudoers rule.
- Network connectivity.

The command is not a verified copy of the original deployment.

---

## Home Assistant Configuration Limitations

During the project, direct editing of Home Assistant's `configuration.yaml` was not available through the user's normal workflow.

This complicated integration methods that depended on YAML configuration.

The investigation therefore considered UI-based alternatives.

These included:

- Wake-on-LAN integration.
- Helper buttons.
- Automation actions.
- Dashboard controls.
- Existing service integrations.

The project emphasized using Home Assistant's graphical configuration capabilities where practical.

For remote shutdown, an additional integration or preconfigured command execution mechanism may still be required.

---

## Creating a Unified Power Control Interface

The desired user experience was a single Home Assistant dashboard control for the Steam Machine.

The control would ideally provide:

- Power-on command.
- Power-off command.
- Online/offline indication.
- Current electrical power consumption.
- Active gaming indication.

A conceptual dashboard layout is:

```text
+-----------------------------------+
| Steam Machine                     |
+-----------------------------------+
|                                   |
| Status: Online                    |
|                                   |
| Power Consumption: 185 W          |
|                                   |
| [ Wake ]       [ Shut Down ]      |
|                                   |
| Gaming Activity: Active           |
|                                   |
+-----------------------------------+
```

The project emphasized that this dashboard should represent actual device state rather than assuming that a successful command means the computer is online or offline.

---

## Combining Power and Network State

A more reliable device-state model can combine multiple sensors.

For example:

| Network Reachability | Power Consumption | Possible Interpretation |
|---|---|---|
| Offline | Very low | Shut down or sleeping |
| Online | Low | Running but idle |
| Online | High | Active workload |
| Offline | High | Booting, network failure, or system problem |

These interpretations are not definitive.

Power consumption depends on the hardware and connected peripherals.

The project emphasized treating these values as indicators rather than absolute proof of a particular operating state.

---

## Emergency Shell Recovery

During the broader Bazzite troubleshooting process, the Steam Machine entered a Linux emergency environment.

This required investigating how to exit the emergency shell and restore normal boot behavior.

A Linux emergency shell may appear when the system cannot complete normal startup.

Potential causes include:

- Filesystem mount failures.
- Invalid `/etc/fstab` entries.
- Missing storage devices.
- Filesystem corruption.
- Failed system dependencies.
- Boot configuration problems.
- Interrupted system updates.

The available project history confirms the emergency-shell troubleshooting event but does not establish its exact root cause.

---

## Diagnosing Emergency Mode

A system entering emergency mode should be investigated before attempting repeated reboots.

Useful commands may include:

```bash
systemctl --failed
```

```bash
journalctl -xb
```

```bash
lsblk -f
```

```bash
cat /etc/fstab
```

These commands can help identify failed units, mount problems, and storage configuration issues.

Their availability depends on the specific recovery environment.

The project emphasized gathering diagnostic information before changing filesystem or boot configuration.

---

## Exiting Emergency Mode

Depending on the failure, normal boot may be resumed after correcting the underlying issue.

A common systemd command is:

```bash
systemctl default
```

This requests the system's default target.

However, it will not resolve an unresolved mount failure or other dependency problem.

If the underlying issue remains, the system may return to emergency mode.

The project emphasized that exiting the emergency shell is not equivalent to repairing the cause of the boot failure.

---

## Atomic Deployment Recovery

Bazzite's atomic deployment model can provide recovery options when a system update or deployment causes problems.

The installed deployments can be inspected using:

```bash
rpm-ostree status
```

Where appropriate, an earlier deployment may be selected through the boot menu or restored using supported rollback procedures.

The project emphasized that rollback is most relevant when the problem is associated with an operating system deployment.

It should not be assumed to repair unrelated storage hardware or filesystem failures.

---

## Troubleshooting Methodology

The project can be organized into several diagnostic stages.

### Stage 1: Verify Power Telemetry

Confirm that Home Assistant receives accurate power readings from the smart plug.

### Stage 2: Validate Lighting Thresholds

Observe the Steam Machine's power consumption during idle and active workloads.

### Stage 3: Test Lighting Automations

Confirm that the lamp dims above 145 watts and restores below 125 watts.

### Stage 4: Inspect Harmony Activity

Verify that Home Assistant can identify the Steam Machine entertainment activity.

### Stage 5: Inspect Bazzite Networking

Identify the Ethernet interface, MAC address, and NetworkManager connection.

### Stage 6: Verify Wake-on-LAN Support

Check firmware settings and network adapter capabilities.

### Stage 7: Configure Wake-on-LAN

Enable magic-packet wake behavior where supported.

### Stage 8: Test Wake-on-LAN Independently

Send a magic packet from another device before configuring Home Assistant.

### Stage 9: Configure SSH

Enable `sshd` and test authentication from another computer.

### Stage 10: Configure Secure Shutdown

Establish a restricted command execution method.

### Stage 11: Integrate Home Assistant Controls

Create dashboard buttons and automations.

### Stage 12: Validate Recovery Behavior

Test the system after normal shutdown, reboot, and Home Assistant restart.

This methodology separates electrical activity detection, network wake behavior, remote administration, and Home Assistant configuration.

---

## Security Considerations

Remote power management introduces additional security requirements.

### SSH Authentication

Key-based authentication should be used for automated connections.

### Least Privilege

A dedicated account should be restricted to necessary administrative commands.

### Network Access

SSH should be accessible only from trusted network segments or authorized remote-access systems.

### Firewall Configuration

OPNsense and host firewall rules should permit only the required connections.

### Credential Protection

Private SSH keys should not be stored in public repositories or exposed through dashboard configuration.

### Host Key Verification

SSH clients should verify the Steam Machine's host key to reduce the risk of connecting to an unintended system.

### Power Control

The smart plug should not be used as the normal shutdown mechanism.

### Recovery Access

Local console access should remain available in case remote administration fails.

The project emphasized that convenience automations must not compromise the reliability or security of the gaming computer.

---

## Reliability Considerations

The Steam Machine was intended to operate as both a gaming system and a smart home-controlled device.

Important reliability requirements included:

- Persistent Wake-on-LAN configuration.
- Stable Ethernet connectivity.
- Correct BIOS or UEFI settings.
- SSH service startup.
- Reliable power telemetry.
- Appropriate automation thresholds.
- Safe operating system shutdown.
- Accurate online-state reporting.
- Recovery from interrupted updates.
- Avoiding conflicting automation actions.

The project emphasized that remote control should fail safely.

For example, an unavailable SSH connection should not trigger an automatic hard power cut unless a separate, deliberate recovery procedure has been established.

---

## Technical Challenges

### Steam Presence Reliability

Steam online status did not provide a sufficiently direct indication of active computer usage.

### Power Threshold Selection

The automation required thresholds that distinguished substantial activity from lower-power states.

### Automation Hysteresis

Separate activation and deactivation thresholds were necessary to avoid rapid brightness changes.

### Bazzite Configuration Differences

The attempted `ujust ssh enable` command failed because the expected recipe was unavailable.

### Remote Startup

Wake-on-LAN depended on firmware, network adapter, and operating system configuration.

### Remote Shutdown

SSH required authentication, permissions, and a command execution mechanism accessible from Home Assistant.

### Containerized Home Assistant

The Home Assistant container had a different execution environment from the Unraid host.

### Configuration Access

Direct editing of Home Assistant YAML configuration was not available through the normal workflow.

### System Recovery

The Steam Machine entered an emergency environment that required Linux boot troubleshooting.

These challenges demonstrated the importance of separating application-level automation from operating system and hardware management.

---

## Evaluation Outcome

The project redesigned the Steam Machine lighting automation around electrical power consumption.

The documented thresholds were:

- Above 145 watts: dim Living Room Lamp 2 to 15%.
- Below 125 watts: restore Living Room Lamp 2 to 100%.

This provided a more direct activity indicator than Steam presence alone.

The project also investigated Wake-on-LAN and SSH as mechanisms for remote startup and shutdown.

An attempted Bazzite SSH helper command failed because the installed system did not contain the expected `ujust` recipe.

Alternative configuration methods using systemd and NetworkManager were identified.

Additional troubleshooting addressed a Bazzite emergency-shell condition and Home Assistant configuration limitations.

The available project history does not confirm that the complete remote power-management workflow was operational at the conclusion of the investigation.

The documented outcome is therefore the power-based automation design and the technical work toward remote management.

---

## Lessons Learned

Electrical power monitoring can provide a useful hardware-level indicator of computer activity.

Power consumption and Steam presence measure different aspects of system state.

Hysteresis improves automation stability by preventing repeated transitions around a single threshold.

Home Assistant numeric state triggers operate on threshold crossings.

Wake-on-LAN provides remote startup but does not confirm that the target computer has booted.

SSH provides a practical mechanism for controlled Linux shutdown.

Bazzite's `ujust` recipes can vary between installed versions.

Standard systemd and NetworkManager tools remain important for troubleshooting Bazzite systems.

Containerized Home Assistant installations require careful consideration of command execution environments.

An emergency shell indicates an underlying startup problem that should be diagnosed rather than bypassed.

Remote administration should use restricted credentials and preserve local recovery access.

The project strengthened understanding of Home Assistant, Bazzite Linux, systemd, NetworkManager, Wake-on-LAN, SSH, electrical power monitoring, and automation design.

---

## Future Improvements

Future work may include:

- Confirming Wake-on-LAN functionality from full shutdown.
- Implementing a persistent NetworkManager Wake-on-LAN configuration.
- Establishing SSH key-based remote administration.
- Creating a restricted shutdown account.
- Integrating remote shutdown into Home Assistant.
- Adding a unified Steam Machine dashboard.
- Combining network and power monitoring.
- Testing automatic startup after power restoration.
- Adding notifications for unexpected system shutdowns.
- Refining lighting thresholds using recorded power measurements.
- Implementing debounce intervals.
- Preventing lighting automations from interfering with manual overrides.
- Saving and restoring previous lamp brightness.
- Integrating Harmony activity selection with power control.
- Documenting Bazzite recovery and rollback procedures.
- Testing behavior after Unraid and Home Assistant restarts.
- Evaluating Sunshine and Moonlight for remote game streaming.
- Adding optional remote access through the existing Tailscale infrastructure.

These improvements would make the Steam Machine a more fully integrated and manageable part of the smart home.

---

## Summary

The Bazzite Steam Machine and Home Assistant Integration project focused on integrating a Linux gaming computer into a self-hosted smart home environment.

The existing infrastructure included Home Assistant, a Logitech Harmony Hub, smart plug power monitoring, and automated living room lighting.

The original automation relied on Steam presence and Harmony activity information.

The revised design used electrical power consumption, dimming the living room lamp above 145 watts and restoring full brightness below 125 watts.

This introduced hysteresis and reduced dependence on application-level presence reporting.

The project also investigated Wake-on-LAN for remote startup, SSH for controlled shutdown, and Bazzite's system administration tools.

Troubleshooting included an unavailable `ujust` SSH recipe, Home Assistant configuration limitations, and recovery from a Linux emergency environment.

Although the complete remote power-management workflow was not verified in the available project history, the work established the architecture and diagnostic procedures necessary for further integration.

The project demonstrates practical experience with Linux systems administration, Home Assistant automations, network power management, smart plug telemetry, SSH, Wake-on-LAN, and infrastructure troubleshooting.

---

## Technical References

- [Bazzite — Official Website](https://bazzite.gg/)
- [Bazzite — Official Documentation](https://docs.bazzite.gg/)
- [Bazzite — GitHub Repository](https://github.com/ublue-os/bazzite)
- [Bazzite — ujust Commands](https://docs.bazzite.gg/Installing_and_Managing_Software/ujust/)
- [Bazzite — Package Layering](https://docs.bazzite.gg/Installing_and_Managing_Software/rpm-ostree/)
- [Home Assistant — Official Documentation](https://www.home-assistant.io/docs/)
- [Home Assistant — Wake-on-LAN Integration](https://www.home-assistant.io/integrations/wake_on_lan/)
- [Home Assistant — Automation Triggers](https://www.home-assistant.io/docs/automation/trigger/)
- [Home Assistant — Numeric State Triggers](https://www.home-assistant.io/docs/automation/trigger/#numeric-state-trigger)
- [Home Assistant — Command Line Integration](https://www.home-assistant.io/integrations/command_line/)
- [Home Assistant — Shell Command Integration](https://www.home-assistant.io/integrations/shell_command/)
- [NetworkManager — Official Documentation](https://networkmanager.dev/docs/)
- [systemd — Official Documentation](https://systemd.io/)
- [OpenSSH — Official Website](https://www.openssh.com/)
- [Fedora Atomic Desktops](https://fedoraproject.org/atomic-desktops/)
- [rpm-ostree — Official Documentation](https://coreos.github.io/rpm-ostree/)
- [Unraid — Official Documentation](https://docs.unraid.net/)
