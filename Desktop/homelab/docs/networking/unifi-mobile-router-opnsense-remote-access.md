# UniFi Mobile Router Ultra and OPNsense Remote Access Evaluation

## Overview

The UniFi Mobile Router Ultra and OPNsense Remote Access project investigated using a cellular-enabled UniFi router to provide secure access to a self-hosted home network while traveling.

The existing homelab used OPNsense as its primary firewall and router, with UniFi networking equipment and a self-hosted UniFi Network controller.

The objective was to determine whether a UniFi Mobile Router Ultra could provide internet connectivity to portable computers while allowing those computers to access services hosted on the home LAN.

The intended use case involved connecting a laptop or other device to the mobile router through Ethernet or Wi-Fi and accessing the home network as though the device were connected locally.

Relevant services included:

- Unraid network storage.
- Home Assistant.
- Jellyfin.
- Music Assistant.
- Docker management interfaces.
- Internal web applications.
- SSH-accessible Linux systems.
- Other self-hosted infrastructure.

The evaluation focused on the distinction between cellular connectivity, remote router management, and private network access.

A central question was whether owning UniFi networking equipment would automatically allow a UniFi Mobile Router Ultra to access devices on the home network.

The investigation established that **UniFi hardware ownership and controller adoption do not, by themselves, establish a routed connection between two networks**.

Because the home network used OPNsense rather than a UniFi gateway, remote connectivity would depend on a compatible VPN or routing arrangement.

The project compared WireGuard, IPsec, and Tailscale as potential methods for providing that connectivity.

It also examined the practical differences between the UniFi Mobile Router Ultra and other mobile or travel routers.

The available project history confirms the technical evaluation but does not establish that a UniFi Mobile Router Ultra was purchased, configured, or successfully connected to the home network.

---

## Project Objectives

The primary objective was to evaluate a portable network gateway capable of providing secure remote access to the existing homelab.

Specific objectives included:

- Evaluating the UniFi Mobile Router Ultra.
- Understanding LTE connectivity and mobile data requirements.
- Comparing cellular and conventional travel routers.
- Determining whether UniFi integration simplifies remote access.
- Investigating compatibility with OPNsense.
- Evaluating WireGuard site-to-site VPN connectivity.
- Comparing IPsec and WireGuard.
- Considering Tailscale as an alternative.
- Providing remote access to home LAN devices.
- Supporting devices without individual VPN clients.
- Understanding network routing requirements.
- Evaluating mobile carrier CGNAT restrictions.
- Avoiding unnecessary exposure of home services.
- Maintaining the existing OPNsense firewall.
- Determining whether the mobile router offers meaningful advantages over simpler hardware.

The broader objective was to extend the home network to a portable location without requiring every connected device to be configured individually.

---

## Existing Homelab Infrastructure

The home network was built around a self-hosted infrastructure environment.

| Component | Function |
|---|---|
| OPNsense | Primary firewall, router, and network gateway |
| Unraid | NAS and application hosting |
| UniFi Network | Network equipment management |
| UniFi controller | Self-hosted management application |
| Tailscale | Existing remote-access technology |
| WireGuard | Potential site-to-site VPN protocol |
| Ethernet | Wired LAN connectivity |
| Wi-Fi | Wireless client connectivity |
| Home Assistant | Smart home automation |
| Jellyfin | Media streaming |
| Docker | Containerized applications |
| Mobile Router Ultra | Proposed portable network gateway |

The network used OPNsense as the routing and security boundary.

UniFi equipment could operate within the network without requiring OPNsense to be replaced.

This distinction was fundamental to the evaluation.

---

## Existing Network Architecture

The home network used a conventional gateway architecture.

```text
                     Internet
                        |
                        v
                  Cable Modem
                        |
                        v
                    OPNsense
                        |
              +---------+---------+
              |                   |
              v                   v
          UniFi LAN           Unraid Server
              |                   |
         Local Clients       Docker Services
                                  |
                         +--------+--------+
                         |        |        |
                         v        v        v
                        HA     Jellyfin   Other
```

