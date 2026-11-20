# Linux Vehicle Computer and SDR Integration

## Overview

The Linux Vehicle Computer and SDR Integration project investigated installing a dedicated Linux-based computer inside a first-generation Toyota Sequoia to create a multifunction mobile computing, radio monitoring, and navigation platform.

The project centered on a Logic Supply RT130-i7-P industrial computer equipped with a third-generation Intel Core i7 processor.

Unlike a conventional aftermarket infotainment system, the proposed computer would provide a general-purpose Linux environment capable of supporting software-defined radio (SDR), GPS navigation, offline mapping, radio communications interfaces, and other specialized applications.

The intended installation was a 2002 Toyota Sequoia equipped with a factory JBL audio system and an aftermarket Pioneer multimedia receiver.

The project aimed to integrate the computer with the vehicle's existing electronics while preserving a practical touchscreen interface.

Several possible functions were investigated:

- Software-defined radio reception and recording.
- Amateur radio monitoring.
- CB radio integration.
- Satellite signal reception.
- Offline GPS navigation.
- Touchscreen computer operation.
- Linux-based application management.
- Vehicle-compatible power delivery.
- Audio integration with the existing stereo system.
- Remote access to the home homelab when connectivity was available.

The project also considered the use of a DisplayPort-to-HDMI adapter, USB touchscreen connectivity, and a dedicated GPS receiver.

A central design requirement was that the computer should remain useful without an internet connection.

The available project history confirms the hardware selection and system design investigation. It does not establish that the complete in-vehicle installation, SDR configuration, GPS integration, or radio transmission interfaces were successfully commissioned.

**Project status:** Hardware evaluation and integration planning, with implementation and validation remaining to be documented.

---

## Project Objectives

The primary objective was to create a flexible, Linux-powered mobile workstation integrated into the vehicle.

Specific objectives included:

- Evaluating the Logic Supply RT130-i7-P industrial computer.
- Selecting a suitable Linux operating system.
- Integrating a touchscreen display.
- Converting DisplayPort video output to HDMI.
- Supporting USB-based touchscreen input.
- Providing reliable vehicle power.
- Implementing ignition-aware startup and shutdown.
- Investigating SDR hardware compatibility.
- Supporting wideband radio monitoring.
- Recording received radio signals.
- Integrating amateur radio equipment.
- Evaluating CB radio interfaces.
- Supporting satellite signal reception.
- Integrating a dedicated GPS receiver.
- Providing offline navigation.
- Connecting computer audio to the vehicle's stereo.
- Evaluating remote connectivity to the home network.
- Maintaining serviceability and modularity.

The broader objective was to build a mobile computing platform that could be expanded over time without being limited by the software capabilities of a proprietary head unit.

---

## Vehicle Platform

The project used a first-generation Toyota Sequoia.

| Component | Description |
|---|---|
| Vehicle | Toyota Sequoia |
| Model year | 2002 |
| Generation | First |
| Factory audio | JBL amplified audio system |
| Aftermarket receiver | Pioneer multimedia receiver |
| Computer | Logic Supply RT130-i7-P |
| Processor | Third-generation Intel Core i7 |
| Operating system | Linux, distribution to be finalized |
| Display connection | DisplayPort-to-HDMI under evaluation |
| Touch input | USB touchscreen under evaluation |
| Navigation | Dedicated GPS receiver proposed |
| Radio | SDR, amateur radio, and CB integration proposed |

The vehicle offered sufficient physical space for a dedicated computer, radio equipment, power electronics, and associated cabling.

The project emphasized retaining normal vehicle functionality while adding computing capabilities.

---

## Why an Industrial Computer Was Considered

A conventional desktop computer is not optimized for installation inside a vehicle.

Automotive environments introduce several challenges:

- Voltage fluctuations.
- Electrical noise.
- Engine cranking voltage drops.
- Vibration.
- Elevated cabin temperatures.
- Limited mounting space.
- Irregular shutdown conditions.
- Restricted ventilation.

An industrial or embedded computer can offer advantages such as compact construction, integrated I/O, and more flexible mounting options.

However, the specific RT130-i7-P unit's environmental ratings, power-input requirements, and automotive suitability must be verified from its hardware documentation.

An industrial designation alone does not guarantee compatibility with direct vehicle battery power.

---

## Logic Supply RT130-i7-P

The selected computer was a Logic Supply RT130-i7-P system.

The previously identified unit incorporated a third-generation Intel Core i7 processor.

The computer was considered suitable for workloads beyond those of a conventional embedded media player.

Potential applications included:

- SDR spectrum visualization.
- Signal demodulation.
- Audio recording.
- GPS processing.
- Offline map rendering.
- Serial device control.
- USB radio interfaces.
- Network utilities.
- Local databases.
- Custom dashboards.

The exact installed RAM, storage capacity, network interfaces, expansion slots, and power specifications were not established in the available project history.

These should be documented from the physical unit before finalizing the installation.

---

## Proposed System Architecture

The design separated computing, radio reception, navigation, and vehicle integration into distinct components.

```text
                    Toyota Sequoia
                          |
                  Vehicle Electrical
                       System
                          |
                          v
                 Automotive DC-DC
                    Power Supply
                          |
                          v
                Logic Supply RT130
                    Linux PC
                          |
         +----------------+----------------+
         |                |                |
         v                v                v
      DisplayPort        USB              USB
         |                |                |
         v                v                v
      HDMI Display    Touch Input      SDR Receiver
         |                                 |
         v                                 v
      Dashboard                        RF Antenna
      Interface

                    Additional I/O
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
             GPS       Radio CAT    Network
           Receiver    Interface    Interface
              |           |           |
              v           v           v
         Navigation   Radio Control  Home Network
```

The architecture was designed to remain modular.

For example, the SDR receiver could be replaced without changing the GPS subsystem or display hardware.

---

## Operating System Evaluation

Linux was selected as the preferred software platform.

Relevant advantages included:

- Extensive SDR software support.
- Native serial device interfaces.
- USB hardware compatibility.
- Open-source navigation applications.
- Customizable desktop environments.
- SSH remote administration.
- Scriptable system services.
- Local data storage.
- Minimal dependence on cloud services.

Potential distributions included Debian, Ubuntu, Fedora, and other Linux variants.

A lightweight desktop environment could reduce system overhead on the third-generation processor.

The project did not establish a final operating system installation for the RT130.

---

## Touchscreen Integration

A central requirement was controlling the computer from the vehicle dashboard.

The proposed display would use HDMI for video and USB for touchscreen input.

This is a common architecture for external touchscreens.

```text
                Linux Vehicle Computer
                         |
              +----------+----------+
              |                     |
              v                     v
          DisplayPort               USB
              |                     |
              v                     |
         DP-to-HDMI Adapter          |
              |                     |
              v                     v
             HDMI               USB Touch
              |                     |
              +----------+----------+
                         |
                         v
                 Touchscreen Display
```

The HDMI connection carries video.

The USB connection carries touch input.

A DisplayPort-to-HDMI adapter does not transmit USB touchscreen events.

The project emphasized keeping these interfaces conceptually separate.

---

## DisplayPort-to-HDMI Conversion

The computer's DisplayPort output was considered for connection to an HDMI display.

Two general adapter types exist:

### Passive DisplayPort Adapter

Relies on the computer's DisplayPort output supporting the appropriate HDMI-compatible signaling mode.

### Active DisplayPort Adapter

Converts DisplayPort signaling into HDMI using an active conversion circuit.

An active adapter may be required when the computer does not support the necessary passive conversion mode.

The project emphasized verifying the computer's video output capabilities before selecting the adapter.

The final adapter model was not established in the available record.

---

## USB Touchscreen Support

Many external touchscreens expose a USB Human Interface Device (HID).

Linux often supports standard HID touch devices without manufacturer-specific drivers.

However, compatibility depends on the touchscreen controller and firmware.

Useful diagnostic commands include:

```bash
lsusb
```

```bash
sudo dmesg | tail -n 50
```

```bash
libinput list-devices
```

The `libinput` command requires the corresponding utility package.

A touchscreen may also require calibration or coordinate mapping if multiple displays are connected.

The project emphasized verifying touchscreen support before permanently mounting the display.

---

## Touchscreen User Interface

A conventional Linux desktop may be difficult to operate on a small vehicle-mounted touchscreen.

Potential improvements include:

- Large application launchers.
- Full-screen applications.
- Oversized buttons.
- High-contrast interfaces.
- On-screen keyboard support.
- Simplified navigation.
- Automatic application startup.
- Minimal window decorations.

A conceptual interface could provide:

```text
+---------------------------------------+
|          VEHICLE COMPUTER             |
+---------------------------------------+
|                                       |
|   [ Navigation ]   [ SDR Receiver ]   |
|                                       |
|   [ Radio ]        [ Media ]          |
|                                       |
|   [ Diagnostics ]  [ Settings ]       |
|                                       |
+---------------------------------------+
```

The objective was to reduce interaction complexity while maintaining access to advanced Linux applications.

