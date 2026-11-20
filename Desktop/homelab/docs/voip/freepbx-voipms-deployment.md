
# FreePBX & VoIP.ms Deployment

## Overview

The FreePBX & VoIP.ms Deployment project focused on implementing a self-hosted Voice over IP (VoIP) telephone system using FreePBX 17, Asterisk and VoIP.ms.

The objective was to establish a centrally managed telecommunications platform capable of supporting traditional analog telephones, internal extensions, inbound calling and outbound connectivity to the public switched telephone network (PSTN).

The deployment involved configuring SIP trunking, telephone extensions, inbound and outbound call routing, analog telephone adapters and network firewall policies.

A significant portion of the project involved troubleshooting intermittent call failures and identifying the relationship between SIP signaling, Network Address Translation (NAT), firewall configuration and RTP media traffic.

The project provided practical experience administering a Linux-based PBX, integrating third-party telecommunications services and diagnosing communication failures across multiple network layers.

---

## Objectives

The primary objective was to deploy a functional private branch exchange using FreePBX and integrate it with an external SIP trunk provider.

The deployment focused on:

- Installing and configuring FreePBX 17.
- Establishing SIP trunk connectivity with VoIP.ms.
- Creating and managing internal telephone extensions.
- Integrating analog telephone adapters.
- Configuring inbound and outbound call routing.
- Troubleshooting SIP registration and authentication.
- Resolving firewall and NAT-related call failures.
- Evaluating telephone system security.
- Establishing a maintainable self-hosted telecommunications platform.

An additional objective was to understand the operational differences between managing individual SIP endpoints and administering a centralized PBX.

---

## Infrastructure Environment

The deployment operated within an existing self-hosted infrastructure environment.

The principal components included:

| Component | Role |
|---|---|
| FreePBX 17 | PBX administration and configuration |
| Asterisk | SIP signaling, call processing and media services |
| Debian Linux | Operating system foundation |
| VoIP.ms | External SIP trunking and PSTN connectivity |
| OPNsense | Network firewall, routing and NAT |
| Grandstream HT802 | Analog telephone adapter |
| Grandstream HT802 V2 | Additional analog endpoint hardware |
| Cisco ATA 192 | Alternative analog telephone adapter |
| Ethernet LAN | Internal connectivity between PBX and endpoints |
| Traditional telephones | Analog user endpoints |

FreePBX was evaluated as an alternative to an earlier FusionPBX deployment.

Both platforms provided opportunities to investigate SIP trunking and PBX administration, but FreePBX offered a different management interface and configuration workflow.

---

## System Architecture

The telephone infrastructure was designed around a centralized PBX that manages internal extensions and routes calls through VoIP.ms.

Analog telephone adapters convert conventional telephone signaling into SIP-compatible communications.

The PBX provides the intermediary between internal endpoints and the external telephone network.

```text
                  Analog Telephones
                         |
                   RJ11 Connections
                         |
                +-------------------+
                | Analog Telephone  |
                | Adapters          |
                | HT802 / ATA 192   |
                +-------------------+
                         |
                      Ethernet
                         |
                         v
                +-------------------+
                | Internal LAN      |
                | OPNsense Network  |
                +-------------------+
                         |
                         v
                +-------------------+
                | FreePBX 17        |
                | Asterisk / PJSIP  |
                +-------------------+
                         |
                    SIP / RTP
                         |
                         v
                +-------------------+
                | VoIP.ms           |
                | SIP Trunk         |
                +-------------------+
                         |
                         v
                 Public Telephone
                      Network
```

The architecture separates endpoint management, call routing and external carrier connectivity.

This separation allows extensions and telephone hardware to be managed independently of the upstream telecommunications provider.

---

## FreePBX 17 Deployment

FreePBX 17 was selected as the PBX management platform.

FreePBX provides a web-based interface for configuring Asterisk services, including extensions, trunks, routing rules and call handling features.

The deployment involved establishing the Linux operating environment and configuring the PBX services required for SIP communications.

FreePBX provides a centralized administration interface, reducing the need to configure routine PBX functionality directly through Asterisk configuration files.

The project developed practical experience managing a telecommunications application that depends on several interconnected services.

Unlike a standalone SIP telephone, the PBX must maintain connectivity with both internal endpoints and the external SIP provider.

---

## Asterisk & PJSIP Architecture

Asterisk provides the underlying telephony engine used by FreePBX.

It handles call signaling, endpoint registration, media negotiation and call routing.