OPNsense handled routing, firewall policies, and connectivity between network segments.

UniFi devices provided networking functionality within that environment.

The proposed mobile router would operate at a remote location rather than as a replacement for the home gateway.

---

## Understanding the UniFi Mobile Router Ultra

The UniFi Mobile Router Ultra is a compact cellular-enabled network router.

The North American model, `UMR-Ultra-US`, supports LTE connectivity alongside wired and wireless networking.

Its published hardware specifications include:

| Feature | Specification |
|---|---|
| Cellular | LTE Cat 4 |
| SIM | Nano-SIM |
| Ethernet | Two 10/100 Mbps RJ45 ports |
| Wi-Fi | Wi-Fi 4 |
| VPN client | WireGuard and OpenVPN |
| Site-to-site VPN | WireGuard and IPsec |
| Power | USB-C or 9–30 V DC input |
| LTE theoretical maximum | 150 Mbps down / 50 Mbps up |
| External antennas | LTE and Wi-Fi antenna connections |
| Management | Local portal or optional Mobility Cloud |

The Ethernet ports are Fast Ethernet rather than Gigabit Ethernet.

The LTE data rates are theoretical modem capabilities, not guaranteed real-world speeds. <Cite ref="turn481270search0"/>

The device is designed for applications such as remote connectivity, mobile networking, industrial installations, and cellular backup.

---

## Why Cellular Connectivity Was Relevant

A conventional travel router generally requires an existing internet connection.

This might be provided by:

- Hotel Wi-Fi.
- Ethernet.
- Smartphone tethering.
- A USB cellular modem.
- Another mobile hotspot.

The UniFi Mobile Router Ultra includes an integrated LTE modem.

With a compatible cellular subscription, it can establish internet connectivity independently of a hotel or local Wi-Fi network.

This provides an important advantage when traveling or operating equipment in locations without reliable wired internet.

However, cellular connectivity and remote home network access are separate functions.

An LTE connection provides internet transport; a VPN provides the private network connection.

---

## Proposed Remote Access Architecture

The desired architecture was:

```text
                  HOME NETWORK

                     Internet
                        |
                        v
                     OPNsense
                        |
                     Home LAN
                        |
             +----------+----------+
             |          |          |
             v          v          v
           Unraid      Home       Other
                      Assistant   Devices


                  Encrypted VPN
                       Tunnel


                 REMOTE LOCATION

                  LTE Network
                        |
                        v
               UniFi Mobile Router
                      Ultra
                        |
                +-------+-------+
                |               |
                v               v
             Laptop         Other Device
                |               |
                +-------+-------+
                        |
                  Home LAN Access
```

The VPN tunnel would carry authorized traffic between the portable network and the home network.

This would allow connected devices to access selected home services using private IP addresses.

The arrangement would not automatically make the two networks identical at Layer 2.

Broadcast discovery, multicast, and certain device discovery protocols might require additional configuration.

---

## UniFi Controller Versus UniFi Gateway

One of the central questions was whether the mobile router could automatically connect to the home network because the user already operated UniFi equipment.

This required distinguishing the UniFi controller from the network gateway.

### UniFi Network Controller

The controller manages supported UniFi networking devices.

It provides configuration, monitoring, and device management.

### UniFi Gateway

A UniFi gateway performs routing, firewalling, and VPN functions.

The existing network used OPNsense for these functions.

Consequently, the UniFi controller did not replace OPNsense as the VPN endpoint.

Ubiquiti documents support for UniFi networks operating behind third-party gateways, with routing and VLAN configuration remaining the responsibility of the third-party gateway. <Cite ref="turn481270search3"/>

---

## UniFi Mobile Router Management

The UniFi Mobile Router series uses its own management model.

Ubiquiti documents local administration through the device's Local Portal and optional remote management through Mobility Cloud.

The mobile router does not require adoption into a conventional UniFi Network controller to operate.