The interface should be operated only when the vehicle is parked or by a passenger for tasks requiring significant attention.

---

## Software-Defined Radio Integration

Software-defined radio was one of the principal reasons for selecting a general-purpose computer.

An SDR receiver converts radio-frequency signals into digital samples that can be processed by software.

Depending on the receiver, applications can provide:

- Spectrum displays.
- Waterfall visualization.
- AM demodulation.
- FM demodulation.
- SSB reception.
- Digital signal decoding.
- Signal recording.
- Frequency scanning.
- Spectrum analysis.

The project considered SDR as a flexible alternative to installing a separate dedicated receiver for every radio service.

---

## SDR Architecture

A typical SDR receiver chain is:

```text
                  RF Antenna
                      |
                      v
                 SDR Receiver
                      |
               USB / Ethernet
                      |
                      v
                 Linux Computer
                      |
                 SDR Software
                      |
             +--------+--------+
             |        |        |
             v        v        v
          Spectrum   Audio   Recording
          Display    Output    Files
```

The antenna, receiver, and software must all support the intended frequency range and signal type.

The computer performs the processing, but the SDR hardware determines important RF capabilities.

---

## SDR Hardware Considerations

The project did not establish a final SDR receiver model.

Potential categories included:

| SDR Category | Typical Application |
|---|---|
| RTL-SDR | Inexpensive receive-only monitoring |
| Airspy | Higher-performance receive-only applications |
| SDRplay | Wideband receive applications |
| HackRF | Experimental half-duplex receive/transmit |
| LimeSDR | Advanced SDR experimentation |
| USRP | Research and professional SDR development |

These devices differ significantly in dynamic range, tuning range, sampling bandwidth, and transmission capabilities.

The project emphasized that a receiver-only SDR cannot transmit merely because its software provides transmission controls.

---

## RTL-SDR as an Initial Receiver

An RTL-SDR-class receiver was considered a practical entry point for radio monitoring.

Potential advantages include:

- Relatively low cost.
- USB connectivity.
- Extensive Linux support.
- Broad software compatibility.
- Compact form factor.

A compatible receiver could be used for applications such as FM broadcast reception, selected VHF/UHF monitoring, and other supported signals.

Actual frequency coverage depends on the specific SDR model.

The project did not confirm that an RTL-SDR was installed in the vehicle.

---

## Linux SDR Software

Several Linux applications were relevant to the proposed system.

### SDR++

SDR++ is a cross-platform SDR application with a graphical interface, waterfall display, and support for multiple receiver backends.

It was a candidate for general-purpose radio monitoring.

### GQRX

GQRX provides a graphical SDR receiver interface and supports several SDR devices through compatible drivers.

### GNU Radio

GNU Radio provides a signal-processing framework for building custom radio applications.

It is particularly useful for experimental workflows and protocol development.

### SoapySDR

SoapySDR provides a hardware abstraction layer used by various SDR applications.

The project emphasized selecting software according to the intended workload rather than installing every available SDR package.

---

## Example SDR Diagnostics

After connecting a compatible SDR receiver, the first step is to confirm that Linux detects it.

```bash
lsusb
```

For a supported RTL-SDR receiver, the following utility can test basic communication:

```bash
rtl_test
```

The command requires an appropriate RTL-SDR driver installation.

A successful hardware test does not guarantee satisfactory reception.

Antenna suitability, RF interference, gain settings, and receiver limitations must also be considered.

---

## SDR Recording

The vehicle computer was intended to support recording received radio signals.

Two general recording methods were considered.

### Demodulated Audio Recording

Stores the audio recovered from a received signal.

This is useful for reviewing monitored transmissions.

### IQ Recording

Stores complex baseband samples before demodulation.

IQ recordings preserve more signal information and can support later analysis.

However, IQ files can become very large.

For example, recording complex 8-bit IQ data at 2.4 million samples per second produces approximately:

\[
2.4 \times 10^6 \times 2 = 4.8\ \text{MB/s}
\]

This is approximately 17.3 GB per hour using decimal storage units.

The project emphasized planning storage capacity according to the intended recording format and sample rate.

---

## Storage Architecture

The RT130 would require local storage for:

- Operating system files.
- SDR applications.
- Radio recordings.
- Offline maps.
- GPS tracks.
- Application logs.
- Configuration backups.

A solid-state drive would be preferable to a mechanical hard drive in a vibration-prone installation.

The exact installed storage hardware was not documented.

The project also considered transferring files to the Unraid server when the vehicle returned to the home network.

This would separate mobile data collection from long-term archival storage.

