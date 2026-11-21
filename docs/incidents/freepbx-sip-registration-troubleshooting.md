
# FreePBX SIP Registration Troubleshooting

## Overview

The FreePBX SIP Registration Troubleshooting project focused on diagnosing and resolving Voice over IP (VoIP) connectivity problems within a self-hosted private branch exchange (PBX) environment.

The project was part of a broader homelab initiative to operate a private telephone system using open-source telecommunications software, SIP trunks, and traditional analog telephone equipment.

The infrastructure used FreePBX 17, the Asterisk telephony engine, and VoIP.ms as the external SIP trunk provider.

The environment also included analog telephone adapters (ATAs), including Grandstream HT802 and Cisco ATA 192 hardware, allowing conventional analog telephones to operate through the IP-based telephone system.

The investigation involved SIP registration, authentication, network address translation (NAT), firewall configuration, SIP signaling, and troubleshooting inbound and outbound telephone calls.

A central challenge was distinguishing registration failures from call-routing problems.

A SIP trunk may successfully register with a provider while incoming calls fail because of incorrect routing, firewall restrictions, or signaling configuration.

Similarly, an analog telephone adapter may successfully register as a local extension while the PBX remains unable to communicate with its external trunk provider.

The troubleshooting process required understanding the relationships between SIP endpoints, Asterisk, FreePBX, the firewall, and the external telecommunications provider.

During the broader FreePBX deployment, inbound and outbound calling experienced intermittent failures. Temporarily disabling a firewall component was associated with restored functionality, indicating that firewall behavior was an important part of the investigation.

The exact firewall component, blocked traffic, and permanent corrective configuration are not preserved in the available project history.

This document describes the confirmed infrastructure and troubleshooting experience while distinguishing general diagnostic procedures from unverified deployment outcomes.

---

## Objectives

The primary objective was to investigate SIP registration and call connectivity problems affecting the FreePBX deployment.

Specific goals included:

- Deploying and administering FreePBX 17.
- Understanding Asterisk SIP architecture.
- Configuring a VoIP.ms SIP trunk.
- Diagnosing failed SIP registrations.
- Investigating SIP authentication.
- Understanding SIP endpoint registration.
- Configuring analog telephone adapters.
- Troubleshooting inbound calls.
- Troubleshooting outbound calls.
- Investigating NAT traversal.
- Understanding firewall behavior.
- Diagnosing SIP signaling failures.
- Identifying RTP audio problems.
- Using Asterisk command-line diagnostics.
- Inspecting SIP registration status.
- Evaluating firewall and port requirements.
- Improving PBX reliability.
- Developing repeatable VoIP troubleshooting procedures.

An additional objective was to integrate traditional analog telephones with a modern, self-hosted SIP infrastructure.

---

## Infrastructure Environment

The project was associated with the homelab's self-hosted telecommunications infrastructure.

The PBX environment used FreePBX and Asterisk to manage local extensions and external telephone connectivity.

| Component | Role |
|---|---|
| FreePBX 17 | PBX administration and configuration |
| Asterisk | SIP signaling and call processing |
| PJSIP | SIP endpoint and trunk management |
| VoIP.ms | External SIP trunk provider |
| Grandstream HT802 v2 | Analog telephone adapter |
| Grandstream HT702 | Additional ATA hardware |
| Cisco ATA 192 MPP | Analog telephone adapter |
| OPNsense | Network firewall and routing |
| NAT | Translation between private and public addresses |
| RTP | Voice media transport |
| Linux | PBX operating environment |
| Ethernet | Network connectivity |
| Analog telephones | Traditional telephone endpoints |

The broader homelab also included a FusionPBX deployment based on FreeSWITCH.

That earlier work provided additional experience with SIP trunk configuration, inbound routing, and NAT behavior.

The FreePBX project used a different PBX software stack and required its own configuration and troubleshooting procedures.

---