This distinction is important because the user's existing self-hosted UniFi controller would not automatically provide centralized management of the mobile router.

Ubiquiti's documentation identifies Mobility Cloud as the platform for remote UMR management and centralized configuration. <Cite ref="turn481270search4"/>

The evaluation emphasized that management-plane connectivity is not equivalent to data-plane connectivity.

A device being remotely manageable does not mean its connected clients can reach the home LAN.

---

## Site-to-Site VPN Architecture

A site-to-site VPN connects two networks through an encrypted tunnel.

In this project, the endpoints would be:

- The UniFi Mobile Router Ultra.
- The OPNsense home firewall.

The remote router would establish a VPN connection over LTE or another available internet connection.

Traffic destined for the home LAN would be routed through the tunnel.

```text
          Remote LAN                 Home LAN
         192.168.50.0/24           192.168.10.0/24
                |                         |
                v                         v
          Mobile Router                OPNsense
                |                         |
                +-----------+-------------+
                            |
                       WireGuard VPN
```

The subnets are illustrative and do not represent verified addresses from the original network.

Both sites require non-overlapping subnets for straightforward routed connectivity.

---

## WireGuard Evaluation

WireGuard was one of the primary VPN technologies considered.

WireGuard is a modern VPN protocol designed around a relatively small implementation and modern cryptographic primitives.

Relevant capabilities include:

- Encrypted network tunnels.
- Public-key authentication.
- UDP transport.
- Site-to-site connectivity.
- Remote client access.
- Support for routed subnets.
- Compatibility with OPNsense.
- Compatibility with the UniFi Mobile Router Ultra.

Ubiquiti documents WireGuard support for the UMR series, including manual site-to-site connections to third-party routers. <Cite ref="turn481270search5"/>

This made WireGuard a practical candidate for connecting the mobile router to OPNsense.

---

## WireGuard Network Design

A possible design would use a dedicated tunnel network.

| Network | Example |
|---|---|
| Home LAN | `192.168.10.0/24` |
| Remote LAN | `192.168.50.0/24` |
| WireGuard tunnel | `10.77.0.0/30` |
| OPNsense tunnel address | `10.77.0.1` |
| Mobile router tunnel address | `10.77.0.2` |

These addresses are examples.

The actual implementation must use ranges that do not conflict with existing networks.

The home firewall would route traffic for the remote LAN through the WireGuard peer.

The mobile router would route traffic for the home LAN through the tunnel.

---

## WireGuard Configuration Principles

A site-to-site WireGuard configuration requires agreement between both endpoints.

Important settings include:

- Local private key.
- Remote public key.
- Tunnel IP addresses.
- Peer endpoint.
- UDP listening port.
- Allowed IP ranges.
- Persistent keepalive behavior.
- Firewall rules.
- Routing configuration.

An illustrative OPNsense-side peer configuration is:

```ini
[Peer]
PublicKey = REMOTE_ROUTER_PUBLIC_KEY
AllowedIPs = 10.77.0.2/32, 192.168.50.0/24
PersistentKeepalive = 25
```

The example is conceptual.

OPNsense normally manages WireGuard configuration through its interface rather than requiring this exact text to be entered directly.

The remote router must be configured with the corresponding home LAN routes.

---

## Routing Requirements

A VPN tunnel alone does not guarantee connectivity between the two LANs.

The routers must know where to send traffic.

For example:

```text
Destination: 192.168.10.0/24
Next hop: WireGuard tunnel toward OPNsense
```

On the home side:

```text
Destination: 192.168.50.0/24
Next hop: WireGuard tunnel toward mobile router
```

The firewall must also permit the intended traffic.

Without appropriate routes and firewall policies, a VPN may appear connected while LAN access fails.

The project emphasized that VPN establishment and routed application connectivity are separate validation steps.

---

## Firewall Considerations

OPNsense acts as the primary security boundary for the home network.

A site-to-site VPN should permit only the necessary traffic.