---

## Amateur Radio Integration

The project included amateur radio functionality.

The existing radio equipment included a Baofeng UV-5R, and the user held an amateur radio Technician license.

The mobile computer could potentially support:

- Receiver monitoring.
- Audio recording.
- Frequency reference information.
- Digital mode applications.
- Radio programming.
- CAT control for compatible transceivers.
- GPS-assisted logging.

However, the UV-5R does not provide the same standardized CAT control capabilities as many modern mobile or base-station transceivers.

The project emphasized evaluating each radio's actual interface capabilities before selecting control software.

---

## Radio Transmission Considerations

The computer was intended to support both receiving and potentially interfacing with transmitting equipment.

Transmission requires more than an SDR application.

A complete transmitting system may require:

- A compatible transceiver.
- An authorized frequency allocation.
- Proper RF output filtering.
- Suitable antenna hardware.
- Appropriate transmit power.
- A push-to-talk interface.
- Electrical isolation.
- Compliance with applicable regulations.

The computer could serve as a control or audio-processing interface, but it would not replace the RF transmitter unless paired with suitable transmitting hardware.

The project did not establish that any computer-controlled transmission interface was implemented.

---

## CB Radio Integration

The project also considered integrating a CB radio.

Potential functions included:

- Audio monitoring.
- Audio recording.
- External speaker integration.
- Microphone or push-to-talk control.
- Radio status monitoring where supported.

Most conventional CB radios do not provide a standardized computer-control interface.

Consequently, integration may require an external audio interface or hardware-specific connections.

The project emphasized preserving proper microphone, speaker, and PTT electrical characteristics.

Directly connecting unknown radio signals to computer audio ports could damage equipment.

---

## Audio Interface Considerations

A USB audio interface could provide isolation between the computer and external radio equipment.

Possible functions include:

- Receiving radio audio.
- Recording communications.
- Providing audio to a compatible transmitter.
- Supporting digital radio modes.

Additional isolation hardware may be required to prevent ground loops and unwanted electrical coupling.

The project emphasized that audio interfaces and transmitter keying circuits should be treated as separate subsystems.

---

## Satellite Reception

Satellite listening was another proposed application.

Potential receive-only activities included:

- Weather satellite reception.
- Amateur radio satellite monitoring.
- Satellite beacon reception.
- Digital satellite telemetry experiments.

These applications have different RF requirements.

Important considerations include:

- Operating frequency.
- Antenna polarization.
- Antenna gain.
- Receiver bandwidth.
- Low-noise amplification.
- Doppler shift.
- Satellite visibility.
- Vehicle movement.

A general-purpose SDR receiver could support some satellite reception tasks, but an appropriate antenna system would be essential.

---

## Satellite Reception While Mobile

Satellite reception from a moving vehicle introduces additional complexity.

A fixed antenna may not provide adequate coverage for every satellite pass.

Some satellite signals require directional antennas or specialized filtering.

The vehicle's roof and surrounding bodywork can also affect antenna performance.

The project emphasized that satellite reception capability should be evaluated separately from general terrestrial SDR reception.

No successful satellite reception test was documented.

---

## GPS Integration

Offline navigation was a major requirement.

The proposed system needed a dedicated GPS receiver capable of supplying position data without relying on cellular service.

Potential GPS interfaces included:

- USB GPS receivers.
- Serial GPS receivers.
- Internal expansion modules.
- GNSS receivers with standard NMEA output.

The exact GPS hardware was not selected in the available project history.

The project emphasized using a receiver with reliable Linux driver support.

---

## GPS Data Architecture

A typical Linux GPS configuration uses `gpsd`.

The service reads position data from a compatible receiver and makes it available to other applications.

```text
                    GNSS Satellites
                           |
                           v
                      GPS Receiver
                           |
                     USB / Serial
                           |
                           v
                         gpsd
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
         Navigation     GPS Logging    Other Apps
```

This architecture allows multiple applications to use the same GPS receiver.

The official gpsd documentation describes support for GPS devices connected through serial or USB interfaces.

---

## GPS Hardware Detection

After connecting a GPS receiver, the first step is to identify its device interface.

Useful commands include:

```bash
lsusb
```

```bash
ls /dev/ttyUSB*
```

```bash
ls /dev/ttyACM*
```

The actual device path depends on the receiver and Linux driver.

Some GPS devices use vendor-specific protocols rather than a conventional serial interface.

The project emphasized identifying the receiver before attempting to configure navigation software.

---

## GPSD Configuration

For a compatible receiver exposing a serial interface, an illustrative test command is:

```bash
sudo gpsd -N -n /dev/ttyUSB0
```

The device path must be replaced with the actual GPS interface.

A GPS client can then inspect the data:

```bash
cgps
```

The receiver may require time to obtain a satellite fix.

A running gpsd process does not guarantee that the receiver has valid position information.

The project emphasized validating actual latitude, longitude, time, and fix status.

---

## Offline Navigation

The vehicle computer was intended to provide navigation without cellular connectivity.

A suitable offline navigation system would need:

- Locally stored map data.
- GPS positioning.
- Searchable destinations.
- Route calculation.
- Turn-by-turn guidance where supported.
- Touchscreen compatibility.
- Periodic map updates.

The project considered OpenStreetMap-based navigation software as a potential foundation.

However, not every Linux map viewer provides full offline routing.

The project emphasized distinguishing map display from complete navigation functionality.

---

## Offline Mapping Architecture

```text
                   OpenStreetMap Data
                           |
                      Map Download
                           |
                           v
                     Local Storage
                           |
                           v
                  Navigation Software
                           |
                           v
                      Touchscreen
                           ^
                           |
                          GPS
```

Once maps are stored locally, the computer can display them without an active internet connection.

Whether routing also works offline depends on the selected application and its routing engine.

---

## Navigation Software Considerations

Potential software components included:

- OpenStreetMap-based map viewers.
- Offline routing engines.
- GPSD-compatible applications.
- GPX track recording tools.
- Custom touchscreen navigation frontends.

Selection criteria included:

- Linux compatibility.
- Offline map storage.
- Offline routing.
- GPS integration.
- Touchscreen usability.
- Hardware requirements.
- Maintenance status.

The project did not establish a final navigation application.

---

## Vehicle Power Integration

Power delivery was one of the most important hardware considerations.

A vehicle electrical system does not provide a perfectly regulated 12-volt supply.

The voltage can vary substantially depending on engine operation and electrical conditions.

Potential conditions include:

- Engine cranking voltage drops.
- Alternator charging voltage.
- Transient voltage spikes.
- Reverse polarity.
- Electrical noise.
- Battery discharge.
- Abrupt power removal.

The computer should not be connected directly to vehicle power unless its manufacturer explicitly supports the relevant automotive input conditions.

---

## Automotive DC-DC Power Supply

A regulated automotive DC-DC supply was the preferred architecture.

```text
                 Vehicle Battery
                        |
                     Fuse
                        |
                        v
                Automotive DC-DC
                  Power Supply
                        |
                  Regulated DC
                        |
                        v
                Linux Vehicle PC
```

The supply must match the computer's required voltage, current, connector, and polarity.

The project emphasized verifying the RT130's actual power specifications before purchasing a converter.

---

## Ignition-Aware Power Management

A vehicle computer should respond predictably to ignition state changes.

A conceptual sequence is:

1. Vehicle ignition turns on.
2. Power controller enables the computer.
3. Linux boots.
4. Navigation and radio applications start.
5. Vehicle ignition turns off.
6. Power controller signals Linux to shut down.
7. Linux completes a clean shutdown.
8. The controller removes computer power after a suitable delay.

This avoids repeated unclean shutdowns.

The project emphasized that ignition-aware power control requires suitable hardware and software coordination.

It was a proposed design rather than a verified installation.

---

## Example Shutdown Service

A Linux system can perform a controlled shutdown through systemd.

For example:

```bash
sudo systemctl poweroff
```

An external automotive power controller could request this action through a supported interface.

Possible interfaces include:

- GPIO.
- USB serial.
- Dedicated ignition controller.
- ACPI power button signaling.
- Network command.

The exact mechanism would depend on the selected hardware.

---

## Battery Protection

A permanently installed computer can discharge the vehicle battery if it remains powered while the engine is off.

The power system should therefore consider:

- Low-voltage cutoff.
- Ignition sensing.
- Delayed shutdown.
- Standby current.
- Battery capacity.
- Power converter efficiency.
- Safe recovery after power loss.

The project emphasized that a computer designed for unattended operation must not compromise the vehicle's ability to start.

---

## Thermal Management

Vehicle interiors can become significantly hotter than typical indoor computing environments.

A computer installed inside the dashboard or under a seat may have limited airflow.

Potential thermal issues include:

- CPU throttling.
- Storage overheating.
- Reduced component lifespan.
- Unexpected shutdown.
- Power supply overheating.

The project emphasized selecting a mounting location with adequate ventilation and verifying the computer's rated operating temperature.

The actual mounting location was not finalized.

---

## Vibration and Mounting

