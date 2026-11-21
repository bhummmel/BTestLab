
# FreePBX Analog Telephony Infrastructure

## Overview

The FreePBX Analog Telephony project focused on integrating traditional analog telephone equipment into a modern, self-hosted Voice over IP (VoIP) environment.

The project combined FreePBX 17, Asterisk, SIP-compatible analog telephone adapters and VoIP.ms trunking to establish a telephone system capable of supporting legacy telephones through IP-based infrastructure.

A primary motivation was preserving the functionality of conventional analog telephone equipment while gaining the flexibility and centralized administration offered by a modern private branch exchange (PBX).

The deployment involved SIP extension configuration, analog line signaling, telephone adapter provisioning, call routing and network troubleshooting.

Additional investigation focused on compatibility with older telephones, electromechanical ringers and traditional answering machines.

The project provided practical experience with Linux-based telecommunications infrastructure, SIP endpoint management, analog signaling and network administration.

---

## Objectives

The primary objective was to establish a centralized telephone system capable of connecting traditional analog telephones to modern SIP services.

The project focused on the following goals:

- Deploying FreePBX as a centralized telephone management platform.
- Integrating analog telephone adapters with Asterisk.
- Configuring SIP extensions for traditional telephone endpoints.
- Establishing internal and external calling functionality.
- Supporting incoming and outgoing calls through VoIP.ms.
- Investigating analog ringing voltage and current requirements.
- Evaluating compatibility with traditional answering machines.
- Troubleshooting SIP registration and call routing failures.
- Understanding analog line supervision and disconnect signaling.
- Maintaining administrative control over telephone infrastructure.

The project also explored the technical differences between traditional circuit-switched telephone systems and packet-based voice communications.

---

## Infrastructure Environment

The telecommunications environment operated within an existing Linux-based homelab infrastructure.

The principal components included:

| Component | Purpose |
|---|---|
| FreePBX 17 | Centralized PBX administration |
| Asterisk | SIP signaling and call processing |
| Debian Linux | PBX operating system |
| VoIP.ms | External SIP trunk and telephone service |
| OPNsense | Firewall, NAT and network routing |
| Grandstream HT802 | Two-port analog telephone adapter |
| Grandstream HT802 V2 | Additional analog telephone adapter |
| Grandstream HT702 | Alternative analog endpoint hardware |
| Cisco ATA 192 | SIP-compatible analog telephone adapter |
| Traditional analog telephones | User endpoints |
| Analog answering machine | Legacy telephone accessory |

The environment was designed to provide conventional telephone functionality while retaining the management capabilities of a modern IP-based PBX.

---

## System Architecture

The architecture centers on FreePBX, which manages internal extensions and external call routing.

Analog telephone adapters provide the electrical interface between conventional telephones and the IP network.

Each adapter converts analog signaling and audio into SIP and RTP communications.

The PBX handles call establishment, extension routing and communication with the external telephone provider.

```text
                 Traditional Telephones
                          |
                     RJ11 Cabling
                          |
            +-------------+-------------+
            |                           |
            v                           v
    +---------------+           +---------------+
    | Grandstream   |           | Cisco ATA 192 |
    | HT802 / HT702 |           |               |
    +---------------+           +---------------+
            |                           |
            +-------------+-------------+
                          |
                       Ethernet
                          |
                          v
                  +---------------+
                  | Internal LAN  |
                  |   OPNsense    |
                  +---------------+
                          |
                          v
                  +---------------+
                  |   FreePBX 17  |
                  |    Asterisk   |
                  +---------------+
                          |
                       SIP/RTP
                          |
                          v
                  +---------------+
                  |    VoIP.ms    |
                  |   SIP Trunk   |
                  +---------------+
                          |
                          v
                   Public Telephone
                        Network
```

The design separates the physical telephone interface from call processing and external network connectivity.

This separation allows analog devices to be replaced or reconfigured without redesigning the entire telephone system.

---

## FreePBX & Asterisk Integration

FreePBX provides the administrative interface for configuring and managing the telephone system.