## System Architecture

The telephone system connected local analog endpoints to an external SIP provider through the FreePBX server.

```text
                    Public Telephone Network
                              |
                              v
                       +-------------+
                       | VoIP.ms     |
                       | SIP Trunk   |
                       +-------------+
                              |
                       SIP / RTP
                              |
                              v
                         Internet
                              |
                              v
                       +-------------+
                       | OPNsense    |
                       | Firewall    |
                       +-------------+
                              |
                         Local LAN
                              |
                              v
                       +-------------+
                       | FreePBX 17  |
                       | Asterisk    |
                       +-------------+
                              |
                     SIP Extensions
                              |
                +-------------+-------------+
                |                           |
                v                           v
        +---------------+           +---------------+
        | Grandstream   |           | Cisco ATA 192 |
        | HT802 / HT702 |           |               |
        +---------------+           +---------------+
                |                           |
                v                           v
         Analog Telephone            Analog Telephone
```

The PBX acts as the central call-processing system.

Local devices register as SIP extensions, while the external provider supplies connectivity to the public telephone network.

The project emphasized that successful communication requires correct configuration across several independent layers.

---

## Understanding SIP

Session Initiation Protocol (SIP) is a signaling protocol used to establish, modify, and terminate communication sessions.

In a VoIP system, SIP commonly manages:

- Endpoint registration.
- Call initiation.
- Call termination.
- Call forwarding.
- Session negotiation.
- Authentication.
- Call routing.

SIP does not ordinarily transport the voice audio itself.

Audio is commonly carried through Real-time Transport Protocol (RTP).

This distinction is important because signaling can function correctly while audio transmission fails.

---

## SIP Registration Architecture

SIP registration allows an endpoint to inform a registrar where it can be reached.

For example, a Grandstream ATA registers with the FreePBX server using extension credentials.

The PBX maintains information about the endpoint's current network contact.

A simplified registration exchange is:

```text
Grandstream ATA                  FreePBX / Asterisk
      |                                  |
      |---------- REGISTER ------------>|
      |                                  |
      |<------- 401 Unauthorized --------|
      |                                  |
      |----- REGISTER + Credentials ---->|
      |                                  |
      |<----------- 200 OK --------------|
      |                                  |
      |       Registration Complete      |
```

The initial authentication challenge is normal for many SIP configurations.

A `401 Unauthorized` response does not necessarily indicate a failed registration.

The important question is whether the endpoint successfully responds to the challenge and receives a successful registration response.

---

## SIP Trunk Registration

The external SIP trunk connects FreePBX to VoIP.ms.

Depending on the provider configuration, a trunk may use registration-based authentication or another supported authentication method.

For a registration-based trunk, Asterisk periodically sends SIP REGISTER requests to the provider.

The provider validates the supplied credentials and maintains the registration state.

A simplified architecture is:

```text
FreePBX / Asterisk
        |
        | SIP REGISTER
        v
VoIP.ms SIP Server
        |
        | Authentication Challenge
        v
FreePBX / Asterisk
        |
        | Authenticated REGISTER
        v
VoIP.ms SIP Server
        |
        | 200 OK
        v
Registered SIP Trunk
```

The project emphasized that trunk registration and local extension registration are separate operations.

A successful local extension registration does not prove that the external trunk is functioning.

---

## FreePBX and Asterisk

FreePBX provides a web-based administration interface for Asterisk.

Asterisk performs the underlying call processing.

FreePBX manages configuration for:

- Extensions.
- SIP trunks.
- Inbound routes.
- Outbound routes.
- Dial patterns.
- Ring groups.
- Voicemail.
- Call forwarding.
- Call recording.
- System administration.

The project emphasized understanding that FreePBX is a management layer rather than a replacement for Asterisk.

Advanced troubleshooting often requires examining Asterisk directly.

---

## PJSIP Architecture

Modern FreePBX deployments commonly use Asterisk's PJSIP channel driver.