Vehicle installation also requires mechanical considerations.

The computer should be mounted securely to prevent movement during braking or collisions.

Potential mounting locations included:

- Under a seat.
- Inside a protected equipment compartment.
- Behind interior trim.
- In a dedicated equipment enclosure.

Mounting hardware should be appropriate for the computer's weight and the vehicle environment.

The project emphasized avoiding loose equipment in passenger areas.

---

## Audio Integration with the Pioneer Receiver

The vehicle used an aftermarket Pioneer multimedia receiver.

The Linux computer could potentially supply audio to the receiver through an available auxiliary input.

Possible connection methods included:

- Analog line-level audio.
- USB audio, if supported by the receiver.
- HDMI audio, if supported by the display and receiver architecture.
- An external DAC connected to an analog input.

The exact connection method depended on the Pioneer model's available inputs.

The project emphasized that HDMI video connectivity does not automatically establish a usable audio path into the car stereo.

---

## Factory JBL Audio Considerations

The Sequoia originally used a factory JBL amplified audio system.

The project history also included troubleshooting poor-quality audio from the factory amplifier and evaluating replacement equipment.

This affected the vehicle computer design because the final audio path depended on the condition and configuration of the amplifier system.

A conceptual arrangement was:

```text
                 Linux Computer
                       |
                  Audio Output
                       |
                       v
                Pioneer Receiver
                       |
                       v
                  Audio Interface
                       |
                       v
                   Amplifier
                       |
                       v
                    Speakers
```

The project emphasized that audio signal levels and amplifier input compatibility must be verified before making permanent connections.

---

## Ground Loop Considerations

Vehicle computers and audio equipment may be powered from different points in the electrical system.

This can create unwanted ground currents.

Symptoms may include:

- Alternator whine.
- Buzzing.
- Hum.
- Digital interference.
- Noise that changes with engine speed.

Potential solutions include:

- Proper grounding.
- Suitable power conversion.
- Shielded signal cables.
- Differential or isolated audio interfaces.
- Avoiding unnecessary ground paths.

The project emphasized diagnosing the source of interference rather than automatically adding isolation transformers to every connection.

---

## RF Antenna Installation

SDR performance depends heavily on antenna selection and placement.

The vehicle could potentially support multiple antennas for different frequency ranges.

Considerations included:

- Antenna mounting location.
- Ground plane requirements.
- Coaxial cable loss.
- Weatherproofing.
- Cable routing.
- RF interference.
- Separation from vehicle electronics.
- Mechanical durability.

A single antenna would not necessarily perform well across every desired frequency range.

The project emphasized designing the antenna system around actual reception requirements.

---

## RF Interference in Vehicles

Modern vehicles contain numerous sources of electrical noise.

Potential sources include:

- Alternators.
- Ignition systems.
- DC-DC converters.
- USB chargers.
- Display electronics.
- Computer power supplies.
- LED lighting.
- Other vehicle electronics.

These can interfere with sensitive radio receivers.

The project emphasized evaluating noise performance with the engine both running and stopped.

A receiver that performs well on a workbench may behave differently after installation inside a vehicle.

---

## Remote Homelab Connectivity

The project also considered connecting the vehicle computer to the home network.

Potential connectivity methods included:

- Wi-Fi when parked at home.
- Cellular networking.
- Smartphone tethering.
- A dedicated mobile router.
- Tailscale or another VPN.

This could allow the computer to access selected services hosted on Unraid.

Potential uses included:

- Uploading SDR recordings.
- Downloading updated map data.
- Synchronizing configuration files.
- Accessing documentation.
- Remote administration.
- Transferring logs.

The project emphasized that these functions should be optional rather than required for basic navigation and radio operation.

---

## Data Synchronization

The vehicle computer could use local storage while away from home.

When a trusted network connection became available, files could be synchronized with the Unraid server.

Possible tools included:

- Syncthing.
- rsync.
- SFTP.
- SMB.
- Scheduled scripts.

A conceptual workflow was:

```text
                Vehicle Computer
                       |
                 Local Storage
                       |
                Wi-Fi / VPN Link
                       |
                       v
                   Unraid NAS
                       |
                 Archive Storage
```

The project emphasized avoiding synchronization processes that would interfere with active SDR recording or navigation.

---

## Security Considerations

A permanently installed vehicle computer introduces several security requirements.

### Physical Security

The computer should not be easily accessible to unauthorized individuals.

### Full-Disk Encryption

Sensitive data should be protected if the computer or storage device is stolen.

### Network Security

Remote access should use encrypted connections.

### SSH Authentication