Potential permitted destinations include:

- Unraid management services.
- Home Assistant.
- Jellyfin.
- Internal DNS.
- Selected SSH hosts.
- Other authorized services.

A broad rule allowing the entire remote LAN unrestricted access to the home network may be unnecessary.

The project emphasized using destination-specific firewall rules when practical.

---

## Cellular Carrier CGNAT

Mobile carriers frequently use Carrier-Grade Network Address Translation.

CGNAT allows multiple subscribers to share public IPv4 addresses.

This can prevent unsolicited inbound connections from reaching a cellular router.

The UniFi Mobile Router Ultra may therefore be unable to act as a directly reachable VPN server on a typical cellular plan.

However, it can initiate outbound connections to a reachable VPN server.

Ubiquiti's documentation specifically identifies CGNAT as a limitation for public-IP remote access. <Cite ref="turn481270search4"/>

This makes an outbound VPN connection from the mobile router to the home network a more practical architecture.

---

## Home Network Reachability

For a conventional WireGuard site-to-site connection, the mobile router needs to reach the home VPN endpoint.

Potential arrangements include:

- A public IPv4 address on the home connection.
- Dynamic DNS pointing to the home WAN address.
- Appropriate UDP firewall rules.
- A VPN relay or overlay solution if direct inbound connectivity is unavailable.

The project emphasized verifying home-side reachability before configuring the remote router.

A functioning LTE connection does not solve an unreachable home VPN endpoint.

---

## Dynamic DNS

Residential internet connections may use dynamic public IP addresses.

Dynamic DNS associates a hostname with the current public IP address.

For example:

```text
vpn.example.net
```

The mobile router could use that hostname to locate the home VPN endpoint.

Dynamic DNS does not bypass CGNAT or create a public route where one does not exist.

It only helps locate an endpoint whose address changes.

The project emphasized separating name resolution from actual network reachability.

---

## IPsec Evaluation

The UniFi Mobile Router Ultra also supports IPsec site-to-site VPN connections.

IPsec is widely used for connecting routers and firewalls from different manufacturers.

Relevant configuration requirements include:

- IKE version.
- Authentication method.
- Encryption algorithms.
- Integrity algorithms.
- Key exchange settings.
- Local and remote networks.
- Tunnel lifetimes.
- NAT traversal.
- Firewall rules.

IPsec can provide reliable site-to-site connectivity.

However, its configuration is generally more complex than a basic WireGuard deployment.

The evaluation therefore considered WireGuard a potentially simpler choice for this use case.

---

## OPNsense Compatibility

OPNsense supports VPN technologies suitable for third-party router integration.

The use of OPNsense does not prevent a UniFi Mobile Router Ultra from connecting to the home network.

It does, however, mean that the VPN must be configured using interoperable protocols.

The proprietary convenience features associated with certain all-UniFi gateway deployments should not be assumed to work automatically with OPNsense.

Ubiquiti explicitly documents manual WireGuard site-to-site configuration for third-party routers. <Cite ref="turn481270search5"/>

This was one of the principal findings of the evaluation.

---

## Tailscale as an Alternative

The existing homelab also used Tailscale for remote connectivity.

Tailscale builds an encrypted overlay network using WireGuard-based technology.

It can simplify remote access by handling peer discovery and many NAT traversal scenarios.

Potential advantages include:

- No requirement for a static home IP.
- Support for devices behind NAT.
- Centralized device authorization.
- Cross-platform clients.
- Subnet routing.
- Remote access without directly exposing individual services.

Tailscale can operate on OPNsense through a community-supported plugin. <Cite ref="turn481270search1"/>

This made it relevant to the evaluation.

---

## Tailscale Subnet Routing

A Tailscale subnet router allows remote devices to access a network even when the destination devices do not run Tailscale.

For example:

```text
                  Remote Laptop
                        |
                    Tailscale
                        |
                        v
                  OPNsense
                 Subnet Router
                        |
                        v
                     Home LAN
                        |
                  Unraid / HA
```