Asterisk performs the underlying call processing, SIP signaling and media negotiation.

FreePBX simplifies common administrative tasks by providing graphical configuration interfaces for extensions, trunks, inbound routes and outbound routes.

The deployment involved configuring FreePBX 17 to operate with SIP-compatible endpoints and an external telecommunications provider.

Asterisk's PJSIP channel driver provides the SIP functionality used by modern endpoint and trunk configurations.

PJSIP was the appropriate choice for this environment because legacy `chan_sip` support is deprecated in modern Asterisk and FreePBX releases.

The project reinforced the importance of understanding both the PBX management interface and the underlying telecommunications engine.

---

## Analog Telephone Adapter Integration

Analog telephone adapters were used to connect conventional telephones to the IP-based PBX.

The Grandstream HT802, HT802 V2, HT702 and Cisco ATA 192 were among the devices investigated.

Each adapter provides one or more Foreign Exchange Station (FXS) ports.

An FXS port supplies the electrical telephone line interface expected by conventional analog equipment.

The adapter performs several important functions:

- Providing telephone line voltage.
- Detecting off-hook and on-hook conditions.
- Generating dial tone.
- Producing incoming-call ringing signals.
- Detecting dialed digits.
- Converting analog audio into digital voice traffic.
- Communicating with the PBX through SIP.

The project demonstrated how an ATA allows older telephone equipment to function as an endpoint within a modern VoIP network.

---

## FXS & FXO Signaling

Understanding the distinction between FXS and FXO interfaces was essential.

### Foreign Exchange Station

An FXS interface provides telephone service to an analog endpoint.

It supplies line voltage, ringing signals and other electrical characteristics required for normal telephone operation.

The analog ports on the Grandstream HT802 and Cisco ATA 192 are FXS interfaces.

### Foreign Exchange Office

An FXO interface receives telephone service from an FXS source.

Traditional analog telephones and answering machines connect to FXS ports using FXO-type interfaces.

The distinction matters because connecting two interfaces that both supply line voltage can create an electrical conflict.

The project reinforced the importance of identifying telecommunications interface types before connecting hardware.

---

## SIP Extension Provisioning

Each analog telephone adapter must be associated with an appropriate SIP extension.

FreePBX provides centralized extension management through its administrative interface.

The deployment architecture uses PJSIP extensions for modern SIP endpoint compatibility.

A typical provisioning workflow involves:

1. Creating a PJSIP extension in FreePBX.
2. Assigning an extension number and display name.
3. Generating a unique authentication password.
4. Applying the PBX configuration.
5. Configuring the corresponding SIP account on the ATA.
6. Verifying endpoint registration.
7. Testing internal calling.
8. Testing inbound and outbound calling.

The ATA requires the PBX address, extension credentials and compatible SIP transport settings.

For devices with multiple FXS ports, separate SIP accounts can be assigned to individual ports.

This allows multiple analog telephones to operate as independent PBX extensions.

---

## Internal Telephone Calling

Internal extension calling is one of the principal advantages of a centralized PBX.

Instead of requiring every telephone to connect independently to an external SIP provider, endpoints register with the local FreePBX server.

Asterisk can then route calls directly between internal extensions.

For example, an analog telephone connected to one ATA can call another telephone connected to a different ATA.

The call can remain within the local network without requiring external PSTN routing.

This arrangement provides a foundation for additional features such as voicemail, ring groups, call transfers and internal dialing plans.

The project demonstrated how traditional telephone equipment can participate in modern PBX functionality.

---

## VoIP.ms External Calling

VoIP.ms provides the external telecommunications connection.

The provider allows the PBX to place and receive calls through the public telephone network.

FreePBX uses a SIP trunk to communicate with the provider.

Outbound routes determine which dialed numbers should be sent through the trunk.

Inbound routes determine how calls arriving from the provider should be delivered to internal endpoints.

The broader FreePBX deployment successfully established outbound calling while requiring additional troubleshooting for intermittent inbound and outbound behavior.