Key-based authentication should be preferred for remote administration.

### Wireless Connectivity

Untrusted public Wi-Fi networks should not be treated as trusted LANs.

### USB Devices

Unknown peripherals should not be connected without consideration of their security implications.

### Radio Transmission

Transmission interfaces should require deliberate operator action and comply with applicable regulations.

### Vehicle Safety

Computer operation should not interfere with essential vehicle electronics.

The project emphasized separating experimental computing functions from safety-critical vehicle systems.

---

## Troubleshooting Methodology

The project can be organized into several validation stages.

### Stage 1: Hardware Inspection

Identify the RT130's actual hardware specifications, connectors, power requirements, and storage configuration.

### Stage 2: Linux Installation

Install and validate a suitable Linux distribution.

### Stage 3: Display Testing

Confirm that the DisplayPort-to-HDMI connection works at the intended resolution.

### Stage 4: Touchscreen Testing

Verify USB touch input, coordinate mapping, and calibration.

### Stage 5: SDR Testing

Connect a compatible SDR receiver and confirm basic reception.

### Stage 6: GPS Testing

Verify that Linux detects the GPS receiver and obtains valid position data.

### Stage 7: Offline Navigation

Confirm that map display and routing work without an internet connection.

### Stage 8: Audio Integration

Test computer audio through the vehicle's amplifier system.

### Stage 9: Power Supply Validation

Verify stable operation during engine startup and shutdown.

### Stage 10: Thermal Testing

Monitor temperatures during representative workloads.

### Stage 11: RF Noise Testing

Compare reception with the engine running and stopped.

### Stage 12: Permanent Installation

Secure the computer, display, antennas, and cables.

### Stage 13: Network Integration

Configure optional remote access and synchronization.

### Stage 14: Reliability Testing

Validate startup, shutdown, suspend behavior, and recovery from power interruptions.

The available project history does not establish completion of these validation stages.

---

## Diagnostic Commands

The following Linux commands would be useful during commissioning.

| Command | Purpose |
|---|---|
| `lscpu` | Identify CPU |
| `free -h` | Inspect memory |
| `lsblk` | Inspect storage |
| `lspci` | Inspect PCI devices |
| `lsusb` | Inspect USB devices |
| `ip addr` | Inspect networking |
| `journalctl -b` | Review boot logs |
| `dmesg` | Inspect kernel messages |
| `ls /dev/ttyUSB*` | Find serial devices |
| `rtl_test` | Test compatible RTL-SDR hardware |
| `cgps` | Inspect GPS data |
| `sensors` | Monitor supported temperature sensors |

Some commands require additional packages.

The project emphasized collecting hardware information before modifying drivers or system configuration.

---

## Technical Challenges

### Legacy Industrial Hardware

The RT130 used an older processor platform, requiring careful operating system and software selection.

### Display Compatibility

The computer's video output needed to be adapted to the intended touchscreen.

### USB Touch Input

Touch functionality required a separate USB connection.

### Automotive Power

The vehicle electrical system required suitable regulation and shutdown handling.

### SDR Hardware Selection

Different radio applications required different receiver capabilities.

### GPS Integration

Offline navigation depended on a compatible GPS receiver and suitable mapping software.

### Radio Interface Compatibility

Amateur and CB radios varied in their computer-control capabilities.

### RF Interference

Vehicle electrical systems could affect radio reception.

### Audio Integration

The computer needed to interface with the existing Pioneer receiver and amplifier system.

### Thermal Management

The computer needed to tolerate the operating conditions of a vehicle interior.

### Remote Connectivity

Network access had to remain optional so the system could function offline.

These challenges demonstrated that vehicle computing requires integration across several engineering disciplines.

---

## Evaluation Outcome

The project established a proposed architecture for a multifunction Linux vehicle computer.

The Logic Supply RT130-i7-P was selected as the computing platform.

The intended installation was a 2002 Toyota Sequoia equipped with an aftermarket Pioneer multimedia receiver and factory JBL audio infrastructure.

The investigation identified several major subsystems:

- Linux operating system.
- Touchscreen display.
- DisplayPort-to-HDMI conversion.
- USB touch input.
- SDR receiver.
- GPS receiver.
- Offline navigation software.
- Radio interfaces.
- Automotive DC-DC power supply.
- Vehicle audio integration.
- Optional remote homelab connectivity.

The project also established the importance of ignition-aware shutdown, RF noise management, and offline operation.

The available project history does not confirm a completed installation or operational testing.

Accordingly, the documented outcome is the system architecture and integration evaluation rather than a finished mobile computing deployment.