This approach could provide remote access to the homelab without requiring a UniFi Mobile Router Ultra.

However, devices connected to a portable router would still need a way to participate in the Tailscale network.

If the router itself cannot run Tailscale, a separate client or gateway device may be required.

Tailscale's subnet router functionality is documented for accessing devices that do not have individual Tailscale clients installed. <Cite ref="turn481270search10"/>

---

## Travel Router Versus Mobile Router

The project compared the UniFi Mobile Router Ultra with conventional travel routers.

The key distinction was not necessarily VPN capability.

Many travel routers can establish WireGuard or OpenVPN connections.

The UniFi Mobile Router Ultra's primary additional feature was its integrated LTE modem.

| Feature | Conventional Travel Router | UniFi Mobile Router Ultra |
|---|---|---|
| Ethernet client connectivity | Common | Yes |
| Wi-Fi client connectivity | Common | Yes |
| WireGuard support | Model-dependent | Yes |
| OpenVPN support | Model-dependent | Yes |
| Integrated cellular modem | Model-dependent | Yes, LTE |
| Dedicated SIM slot | Model-dependent | Yes |
| Home LAN VPN access | Possible | Possible |
| Requires UniFi home gateway | No | No, for manual VPN |
| Cloud management | Model-dependent | Optional Mobility Cloud |
| Wired port speed | Model-dependent | 100 Mbps |

The comparison established that the UniFi Mobile Router Ultra was not inherently required for remote home LAN access.

Its value depended on whether integrated cellular connectivity and its management features were desirable.

---

## Comparison with the Original UniFi Mobile Router

The evaluation also considered the earlier UniFi Mobile Router.

The original UMR and UMR Ultra share several networking capabilities.

Both support:

- LTE connectivity.
- WireGuard VPN client functionality.
- OpenVPN.
- WireGuard site-to-site VPN.
- IPsec site-to-site VPN.
- Wi-Fi.
- Ethernet connectivity.

However, their hardware differs.

| Specification | UniFi Mobile Router | Mobile Router Ultra |
|---|---|---|
| Cellular | LTE Cat 4 | LTE Cat 4 |
| Ethernet | Two Gigabit ports | Two Fast Ethernet ports |
| Wi-Fi | Wi-Fi 4 | Wi-Fi 4 |
| WireGuard | Yes | Yes |
| OpenVPN | Yes | Yes |
| IPsec | Yes | Yes |
| Integrated display | 1.3-inch touchscreen | No touchscreen listed |
| PoE | Supported | Not listed |
| USB-C power | Supported | Supported |

The original UMR provides Gigabit Ethernet interfaces, while the Ultra uses 100 Mbps ports. <Cite refs={["turn481270search0","turn481270search2"]}/>

This hardware difference matters when evaluating local wired throughput.

---

## Understanding the Performance Limitations

A VPN connection's effective throughput depends on several factors.

These include:

- Cellular signal quality.
- Mobile carrier congestion.
- LTE radio capabilities.
- VPN encryption overhead.
- Router CPU performance.
- Home internet upload speed.
- Network latency.
- Ethernet port speed.
- Wi-Fi conditions.

The UMR Ultra's 100 Mbps Ethernet ports impose a practical limit on wired traffic passing through those interfaces.

Its LTE Cat 4 modem has higher theoretical download capability, but real-world speeds depend on carrier conditions.

The project emphasized evaluating end-to-end performance rather than relying exclusively on advertised LTE speeds.

---

## Accessing Unraid Remotely

One intended use was accessing Unraid-hosted services from a portable computer.

A successful VPN connection could provide access to:

- SMB shares.
- Docker web interfaces.
- SSH services.
- Jellyfin.
- Home Assistant.
- Music Assistant.
- Other local applications.

For example, a remote computer might access:

```text
http://HOME_UNRAID_IP
```

or:

```text
http://HOME_ASSISTANT_IP:8123
```

These addresses would use the actual private IPs of the home network.