PJSIP separates several SIP concepts.

These include:

- Endpoints.
- Authentication objects.
- Address-of-record objects.
- Contacts.
- Registrations.
- Transports.
- Identification rules.

An endpoint represents a SIP device or logical connection.

An authentication object defines the credentials used to authenticate a connection.

An address-of-record object tracks contacts associated with an endpoint.

The project emphasized that these components are related but distinct.

A trunk registration problem may involve a different configuration object from an endpoint identification problem.

---

## VoIP.ms Trunk Configuration

The FreePBX deployment used VoIP.ms for external telephone connectivity.

Important trunk configuration parameters include:

- Provider server hostname.
- SIP username.
- SIP password.
- Authentication method.
- Registration settings.
- SIP transport.
- Caller ID configuration.
- Codec preferences.
- Endpoint identification.
- Inbound routing.
- Outbound routing.

The exact values depend on the VoIP.ms account and selected provider server.

The project emphasized keeping provider credentials separate from publicly shared configuration examples.

No actual SIP usernames, passwords, or account identifiers are included in this document.

---

## SIP Server Selection

VoIP.ms provides multiple SIP server locations.

The selected server affects where the PBX sends registration requests.

A configuration mismatch between the intended server and the actual trunk settings can cause registration or connectivity problems.

The project emphasized verifying that the configured server hostname corresponds to the intended provider endpoint.

It also considered the importance of DNS resolution.

If the PBX cannot resolve the provider hostname, registration cannot proceed normally.

---

## DNS Troubleshooting

SIP trunks frequently use DNS hostnames rather than fixed IP addresses.

The PBX must be able to resolve the configured provider server.

On a Linux host, DNS resolution can be tested using:

```bash
getent hosts PROVIDER_HOSTNAME
```

Alternatively:

```bash
dig PROVIDER_HOSTNAME
```

The actual provider hostname must be substituted.

The broader homelab used local DNS infrastructure, including Unbound.

The project emphasized distinguishing DNS failures from SIP authentication failures.

If the hostname cannot be resolved, changing SIP credentials will not correct the underlying problem.

---

## SIP Authentication Failures

SIP authentication problems can occur when the provider rejects the credentials supplied by the PBX.

Potential causes include:

- Incorrect SIP username.
- Incorrect SIP password.
- Wrong authentication realm.
- Incorrect registration configuration.
- Provider account restrictions.
- Incorrect server selection.
- Conflicting registration sessions.
- Unsupported authentication settings.

Representative SIP responses include:

| Response | General Meaning |
|---|---|
| `401 Unauthorized` | Authentication challenge |
| `403 Forbidden` | Request rejected |
| `404 Not Found` | Requested resource not found |
| `408 Request Timeout` | No timely response |
| `407 Proxy Authentication Required` | Proxy authentication challenge |
| `200 OK` | Successful request |

These responses must be interpreted in the context of the complete SIP exchange.

For example, an initial `401` followed by an authenticated request and `200 OK` is generally normal.

The project emphasized examining complete SIP transactions rather than interpreting individual responses in isolation.

---

## Inspecting PJSIP Registrations

Asterisk provides command-line tools for inspecting SIP status.

The Asterisk CLI can be opened using:

```bash
sudo asterisk -rvvv
```

Within the CLI, outbound PJSIP registrations can be inspected using:

```text
pjsip show registrations
```

This command provides information about registration objects and their status.

Possible states depend on the Asterisk version and configuration.

The project emphasized checking the actual registration status before investigating unrelated call-routing settings.

---

## Inspecting SIP Endpoints

Local SIP endpoints can be inspected using:

```text
pjsip show endpoints
```

A specific endpoint can be examined using:

```text
pjsip show endpoint EXTENSION_NUMBER
```

For example:

```text
pjsip show endpoint 101
```

The extension number is illustrative.