This experience reinforced the distinction between SIP trunk registration, call routing and actual end-to-end call functionality.

---

## Analog Ringing Infrastructure

One of the more specialized aspects of the project involved evaluating ringing requirements for traditional telephone equipment.

Conventional analog telephones may use electromechanical bells that require substantially more ringing energy than modern electronic ringers.

The ATA must generate an alternating-current ringing signal capable of activating the connected telephone.

North American telephone systems commonly use ringing frequencies around 20 Hz.

Ringing voltage may approach 90 volts AC in traditional telephone systems, although the actual signal depends on the equipment.

The project involved investigating whether available ATAs could reliably operate older ringing mechanisms.

The evaluation demonstrated that compatibility depends on more than nominal ringing voltage.

Ringing current, frequency, waveform and connected load must also be considered.

---

## Ringer Equivalence Number

The Ringer Equivalence Number (REN) describes the ringing load presented by telephone equipment.

An ATA has a limited ability to supply ringing power.

Connecting multiple devices to a single FXS port increases the total ringing load.

This is particularly relevant when combining a mechanical-bell telephone with an answering machine or additional telephone extensions.

Potential symptoms of excessive ringing load include:

- Weak mechanical ringing.
- Intermittent bell operation.
- Failure to ring multiple devices.
- Incomplete ringing cadence.
- Ringing that stops unexpectedly.

The project highlighted the importance of comparing connected equipment requirements against the ATA manufacturer's supported ringing capacity.

Increasing ringing voltage alone does not necessarily resolve a ringing-current limitation.

---

## Legacy Answering Machine Integration

The project also examined compatibility with traditional analog answering machines.

These devices depend on telephone line signaling to detect incoming calls, answer the line and determine when a caller disconnects.

An answering machine may rely on several characteristics of the analog line:

- Ring detection.
- Caller ID signaling.
- Off-hook current.
- Call-progress tones.
- Loop current interruption.
- Disconnect supervision.

Modern ATAs can reproduce many traditional telephone behaviors, but configuration differences may affect older equipment.

The evaluation included investigating how to provide appropriate line signaling for legacy answering machines.

This work reinforced the importance of understanding the electrical and signaling requirements of the connected endpoint.

---

## Loop Current Disconnect

Loop current disconnect is a mechanism used by telephone equipment to recognize when a call has ended.

A brief interruption in telephone line current can indicate that the remote party has disconnected.

Some answering machines depend on this signal to terminate recording and release the telephone line.

If the ATA does not provide compatible disconnect supervision, the answering machine may continue recording after the caller hangs up.

The Grandstream HT802 provides configurable analog line supervision options that can be evaluated for compatibility with connected equipment.

The appropriate settings depend on the behavior and requirements of the specific telephone or answering machine.

The project demonstrated that analog telephone compatibility includes signaling behavior beyond ordinary call establishment.

---

## DTMF Signaling

Dual-Tone Multi-Frequency signaling allows touch-tone telephones to transmit dialed digits.

The ATA detects these tones and communicates the dialing information to the PBX.

Depending on the configuration, DTMF can be transmitted using RTP telephone events, SIP signaling or in-band audio.

Incorrect DTMF settings may cause problems with voicemail systems, automated attendants or other services that require keypad input.

Testing must therefore include more than basic call establishment.

The project emphasized verifying that dialing and interactive telephone functions operate correctly across the ATA and PBX.

---

## Caller ID Compatibility

Caller ID is another important feature when integrating traditional telephone equipment.

The PBX receives caller information through SIP signaling.

The ATA must then present that information to the analog telephone using a compatible signaling method.

Older telephones may support only particular regional caller ID standards.

In North America, Bellcore/Telcordia-style signaling is commonly used.

Caller ID problems can originate from the provider, PBX, ATA or telephone.

Troubleshooting requires determining whether caller information is missing from the incoming SIP call or simply not being displayed by the analog endpoint.

This distinction improves fault isolation.

---

## Network & Firewall Integration

The telephone infrastructure operates behind an OPNsense firewall.