The project emphasized that remote access should use encrypted VPN transport rather than exposing administrative services directly to the public internet.

---

## SMB Access Over VPN

SMB file sharing was particularly relevant because the homelab used Unraid as a central file server.

A routed VPN could allow a remote computer to access SMB shares using the server's private address.

For example:

```text
smb://HOME_UNRAID_IP/SHARE_NAME
```

However, network discovery may not work identically across routed subnets.

Directly specifying the server's IP address or a properly configured internal DNS name is generally more predictable.

The project emphasized distinguishing service connectivity from automatic network discovery.

---

## DNS Considerations

The home network used local DNS services.

A remote computer might need to resolve internal hostnames such as:

```text
mouseframe.local
```

However, `.local` names commonly depend on multicast DNS.

Multicast DNS generally does not cross routed VPN boundaries automatically.

A more reliable approach is to use:

- A conventional internal DNS domain.
- DNS records hosted on the home network.
- A VPN DNS configuration.
- Direct private IP addresses.

The project emphasized that successful VPN routing does not automatically provide local hostname discovery.

---

## Network Subnet Conflicts

A common problem with remote access is overlapping private IP ranges.

For example, if both the home network and the mobile router use:

```text
192.168.1.0/24
```

the remote computer may not know whether a destination belongs to the local or remote network.

This can prevent access to home services.

The project emphasized assigning the mobile router a distinct subnet.

A possible arrangement is:

```text
Home LAN:
192.168.10.0/24

Mobile LAN:
192.168.50.0/24
```

These are illustrative addresses.

The actual subnets should be selected based on the existing network configuration.

---

## Full-Tunnel Versus Split-Tunnel Routing

A remote VPN can be configured in different ways.

### Split Tunnel

Only traffic destined for selected home networks passes through the VPN.

General internet traffic uses the mobile router's local internet connection.

### Full Tunnel

All internet traffic passes through the VPN and exits through the home network.

For the original use case, split tunneling was a practical option because the primary objective was accessing home LAN services.

Full tunneling could be useful when the remote computer should use the home network's internet connection, but it would consume home upload bandwidth and increase latency.

---

## Mobile Carrier Data Usage

A cellular router introduces ongoing data usage considerations.

Accessing Home Assistant or administrative interfaces generally consumes relatively little bandwidth.

Streaming media or transferring large files can consume significantly more.

For example:

- Home Assistant dashboards typically require modest bandwidth.
- SSH sessions require little bandwidth.
- SMB file transfers can be data-intensive.
- Jellyfin streaming depends on media bitrate.
- Software downloads and updates may consume substantial data.

The project emphasized evaluating cellular data allowances before relying on the mobile router for routine media streaming or large file transfers.

---

## Remote Access Security

The proposed architecture involved connecting an external network to the home LAN.

Security considerations included:

- Strong VPN authentication.
- Restricted firewall rules.
- Unique cryptographic keys.
- Secure router administration.
- Device firmware updates.
- Network segmentation.
- Logging and monitoring.
- Avoiding unnecessary public port forwarding.
- Limiting access to sensitive management interfaces.
- Protecting mobile devices against unauthorized access.

The project emphasized treating the portable network as a separate security zone rather than automatically granting it unrestricted access to every home device.

---

## Example Access Policy

A conceptual access policy might allow:

| Destination | Access |
|---|---|
| Home Assistant | HTTPS or configured local port |
| Jellyfin | Application port |
| Unraid SMB | SMB service |
| Unraid SSH | Authorized administrative access |
| Internal DNS | DNS queries |
| OPNsense administration | Restricted to trusted devices |
| Other home devices | Denied unless required |

This approach limits the scope of remote connectivity.

The exact firewall rules would depend on the home network's addressing and security requirements.

---

## Validation Procedure

A practical implementation should be validated in stages.

### Stage 1: Establish Internet Connectivity

Confirm that the mobile router can access the internet through LTE or another configured WAN connection.