These commands help identify whether Asterisk recognizes the endpoint and how it is configured.

The project emphasized distinguishing an endpoint configuration from an active registration contact.

---

## Inspecting SIP Contacts

Registered contacts can be examined using:

```text
pjsip show contacts
```

This provides information about contacts associated with PJSIP address-of-record objects.

A contact may include an IP address, port, and availability information.

The project emphasized using contact information to verify where Asterisk believes an endpoint is reachable.

This is particularly useful when troubleshooting ATAs that register successfully but cannot receive calls.

---

## SIP Logging

Asterisk can display SIP signaling through its PJSIP logger.

Within the Asterisk CLI:

```text
pjsip set logger on
```

The logger displays SIP messages exchanged by the PBX.

It can help identify:

- REGISTER requests.
- Authentication challenges.
- INVITE requests.
- SIP responses.
- Contact headers.
- SDP negotiation.
- Registration failures.
- Call termination messages.

Logging can be disabled using:

```text
pjsip set logger off
```

SIP logs may expose credentials, authentication material, telephone numbers, and network addresses.

The project emphasized protecting diagnostic logs and avoiding unnecessary public disclosure.

---

## Understanding NAT

Network Address Translation allows devices using private IP addresses to communicate with external networks.

The FreePBX server operated behind the homelab firewall.

This introduced a distinction between the PBX's private address and the public address visible to the SIP provider.

A simplified architecture is:

```text
FreePBX
192.168.x.x
     |
     v
OPNsense Firewall
     |
     v
Public WAN Address
     |
     v
VoIP.ms
```

The private address is illustrative.

SIP signaling can contain network addresses inside protocol headers and session descriptions.

This creates additional considerations beyond ordinary IP packet translation.

---

## SIP NAT Traversal

SIP traffic may include addresses in:

- Contact headers.
- Via headers.
- Record-Route headers.
- Session Description Protocol (SDP).
- RTP connection information.

If the PBX advertises an address that the remote provider cannot reach, calls may fail or audio may be unavailable.

The project emphasized configuring Asterisk's external signaling and media addresses appropriately for the network environment.

The exact settings depend on the PBX deployment, firewall configuration, and NAT topology.

---

## FreePBX NAT Settings

FreePBX provides SIP settings for network address configuration.

Important concepts include:

- External address.
- Local networks.
- SIP transport.
- RTP configuration.
- NAT traversal behavior.

The external address identifies the public-facing address used for relevant SIP communication.

Local network definitions help Asterisk distinguish internal endpoints from external connections.

The project emphasized ensuring that the PBX's configured network information matches the actual firewall topology.

Incorrect local network definitions can affect SIP signaling and media negotiation.

---

## SIP ALG

Some routers and firewalls provide SIP Application Layer Gateway functionality.

A SIP ALG attempts to inspect or modify SIP signaling to assist with NAT traversal.

However, SIP ALG behavior can interfere with systems that already manage NAT correctly.

The broader homelab networking configuration included disabling SIP ALG during VoIP troubleshooting.

The project emphasized avoiding multiple systems independently rewriting SIP signaling.

The actual behavior depends on the firewall and network configuration.

---

## Firewall Troubleshooting

Firewall behavior was a significant part of the FreePBX investigation.

During the broader deployment, inbound and outbound calling experienced intermittent failures.

Temporarily disabling a firewall component was associated with restored functionality.

This provided evidence that firewall policy or packet filtering behavior was relevant.

However, the available project history does not establish which specific firewall rule, service, or filtering component caused the problem.

The project emphasized using temporary firewall changes as diagnostic evidence rather than treating permanently disabled filtering as an acceptable solution.

---

## SIP Signaling Ports

SIP commonly uses UDP or TCP port 5060 for unencrypted signaling.

TLS-based SIP commonly uses port 5061.

However, these are conventions rather than universal requirements.

The actual port depends on:

- PBX transport configuration.
- Provider requirements.
- Endpoint settings.
- Network topology.
- Custom port assignments.

The project emphasized verifying the configured ports rather than assuming that every SIP service uses the defaults.

---

## RTP Media Ports

RTP carries the audio stream after a call is established.

Asterisk commonly uses a configurable UDP port range for RTP media.

A conventional example is:

```text
10000-20000/UDP
```

The actual range must be verified in the Asterisk configuration.

The project emphasized that SIP registration can succeed even when RTP traffic is blocked.

This can produce calls that connect but have no audio.

---

## SIP Versus RTP Troubleshooting

SIP and RTP failures often produce different symptoms.

| Symptom | Possible Area |
|---|---|
| Trunk cannot register | SIP authentication, DNS, firewall, provider |
| Extension cannot register | Endpoint credentials, network, transport |
| Outbound call rejected | Outbound route, trunk, provider |
| Incoming call does not ring | Inbound route, SIP signaling, endpoint |
| Call connects with no audio | RTP, NAT, firewall |
| One-way audio | RTP routing, NAT, endpoint configuration |
| Call drops unexpectedly | Session timers, signaling, network interruption |

The project emphasized classifying the failure before changing configuration.

This helps prevent unrelated troubleshooting steps from introducing additional problems.

---

## Inbound Call Routing

An inbound call from VoIP.ms must reach the PBX and match an appropriate inbound route.

A simplified inbound call path is:

```text
Public Telephone Network
          |
          v
        VoIP.ms
          |
          v
       FreePBX
          |
          v
     Inbound Route
          |
          v
       Extension
          |
          v
     Analog Telephone
```

The project emphasized that successful SIP registration does not automatically configure inbound call routing.

The PBX must know how to handle incoming calls.

---

## Outbound Call Routing

Outbound calls require a matching outbound route.

The route determines which trunk should handle the call.

A simplified outbound call path is:

```text
Analog Telephone
       |
       v
Grandstream ATA
       |
       v
FreePBX Extension
       |
       v
Outbound Route
       |
       v
VoIP.ms Trunk
       |
       v
Public Telephone Network
```

Potential problems include:

- Incorrect dial patterns.
- Missing trunk assignment.
- Caller ID restrictions.
- Provider authentication.
- Number formatting.
- Trunk availability.

The project emphasized testing outbound routing separately from registration.

---

## Grandstream HT802 Integration

The Grandstream HT802 provides analog telephone ports that connect conventional telephones to SIP networks.

Each configured FXS port can register as a SIP endpoint.

Important settings include:

- SIP server address.
- SIP user ID.
- Authentication ID.
- SIP password.
- SIP transport.
- Registration interval.
- NAT behavior.
- Codec preferences.
- DTMF signaling.

The project emphasized matching ATA credentials with the corresponding FreePBX extension.

A mismatch can prevent registration even when the PBX and ATA are reachable over the network.

---

## Cisco ATA 192 Integration

The Cisco ATA 192 MPP was another component of the homelab telephone infrastructure.

Like the Grandstream devices, it provides a bridge between analog telephone equipment and SIP services.

The device requires compatible firmware and correct SIP account configuration.

The project emphasized that different ATA manufacturers expose similar SIP concepts through different configuration interfaces.

Troubleshooting should therefore focus on the underlying SIP exchange rather than relying exclusively on vendor-specific terminology.

---

## Analog Telephone Considerations

Traditional analog telephones introduce requirements that differ from those of modern IP phones.

These may include:

- Ring voltage.
- Ring cadence.
- Off-hook detection.
- Dial tone generation.
- DTMF signaling.
- Caller ID delivery.
- Analog line impedance.

The broader homelab included interest in operating vintage telephone equipment and answering machines.

The project emphasized that a registered ATA does not automatically guarantee compatibility with every analog telephone.

SIP registration, analog line signaling, and physical telephone behavior are separate layers.