The firewall manages network access between internal endpoints, the PBX and external SIP services.

SIP signaling and RTP media traffic have different networking requirements.

A telephone may register successfully while experiencing incoming call failures or one-way audio because of firewall or NAT behavior.

The broader FreePBX deployment encountered intermittent call failures associated with network filtering.

Temporarily disabling firewall protection during controlled troubleshooting allowed calls to function, helping isolate the problem to the network layer.

This was a diagnostic step rather than an acceptable permanent configuration.

The appropriate long-term approach is to maintain firewall protection while permitting the specific traffic required by the PBX.

The project reinforced the importance of understanding SIP, RTP and NAT when administering VoIP systems.

---

## Analog Line Troubleshooting

Analog telephone problems require a different diagnostic approach from purely IP-based endpoint failures.

A useful troubleshooting process begins with a known-working analog telephone connected directly to the ATA.

This eliminates additional building wiring and accessories from the initial test.

The next step is to verify that the ATA is operational and that its SIP account is registered.

Basic dial tone, outbound calling and inbound ringing can then be tested.

If the known-working telephone operates correctly, the original telephone or answering machine can be introduced.

This helps distinguish between ATA configuration problems and compatibility issues involving the connected equipment.

For electrical investigations, suitable telecommunications test equipment should be used.

Ringing circuits can produce hazardous voltages, so measurements should be performed only with appropriately rated instruments and safe procedures.

---

## SIP Troubleshooting

SIP-related failures were investigated separately from analog line problems.

The first diagnostic stage involves verifying endpoint registration.

If the ATA is not registered, the investigation should focus on authentication credentials, SIP server configuration and network connectivity.

If registration succeeds but calls fail, attention shifts to call routing, dial plans and SIP signaling.

Asterisk provides command-line diagnostic tools for inspecting endpoint status.

For example:

```bash
sudo asterisk -rvvv
```

Within the Asterisk console:

```text
pjsip show endpoints
pjsip show contacts
pjsip show registrations
```

Detailed SIP signaling can be inspected using:

```text
pjsip set logger on
```

These tools help distinguish authentication failures, unreachable endpoints and unexpected call signaling responses.

The project demonstrated the value of inspecting actual protocol behavior rather than relying exclusively on graphical status indicators.

---

## Troubleshooting Methodology

The overall troubleshooting approach was organized into distinct layers.

### Physical Layer

Verify the telephone, RJ11 cable, ATA power supply and FXS connection.

### Analog Signaling Layer

Check dial tone, ringing behavior, off-hook detection and disconnect supervision.

### Endpoint Layer

Confirm that the ATA has valid SIP configuration and can register with FreePBX.

### PBX Layer

Verify extension configuration, dial plans and call routing.

### Network Layer

Inspect firewall policies, NAT behavior and network connectivity.

### Provider Layer

Confirm external trunk availability and inbound or outbound call delivery.

### Media Layer

Verify RTP connectivity and bidirectional audio.

This layered approach reduces unnecessary configuration changes and makes troubleshooting more systematic.

---

## Technical Challenges Encountered

### Analog Hardware Compatibility

Traditional telephone equipment introduced electrical requirements that differed from modern electronic telephones.

### Mechanical Ringer Requirements

Older electromechanical bells required evaluation of ringing voltage, current and total load.

### Answering Machine Disconnect Behavior

Legacy answering machines introduced additional requirements for line supervision and call termination signaling.

### SIP Registration

Analog telephone adapters required appropriate authentication and endpoint configuration.

### PBX Routing

External calls depended on correct FreePBX inbound and outbound routing rules.

### Firewall & NAT Behavior

Intermittent calling problems required investigation of network filtering and SIP/RTP connectivity.

### Multiple Hardware Platforms

Comparing Grandstream and Cisco adapters required understanding differences in management interfaces, firmware behavior and analog line capabilities.

These challenges provided experience diagnosing systems that combine physical telecommunications hardware with software-defined call processing.

---

## Security Considerations

Telecommunications infrastructure requires strong access controls because unauthorized use can result in fraudulent calls and unexpected charges.