PJSIP provides a SIP protocol stack used for endpoint and trunk communications.

The deployment required understanding the relationship between the FreePBX configuration interface and the underlying Asterisk services.

FreePBX generates and manages Asterisk configuration based on administrative settings.

Directly modifying generated configuration files can cause changes to be overwritten when FreePBX applies updated settings.

The project reinforced the importance of using the supported administrative configuration workflow whenever possible.

---

## VoIP.ms SIP Trunk Configuration

VoIP.ms was selected as the external telecommunications provider.

The service provides SIP-based connectivity to telephone numbers on the public telephone network.

A SIP trunk establishes the connection between the local PBX and the provider.

Configuration considerations included:

- SIP server selection.
- Trunk authentication credentials.
- Registration configuration.
- PJSIP transport settings.
- Caller ID presentation.
- Outbound dialing rules.
- Inbound number routing.
- NAT and external address handling.

The trunk was configured to support external calling through the FreePBX environment.

Successful trunk registration was an important initial milestone.

However, subsequent troubleshooting demonstrated that a registered trunk does not guarantee that all inbound and outbound call paths will function correctly.

---

## Extension Management

FreePBX provides centralized management of internal telephone extensions.

Each extension represents a logical telephone endpoint with its own authentication and call handling configuration.

Analog telephone adapters can register their FXS ports as SIP extensions.

The PBX then handles communication between registered endpoints.

This architecture allows traditional analog telephones to participate in a modern IP-based telephone system.

Extension management considerations included:

- Unique extension numbers.
- SIP authentication credentials.
- Endpoint registration status.
- Caller ID configuration.
- Voicemail settings.
- Internal dialing behavior.

The project demonstrated how centralized extension management simplifies administration compared with independently configuring every telephone to use an external provider.

---

## Analog Telephone Adapter Integration

Traditional analog telephone equipment was integrated through SIP-compatible analog telephone adapters.

The Grandstream HT802 and Cisco ATA 192 were among the devices evaluated within the telecommunications environment.

These devices provide Foreign Exchange Station (FXS) interfaces that supply the electrical characteristics expected by conventional analog telephones.

The adapters convert analog audio and signaling into SIP and RTP traffic.

Each adapter requires appropriate SIP credentials and network configuration to communicate with the PBX.

Troubleshooting analog endpoints requires distinguishing between physical telephone problems, ATA configuration errors and PBX registration failures.

This integration provided experience bridging legacy telecommunications hardware with modern IP-based services.

---

## Outbound Call Routing

Outbound routes determine how FreePBX handles telephone numbers dialed by internal extensions.

A route evaluates the dialed number and selects an appropriate trunk for external calling.

The deployment required configuring outbound calling through the VoIP.ms trunk.

Outbound route considerations included:

- Dial pattern matching.
- Trunk selection.
- Caller ID configuration.
- Number formatting.
- Provider dialing requirements.

Outbound calling was successfully established during the deployment.

This confirmed that the PBX could process calls from internal endpoints and route them through the external SIP provider.

It also provided a baseline for investigating subsequent inbound calling problems.

---

## Inbound Call Routing

Inbound routing determines how external calls are delivered to internal extensions or other PBX destinations.

When a call arrives from VoIP.ms, FreePBX must match the incoming call against the configured inbound routes.

The route then determines the destination.

Potential destinations include extensions, ring groups, interactive voice response systems and voicemail.

The deployment included troubleshooting inbound calls that did not consistently reach the intended destination.

This required distinguishing between provider-side delivery, trunk signaling and PBX routing.

A call reaching the PBX does not automatically mean that FreePBX has a valid destination configured for it.

The project reinforced the importance of testing inbound and outbound call paths independently.

---

## SIP Registration & Authentication

SIP registration establishes the relationship between a SIP endpoint or trunk and its configured registrar.

Authentication failures can prevent an endpoint from registering or accepting calls.

The troubleshooting process involved validating:

- SIP usernames.
- Authentication identifiers.
- Passwords.
- Registrar addresses.
- Transport settings.
- Network reachability.
- Endpoint registration state.

Successful registration indicates that the authentication exchange completed.

However, additional validation is necessary to confirm that call setup and media transport function correctly.

The project demonstrated the importance of treating registration, call signaling and audio transmission as separate diagnostic stages.

---

## NAT Traversal

Network Address Translation was one of the most important technical considerations.

The PBX operated behind an OPNsense firewall rather than being directly connected to the public internet.