---

## Lessons Learned

A general-purpose Linux computer provides more flexibility than a proprietary infotainment platform.

Industrial computers can be useful in vehicles, but their power and environmental specifications must be verified.

HDMI video and USB touchscreen input are separate interfaces.

Software-defined radio capability depends on the selected RF hardware as well as the computer.

A receive-only SDR cannot be used as a transmitter without suitable transmission hardware.

GPS receivers and navigation software perform different functions.

Offline maps do not automatically provide offline route calculation.

Automotive power systems require protection against voltage fluctuations and improper shutdown.

Vehicle electrical noise can significantly affect radio reception.

A modular architecture simplifies hardware upgrades and troubleshooting.

Existing stereo equipment can be integrated with a Linux computer through appropriate audio interfaces.

Remote network connectivity should enhance the system rather than become a requirement for basic operation.

The project strengthened understanding of embedded Linux, SDR, GPS, automotive electronics, radio communications, and systems integration.

---

## Future Improvements

Future work may include:

- Documenting the RT130's complete hardware specifications.
- Selecting and installing a Linux distribution.
- Testing the DisplayPort-to-HDMI adapter.
- Selecting a USB touchscreen.
- Implementing a touch-friendly launcher.
- Selecting an SDR receiver.
- Testing FM, VHF, and UHF reception.
- Evaluating satellite reception.
- Installing a dedicated GPS receiver.
- Configuring gpsd.
- Selecting an offline navigation platform.
- Implementing GPX track recording.
- Integrating a compatible amateur radio transceiver.
- Investigating CB radio audio interfacing.
- Designing the antenna installation.
- Selecting an automotive DC-DC supply.
- Implementing ignition-aware shutdown.
- Adding low-voltage battery protection.
- Testing audio integration with the Pioneer receiver.
- Measuring RF interference.
- Configuring Tailscale or another remote-access solution.
- Synchronizing SDR recordings with Unraid.
- Creating a dedicated mobile computing dashboard.
- Documenting the completed installation and validation results.

These improvements would move the project from system design toward a functional Linux-based mobile workstation.

---

## Summary

The Linux Vehicle Computer and SDR Integration project investigated creating a multifunction mobile computing platform inside a 2002 Toyota Sequoia.

The project used a Logic Supply RT130-i7-P industrial computer as the proposed foundation.

The system was intended to provide SDR reception, radio monitoring, satellite listening, amateur radio integration, CB radio interfacing, GPS navigation, and general-purpose Linux computing.

The investigation considered DisplayPort-to-HDMI conversion, USB touchscreen input, automotive power regulation, offline mapping, radio antennas, and integration with the existing Pioneer multimedia receiver.

A central design principle was maintaining offline functionality while allowing optional connectivity to the home Unraid server.

The project established a modular architecture and identified the technical challenges associated with integrating computing and RF equipment into a vehicle.

Although a completed installation was not verified, the work demonstrates practical understanding of Linux systems, SDR technology, GPS, radio communications, automotive electronics, and embedded system design.

---

## Technical References

- [GNU Radio — Official Website](https://www.gnuradio.org/)
- [GNU Radio — Documentation](https://wiki.gnuradio.org/)
- [SDR++ — Official Website](https://www.sdrpp.org/)
- [SDR++ — User Guide](https://www.sdrpp.org/manual.pdf)
- [GQRX — Official Website](https://gqrx.dk/)
- [SoapySDR — GitHub Repository](https://github.com/pothosware/SoapySDR)
- [RTL-SDR — Quick Start Guide](https://www.rtl-sdr.com/rtl-sdr-quick-start-guide/)
- [gpsd — Official Documentation](https://gpsd.io/)
- [gpsd — Installation and Testing](https://gpsd.io/installation.html)
- [OpenStreetMap — Official Website](https://www.openstreetmap.org/)
- [Linux Kernel — Official Documentation](https://docs.kernel.org/)
- [NetworkManager — Official Documentation](https://networkmanager.dev/docs/)
- [systemd — Official Documentation](https://systemd.io/)
- [Tailscale — Official Documentation](https://tailscale.com/kb/)
- [ARRL — Amateur Radio Resources](https://www.arrl.org/)
- [FCC — Amateur Radio Service](https://www.fcc.gov/wireless/bureau-divisions/mobility-division/amateur-radio-service)
- [FCC — Citizens Band Radio Service](https://www.fcc.gov/wireless/bureau-divisions/mobility-division/citizens-band-radio-service-cbrs)
- [Unraid — Official Documentation](https://docs.unraid.net/)