---

## Firewall Rule Evaluation

OPNsense uses stateful firewall rules to control network traffic.

A PBX behind a firewall may require carefully designed rules for provider communication.

Important considerations include:

- Outbound SIP signaling.
- Inbound signaling responses.
- RTP media traffic.
- NAT behavior.
- Firewall state tracking.
- Provider source addresses.
- Local endpoint communication.

The project emphasized avoiding unnecessarily broad port exposure.

The required firewall policy should be based on the actual deployment architecture and provider requirements.

---

## Packet Capture Diagnostics

Packet captures can provide direct evidence of SIP and RTP communication.

On a Linux PBX, packet capture tools may include:

```bash
sudo tcpdump -ni any port 5060
```

This is an illustrative command for a deployment using SIP port 5060.

A broader capture may be needed when custom signaling ports or additional protocols are used.

Packet captures can help determine whether SIP requests reach the PBX and whether responses are transmitted.

The project emphasized correlating packet captures with Asterisk logs.

---

## sngrep Diagnostics

`sngrep` is a useful SIP signaling analysis utility.

It provides a terminal interface for examining SIP call flows.

On a supported Linux system:

```bash
sudo sngrep
```

The utility can display:

- REGISTER exchanges.
- INVITE transactions.
- Authentication challenges.
- SIP response codes.
- Call setup sequences.
- Call termination messages.

The project considered SIP flow visualization valuable for understanding failures that are difficult to diagnose from application status screens alone.

---

## Troubleshooting Registration Failures

A structured registration troubleshooting process begins with basic connectivity.

### Stage 1: Verify Network Connectivity

Confirm that the PBX has working network access.

### Stage 2: Verify DNS Resolution

Ensure that the provider hostname resolves correctly.

### Stage 3: Check Trunk Configuration

Review the SIP server, username, authentication settings, and transport.

### Stage 4: Inspect Registration Status

Use Asterisk CLI commands to determine whether the registration is active.

### Stage 5: Enable SIP Logging

Inspect REGISTER requests and provider responses.

### Stage 6: Review Firewall Behavior

Determine whether traffic is being blocked or modified.

### Stage 7: Check NAT Configuration

Verify the PBX's external address and local network definitions.

### Stage 8: Test Provider Connectivity

Determine whether the configured provider endpoint is reachable.

### Stage 9: Validate Call Routing

Test inbound and outbound calls independently.

### Stage 10: Verify RTP Audio

Confirm that established calls carry audio in both directions.

This methodology helps isolate failures without changing multiple unrelated settings simultaneously.

---

## Troubleshooting Intermittent Failures

Intermittent SIP failures can be more difficult to diagnose than persistent failures.

Potential causes include:

- Firewall state expiration.
- Registration timing.
- NAT mapping changes.
- Provider endpoint changes.
- Network interruptions.
- SIP transport issues.
- Conflicting registrations.
- Endpoint availability.
- Packet loss.

The project emphasized collecting logs during an actual failure.

A system that functions correctly after a restart may still have an unresolved underlying problem.

---

## Registration Expiration

SIP registrations have expiration intervals.

Endpoints must periodically refresh their registrations.

If refresh requests fail, the registrar may remove the endpoint's contact.

This can cause incoming calls to fail even when the device remains powered on.

The project emphasized monitoring registration status over time rather than checking it only once.

Registration stability is an important operational requirement for a telephone system expected to receive calls reliably.

---

## Multiple Registrations

A SIP account may behave differently depending on whether the provider supports multiple simultaneous registrations.

If multiple devices attempt to register using the same account, they may interfere with one another under certain provider configurations.

The project emphasized distinguishing the PBX's provider trunk credentials from the local extension credentials used by ATAs.

In a conventional PBX architecture, local telephones register to FreePBX rather than independently registering to the external provider.

---

## Codec Negotiation

SIP uses SDP to negotiate media parameters.