FreePBX administration should be restricted to trusted networks.

Analog telephone adapters should use unique administrative credentials.

SIP extensions should use strong authentication secrets.

External access should be limited to the services and ports required by the selected deployment architecture.

PBX call records and logs should be reviewed for unusual activity.

Outbound dialing restrictions can reduce exposure to unauthorized international or premium-rate calls.

The project reinforced the importance of treating VoIP infrastructure as a security-sensitive network service.

---

## Monitoring & Operational Visibility

Reliable telephone service requires monitoring at both the application and network layers.

Useful operational indicators include:

- PBX availability.
- SIP trunk registration.
- ATA endpoint registration.
- Extension availability.
- Call completion.
- RTP connectivity.
- Firewall activity.
- System resource utilization.

FreePBX and Asterisk provide logs and runtime status information for investigating service failures.

Additional monitoring could be introduced through Home Assistant or a centralized infrastructure monitoring platform.

Periodic test calls would provide more meaningful validation than registration status alone.

The project emphasized that an endpoint appearing online does not guarantee successful telephone operation.

---

## Reliability & Recovery

The telephone system depends on several components operating together.

The PBX requires a functioning host, network connectivity and access to the SIP provider.

Analog telephones depend on the ATA for line voltage and ringing.

Unlike a traditional telephone line powered from a telephone exchange, an ATA requires local electrical power.

UPS-backed network and telephone equipment can improve availability during brief power interruptions.

However, external calling still depends on internet connectivity and upstream provider availability.

Configuration backups are important for restoring FreePBX extensions, trunks and routing rules.

ATA configuration backups can also reduce recovery time following hardware replacement.

The project reinforced the importance of maintaining documented configurations and testing service recovery procedures.

---

## Lessons Learned

Traditional analog telephones can be integrated into modern SIP infrastructure through compatible analog telephone adapters.

FXS interfaces provide the electrical telephone service required by conventional endpoints.

Legacy equipment may have ringing and line supervision requirements that differ from modern telephones.

A successful SIP registration does not guarantee that all telephone functions operate correctly.

Inbound calling, outbound calling, audio transport and analog ringing should be validated independently.

Firewall and NAT behavior can introduce failures that resemble PBX or endpoint configuration problems.

Centralized PBX administration simplifies extension management and provides greater flexibility than direct registration of individual telephones with a SIP provider.

The project strengthened understanding of the relationship between analog telecommunications and modern IP networking.

---

## Future Improvements

Future development will focus on improving analog compatibility, monitoring and service reliability.

Potential improvements include:

- Evaluating additional analog telephone adapters.
- Testing mechanical ringer compatibility under controlled loads.
- Measuring analog line characteristics with suitable equipment.
- Refining answering machine disconnect supervision.
- Documenting known-working ATA configurations.
- Improving FreePBX configuration backups.
- Implementing endpoint registration alerts.
- Refining OPNsense firewall policies for SIP and RTP.
- Expanding internal extension functionality.
- Establishing periodic call testing.
- Evaluating additional traditional telephone equipment.

These improvements would help create a more maintainable telephone environment while preserving compatibility with legacy hardware.

---

## Summary

The FreePBX Analog Telephony project focused on integrating traditional analog telephone equipment into a self-hosted VoIP system using FreePBX 17, Asterisk and SIP-compatible analog telephone adapters.

The project involved configuring telephone extensions, evaluating analog line characteristics, investigating mechanical ringing requirements and troubleshooting PBX connectivity.

Additional work explored traditional answering machine compatibility, caller ID signaling, DTMF handling and disconnect supervision.

The experience demonstrated that reliable analog telephone integration requires coordination between electrical signaling, endpoint configuration, PBX routing and network infrastructure.

The project strengthened practical skills in Linux systems administration, telecommunications infrastructure, SIP configuration, network troubleshooting, analog signaling and legacy equipment integration.

The resulting architecture provides a foundation for maintaining traditional telephone functionality within a modern, centrally administered communications environment.