This introduced complications because SIP messages can contain network addresses and ports used to establish media sessions.

If the PBX advertises an internal address to an external provider, the provider may be unable to establish the expected media path.

NAT configuration must account for the PBX's internal network, external address and the behavior of the firewall.

The project required investigating how SIP and RTP traffic traverse the network boundary.

It reinforced the importance of configuring NAT consistently across the PBX and firewall.

---

## Firewall Troubleshooting

A major troubleshooting milestone involved identifying firewall behavior as a contributing factor to intermittent telephone service failures.

The deployment experienced inconsistent inbound and outbound calling despite apparently valid trunk configuration.

Temporarily disabling the firewall during controlled testing allowed telephone service to function.

This result helped isolate the problem to network security policy or traffic handling rather than the SIP account configuration alone.

The test was useful diagnostically, but leaving the firewall disabled would not be an acceptable production configuration.

The appropriate remediation is to restore firewall protection and identify the specific rules, NAT behavior or connection-state handling responsible for the failure.

The investigation demonstrated the value of changing one variable at a time when diagnosing complex network services.

---

## SIP ALG Considerations

SIP Application Layer Gateway functionality was also considered during network troubleshooting.

Some routers and firewalls attempt to inspect or modify SIP packets to assist with NAT traversal.

However, this behavior can interfere with SIP implementations that already manage external addressing.

Potential symptoms include failed incoming calls, incorrect media addresses and intermittent audio problems.

The broader OPNsense telecommunications configuration included disabling SIP ALG behavior where applicable.

The project emphasized the importance of understanding how intermediate network devices handle application-layer protocols.

---

## RTP Media Transport

Real-time Transport Protocol carries the audio streams associated with VoIP calls.

SIP establishes and manages the call, while RTP transports the audio.

This distinction is important because SIP registration and call setup can succeed even when audio traffic is blocked.

Typical symptoms of RTP connectivity problems include:

- One-way audio.
- No audio in either direction.
- Calls connecting without usable speech.
- Audio interruptions associated with network behavior.

Troubleshooting requires examining the media addresses and ports negotiated during call setup.

Firewall policies must allow the required media flows while maintaining appropriate network restrictions.

The project provided practical experience understanding the separation between signaling and media transport.

---

## Call Flow Analysis

Understanding the complete call path was essential for troubleshooting.

For an outbound call, the expected sequence is:

1. An analog telephone goes off-hook.
2. The ATA detects the off-hook condition and accepts dialed digits.
3. The ATA submits a SIP call request to FreePBX.
4. Asterisk evaluates the outbound route.
5. The call is forwarded through the VoIP.ms trunk.
6. The provider routes the call toward the destination.
7. SIP negotiates the call session.
8. RTP carries the audio between the negotiated media endpoints.

An inbound call follows the reverse signaling direction, with FreePBX selecting the appropriate internal destination.

Breaking the process into individual stages made it easier to identify where failures occurred.

---

## Monitoring & Diagnostics

FreePBX and Asterisk provide several diagnostic capabilities for investigating telephone service issues.

Useful information includes endpoint registration status, trunk status, call logs and SIP signaling messages.

Asterisk's command-line interface provides direct access to runtime diagnostic information.

Common diagnostic commands include:

```bash
sudo asterisk -rvvv
```

Once connected to the Asterisk console:

```text
pjsip show endpoints
pjsip show registrations
pjsip show contacts
```

For detailed SIP signaling inspection:

```text
pjsip set logger on
```

These commands can help identify authentication failures, missing registrations and unexpected SIP responses.

Packet capture tools such as tcpdump and Wireshark provide additional visibility when diagnosing network-layer issues.

The project reinforced the importance of collecting evidence from both the PBX and firewall.

---

## Troubleshooting Methodology

The troubleshooting process followed a staged approach.

### Stage 1: Verify Host Services

Confirm that the FreePBX host is operational and that Asterisk is running.

### Stage 2: Validate Endpoint Registration

Confirm that internal extensions and analog telephone adapters are registered.

### Stage 3: Verify Trunk Connectivity

Inspect the VoIP.ms trunk configuration and registration state.

### Stage 4: Test Internal Calling

Place calls between internal extensions to validate local PBX functionality independently of the provider.

### Stage 5: Test Outbound Calling

Verify outbound route selection and provider connectivity.

### Stage 6: Test Inbound Calling

Confirm that external calls reach the PBX and match the appropriate inbound route.