### Stage 2: Verify VPN Connectivity

Confirm that the mobile router establishes an encrypted tunnel with the home VPN endpoint.

### Stage 3: Test Tunnel Addresses

Verify connectivity between the VPN endpoint addresses.

### Stage 4: Test Home LAN Routing

Attempt to reach a permitted home LAN device.

### Stage 5: Test DNS

Verify that intended internal hostnames resolve correctly.

### Stage 6: Test Application Access

Open Home Assistant, Jellyfin, and other permitted services.

### Stage 7: Test SMB

Connect to an Unraid share using its private IP or internal DNS name.

### Stage 8: Test Reconnection

Disconnect and reconnect the cellular WAN.

### Stage 9: Test Device Roaming

Verify behavior when switching between LTE and another available internet connection.

### Stage 10: Evaluate Performance

Measure latency, throughput, and stability under representative workloads.

These stages distinguish VPN establishment from actual end-to-end application functionality.

---

## Troubleshooting Matrix

| Symptom | Likely Diagnostic Area |
|---|---|
| Mobile router has no internet | SIM, APN, carrier, LTE signal |
| VPN tunnel does not establish | Endpoint reachability, keys, firewall |
| VPN connects but home LAN is unreachable | Routing, Allowed IPs, firewall |
| Home LAN reachable by IP but not hostname | DNS or multicast discovery |
| Some services work but others fail | Firewall or application binding |
| SMB connection fails | Routing, SMB permissions, firewall |
| VPN disconnects during cellular changes | WAN instability, keepalive |
| Home and remote LAN addresses overlap | Subnet design |
| Slow media streaming | LTE bandwidth, home upload, latency |
| Remote management unavailable | Mobility Cloud or local management configuration |

The matrix provides a structured way to isolate network-layer and application-layer problems.

---

## Technical Challenges

### UniFi and OPNsense Interoperability

The home network used OPNsense rather than a UniFi gateway.

This required evaluating manual VPN configuration instead of assuming automatic UniFi site connectivity.

### Controller Versus Gateway Functionality

The UniFi controller managed networking equipment but did not serve as the home VPN endpoint.

### Cellular CGNAT

The mobile carrier could prevent inbound connections to the mobile router.

### VPN Routing

Both networks required correct routing and firewall rules.

### Local DNS

Home LAN names might not resolve automatically over a routed VPN.

### Hardware Performance

The Mobile Router Ultra's Fast Ethernet interfaces could limit wired throughput.

### Subscription Requirements

The router required a compatible cellular plan for independent LTE operation.

### Remote Management

Mobility Cloud and local router administration were separate from access to home LAN services.

### Alternative Technologies

Tailscale and conventional travel routers could potentially meet the same remote-access requirements with less specialized hardware.

These challenges reinforced the importance of evaluating network architecture before selecting hardware.

---

## Evaluation Outcome

The investigation established that a UniFi Mobile Router Ultra could potentially provide secure access to the existing OPNsense-based home network.

The router supported interoperable VPN technologies, including WireGuard and IPsec.

Because OPNsense served as the home gateway, the connection would require appropriate VPN configuration, routing, and firewall policies.

The existing UniFi controller did not automatically provide a private network tunnel.

The evaluation also established that conventional travel routers with compatible VPN functionality could provide similar access to home LAN services.

The Mobile Router Ultra's primary advantage was its integrated LTE connectivity and related mobile networking features.

The available project history does not confirm a completed deployment, a working VPN tunnel, or measured performance.

The documented outcome is therefore a hardware and network architecture evaluation.

---

## Lessons Learned

A UniFi controller and a UniFi gateway perform different functions.

Using UniFi networking equipment does not require replacing OPNsense.

A mobile router does not automatically gain access to the home LAN merely because both networks use UniFi products.

WireGuard and IPsec provide interoperable options for connecting third-party gateways.

Cellular connectivity and VPN connectivity are separate requirements.

CGNAT commonly affects inbound connectivity on mobile networks.