Supported audio codecs may include:

- G.711 μ-law.
- G.711 A-law.
- G.722.
- Other codecs supported by the endpoints and provider.

Codec compatibility affects whether the endpoints can establish a usable audio session.

The project emphasized checking negotiated codecs when calls connect but media behavior is abnormal.

However, codec configuration should not be treated as the first explanation for a failed SIP registration.

---

## DTMF Signaling

Dual-tone multi-frequency signaling is used for telephone keypad input.

VoIP systems can transmit DTMF through different mechanisms.

A common approach is RFC 4733 telephone-event signaling.

Incorrect DTMF configuration may affect:

- Automated menus.
- Voicemail access.
- Interactive voice response systems.
- Telephone banking systems.
- Extension feature codes.

The project emphasized treating DTMF problems separately from basic registration and call establishment.

---

## Call Detail Records

FreePBX provides call detail records that can help investigate call behavior.

These records may include:

- Calling number.
- Called number.
- Call start time.
- Call duration.
- Call disposition.
- Selected routing information.

The project emphasized correlating call records with SIP logs.

A call that never reaches the PBX may not appear in the same way as a call that reaches the PBX but fails during routing.

---

## Security Considerations

A self-hosted PBX is a security-sensitive service because it can initiate telephone calls through a paid provider.

The project emphasized:

- Strong SIP credentials.
- Restricted administrative access.
- Firewall segmentation.
- Avoiding unnecessary public SIP exposure.
- Protecting provider account credentials.
- Limiting extension registration access.
- Monitoring failed authentication attempts.
- Keeping FreePBX and Asterisk updated.
- Protecting SIP diagnostic logs.
- Maintaining secure backups.
- Reviewing outbound calling permissions.
- Monitoring unusual call activity.

The investigation reinforced that temporarily disabling a firewall can be useful for isolating a problem, but it is not an appropriate permanent operating configuration.

---

## Reliability and Recovery

A reliable PBX deployment should maintain stable registration and calling functionality across routine network events.

Important considerations include:

- Consistent IP addressing.
- Reliable DNS resolution.
- Stable SIP registration.
- Correct NAT configuration.
- Firewall state handling.
- Provider endpoint availability.
- Configuration backups.
- Service restart behavior.
- Monitoring and alerting.

The project emphasized that telephone infrastructure requires continuous availability rather than merely successful initial configuration.

---

## Monitoring and Diagnostics

Useful operational indicators include:

- SIP trunk registration status.
- Local extension availability.
- Failed authentication attempts.
- SIP response codes.
- Call setup failures.
- RTP packet loss.
- Firewall blocks.
- Network latency.
- Provider connectivity.
- Asterisk service status.
- Call detail records.

The project emphasized combining multiple sources of evidence.

A registration status indicator alone cannot explain every call failure.

---

## Relationship to FusionPBX

The broader homelab previously included FusionPBX and FreeSWITCH troubleshooting.

That deployment encountered inbound call routing and NAT-related problems.

The experience provided useful background for the later FreePBX investigation.

However, the two systems use different configuration models.

FusionPBX manages FreeSWITCH, while FreePBX manages Asterisk.

The project emphasized transferring protocol knowledge between platforms without assuming that their administrative procedures are identical.

---

## Evaluation Outcome

The FreePBX deployment successfully established VoIP.ms trunk connectivity during the broader project.

However, inbound and outbound calling experienced intermittent problems.

Temporarily disabling a firewall component was associated with restored call functionality.

This suggested that packet filtering or firewall configuration contributed to the observed failures.

The exact blocked traffic and permanent corrective rule changes are not preserved in the available project history.

The investigation reinforced the importance of distinguishing SIP registration, call routing, NAT traversal, and RTP media transport.

It also demonstrated the value of Asterisk diagnostics and firewall testing when troubleshooting self-hosted telecommunications systems.

---