### Stage 7: Inspect Firewall & NAT

Review network policies, NAT behavior and RTP connectivity.

### Stage 8: Validate End-to-End Audio

Confirm that calls establish successfully and support bidirectional audio.

This methodology reduced unnecessary configuration changes and improved fault isolation.

---

## Technical Challenges Encountered

### Intermittent Call Failures

The PBX experienced inconsistent call behavior despite successful trunk configuration.

This required investigating network connectivity and firewall handling.

### Inbound Routing Problems

Incoming calls required correct FreePBX route configuration before they could reach the intended destination.

### Firewall Interference

Temporarily disabling the firewall restored functionality during testing, indicating that network policy or traffic handling required further investigation.

### NAT Configuration

The PBX's location behind OPNsense required careful handling of internal and external network addressing.

### Analog Endpoint Integration

Traditional telephone equipment introduced additional dependencies involving ATA registration and analog line behavior.

### Multiple Diagnostic Layers

Failures could originate from the endpoint, PBX, firewall, provider or public telephone network.

The project demonstrated the importance of testing each layer independently.

---

## Security Considerations

A PBX connected to an external SIP provider requires strong security controls.

Unauthorized access can lead to fraudulent telephone calls and financial charges.

The deployment architecture requires protection of SIP credentials, administrative interfaces and external call routing.

Recommended controls include:

- Restricting PBX administration to trusted networks.
- Using unique SIP authentication credentials.
- Maintaining appropriate firewall policies.
- Limiting unnecessary external port exposure.
- Applying security updates.
- Reviewing call activity for unexpected behavior.
- Restricting international or premium-rate dialing when not required.
- Maintaining configuration backups.

The firewall troubleshooting process reinforced the need to distinguish temporary diagnostic changes from acceptable long-term security configurations.

---

## Reliability & Recovery

Telephone infrastructure depends on multiple services operating together.

The PBX requires stable host resources, network connectivity and access to the SIP provider.

Analog endpoints also require electrical power and a functioning local network.

UPS-backed infrastructure can improve availability during short power interruptions.

Configuration backups are important because rebuilding a PBX involves restoring trunks, extensions, inbound routes, outbound routes and other application settings.

A documented recovery procedure should include restoring the PBX configuration and validating both internal and external call paths.

The project emphasized that telecommunications recovery must be verified through actual call testing rather than service startup status alone.

---

## Lessons Learned

SIP registration does not guarantee complete telephone service functionality.

Inbound and outbound calling must be tested independently because they rely on different routing configurations.

SIP signaling and RTP media transport introduce separate networking requirements.

Firewall and NAT behavior can cause intermittent failures that resemble application configuration problems.

Controlled diagnostic testing can identify the affected infrastructure layer more effectively than repeatedly changing PBX settings.

Centralized PBX management provides greater flexibility than independently configuring each analog endpoint.

Traditional telephone equipment can remain useful when integrated through appropriate SIP-compatible adapters.

The project strengthened understanding of the relationship between telecommunications applications and the underlying network infrastructure.

---

## Future Improvements

Future work will focus on improving security, reliability and operational visibility.

Potential improvements include:

- Refining OPNsense firewall and NAT policies for SIP and RTP.
- Documenting known-working VoIP.ms trunk settings.
- Implementing registration monitoring.
- Improving PBX configuration backup procedures.
- Evaluating automatic call routing and failover.
- Expanding internal extension management.
- Improving call quality diagnostics.
- Investigating additional analog telephone integrations.
- Establishing periodic end-to-end call testing.
- Documenting recovery procedures for PBX and provider outages.

These improvements would support a more maintainable and reliable self-hosted telecommunications platform.

---

## Summary

The FreePBX & VoIP.ms Deployment project involved implementing a self-hosted telephone system using FreePBX 17, Asterisk and an external SIP trunk provider.

The project included configuring extensions, integrating analog telephone adapters, establishing outbound calling and troubleshooting inbound call routing.

A major technical challenge involved diagnosing intermittent call failures associated with firewall and network behavior.

Controlled testing helped isolate the problem and demonstrated the importance of understanding SIP signaling, RTP media transport and NAT traversal.

The project strengthened practical skills in Linux systems administration, PBX deployment, SIP trunk configuration, firewall troubleshooting, network diagnostics and legacy telecommunications integration.

The experience provides a foundation for continued development and administration of reliable, self-hosted VoIP infrastructure.