Outbound VPN tunnels are often more practical for cellular routers.

A connected VPN tunnel does not guarantee that routing and firewall policies are correct.

Remote and home networks should use non-overlapping IP ranges.

Internal DNS and multicast discovery require separate consideration.

A conventional travel router may provide the same VPN functionality without an integrated cellular modem.

The project strengthened understanding of OPNsense, UniFi networking, LTE routing, WireGuard, IPsec, Tailscale, NAT traversal, and site-to-site VPN architecture.

---

## Future Improvements

Future work may include:

- Selecting a mobile router based on actual cellular and VPN requirements.
- Configuring a WireGuard endpoint on OPNsense.
- Establishing a manual site-to-site VPN.
- Testing remote LAN routing.
- Configuring restricted firewall access.
- Integrating home DNS resolution.
- Testing SMB access to Unraid.
- Measuring Jellyfin streaming performance.
- Evaluating cellular data consumption.
- Testing LTE failover behavior.
- Comparing Tailscale-based alternatives.
- Testing a conventional WireGuard travel router.
- Evaluating persistent VPN reconnection.
- Monitoring tunnel availability through Home Assistant.
- Documenting the final network topology.
- Testing access from multiple remote devices.
- Creating a reusable portable networking configuration.

These improvements would move the project from technical evaluation toward a deployable portable extension of the home network.

---

## Summary

The UniFi Mobile Router Ultra and OPNsense Remote Access project evaluated a cellular-enabled network gateway for accessing a self-hosted home network while traveling.

The existing infrastructure used OPNsense as the primary firewall and router, alongside UniFi networking equipment and a self-hosted controller.

The investigation determined that the UniFi Mobile Router Ultra could provide LTE internet connectivity and support compatible VPN protocols for remote access.

However, the presence of UniFi equipment on both networks would not automatically establish connectivity to the home LAN.

A WireGuard or IPsec tunnel, appropriate routing, and firewall configuration would be required.

The evaluation also considered Tailscale and conventional travel routers as alternative approaches.

A key conclusion was that the Mobile Router Ultra's principal differentiator was integrated cellular connectivity rather than exclusive remote-access functionality.

Although a production deployment was not verified, the project established a practical architecture for securely extending access to Unraid, Home Assistant, Jellyfin, and other self-hosted services.

The work demonstrates knowledge of network architecture, OPNsense, UniFi, cellular networking, WireGuard, IPsec, Tailscale, VPN routing, and remote infrastructure access.

---

## Technical References

- [UniFi Mobile Router Ultra — Technical Specifications](https://techspecs.ui.com/unifi/integrations/umr-ultra-us)
- [UniFi Mobile Router — Technical Specifications](https://techspecs.ui.com/unifi/integrations/umr-us)
- [Ubiquiti — Getting Started with UniFi Mobile Routers](https://help.ui.com/hc/en-us/articles/31169556838807-Getting-Started-with-UniFi-Mobile-Router-Series)
- [Ubiquiti — WireGuard Site-to-Site VPN for UMR](https://help.ui.com/hc/en-us/articles/22318184801815-Wireguard-S2S-VPN-for-UMR)
- [Ubiquiti — UniFi VPN Overview](https://help.ui.com/hc/en-us/articles/7951513517079-UniFi-Gateway-Introduction-to-VPNs)
- [Ubiquiti — Third-Party Gateway Configuration](https://help.ui.com/hc/en-us/articles/9761080275607-Creating-Virtual-Networks-VLANs)
- [OPNsense — Official Documentation](https://docs.opnsense.org/)
- [WireGuard — Official Website](https://www.wireguard.com/)
- [Tailscale — OPNsense Installation](https://tailscale.com/docs/install/opnsense)
- [Tailscale — Subnet Routers](https://tailscale.com/docs/features/subnet-routers)
- [Unraid — Official Documentation](https://docs.unraid.net/)
- [Home Assistant — Official Documentation](https://www.home-assistant.io/docs/)