## Lessons Learned

SIP registration and successful call routing are separate requirements.

A local extension can register while the external SIP trunk remains unavailable.

An external SIP trunk can register while incoming calls still fail.

An initial SIP authentication challenge is not necessarily an error.

NAT configuration affects both SIP signaling and RTP media.

Firewall behavior can cause intermittent VoIP failures.

SIP ALG may interfere with applications that already manage NAT traversal.

A call that connects without audio may indicate an RTP problem rather than an authentication failure.

Analog telephone adapters require correct local extension credentials.

Provider trunk credentials should not be confused with endpoint credentials.

The project strengthened understanding of Asterisk, FreePBX, SIP, PJSIP, NAT, firewall administration, RTP, and analog telephone integration.

---

## Future Improvements

Future work may include:

- Documenting the complete VoIP.ms trunk configuration.
- Establishing a standardized PJSIP troubleshooting procedure.
- Monitoring trunk registration status.
- Implementing alerts for registration failures.
- Recording firewall rules required for VoIP connectivity.
- Testing registration behavior during WAN interruptions.
- Monitoring SIP authentication failures.
- Capturing SIP traces during intermittent call failures.
- Evaluating provider failover options.
- Testing RTP performance under network load.
- Documenting ATA configuration templates.
- Maintaining encrypted PBX configuration backups.
- Implementing automated test calls.
- Reviewing outbound calling restrictions.
- Establishing a repeatable PBX recovery procedure.

These improvements would support more reliable and maintainable telephone infrastructure.

---

## Summary

The FreePBX SIP Registration Troubleshooting project focused on diagnosing SIP connectivity and call-processing problems within a self-hosted telephone system.

The infrastructure used FreePBX 17, Asterisk, VoIP.ms, and analog telephone adapters to connect traditional telephone equipment to an IP-based telecommunications network.

The work examined SIP registration, authentication, PJSIP endpoints, NAT traversal, firewall configuration, inbound routing, outbound routing, and RTP audio transport.

A central finding was that reliable VoIP connectivity depends on multiple independent layers.

Successful registration does not guarantee successful call routing, and successful call setup does not guarantee working audio.

The troubleshooting process also identified firewall behavior as an important factor in the broader deployment's intermittent calling problems.

Although the exact permanent firewall correction was not established in the available project record, the work provided practical experience with SIP diagnostics and network-level VoIP troubleshooting.

The project demonstrates knowledge of Linux systems administration, FreePBX, Asterisk, PJSIP, VoIP.ms, SIP trunking, NAT, firewall administration, RTP, packet analysis, and analog telephony integration.

---

## Technical References

- [FreePBX — Official Website](https://www.freepbx.org/)
- [FreePBX — Documentation](https://sangomakb.atlassian.net/wiki/spaces/FP)
- [Asterisk — Official Documentation](https://docs.asterisk.org/)
- [Asterisk — PJSIP Configuration](https://docs.asterisk.org/Configuration/Channel-Drivers/SIP/Configuring-res_pjsip/)
- [Asterisk — PJSIP Troubleshooting](https://docs.asterisk.org/Deployment/Troubleshooting/SIP-Troubleshooting/)
- [VoIP.ms — Official Wiki](https://wiki.voip.ms/)
- [OPNsense — Official Documentation](https://docs.opnsense.org/)
- [Grandstream — HT802 Documentation](https://www.grandstream.com/products/gateways-and-atas/analog-telephone-adaptors/product/ht802)
- [sngrep — Official Repository](https://github.com/irontec/sngrep)
- [RFC 3261 — Session Initiation Protocol](https://www.rfc-editor.org/rfc/rfc3261)
- [RFC 3550 — Real-time Transport Protocol](https://www.rfc-editor.org/rfc/rfc3550)
- [RFC 4733 — RTP Payload for DTMF Digits](https://www.rfc-editor.org/rfc/rfc4733)
