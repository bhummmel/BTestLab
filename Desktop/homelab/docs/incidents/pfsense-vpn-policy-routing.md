
# pfSense VPN Policy Routing

## Overview

The pfSense VPN Policy Routing project focused on designing and troubleshooting selective VPN traffic routing within a self-hosted network infrastructure.

The objective was to control which devices and services used a VPN tunnel while allowing other traffic to continue using the normal WAN connection.

Rather than routing the entire network through a VPN, policy-based routing provides granular control over traffic based on source addresses, destination networks, firewall rules and selected gateways.

This approach is particularly useful in a homelab environment where different services have different connectivity requirements.

For example, media download services may benefit from VPN routing, while network management interfaces, local application traffic and latency-sensitive services may require direct connectivity.

The project was associated with the homelab's earlier pfSense firewall deployment and its broader use of VPN technologies.

The infrastructure subsequently transitioned to OPNsense, while VPN isolation continued through a Gluetun-based Docker deployment for qBittorrent.

The available project history establishes the use of pfSense, VPN and DNS infrastructure, along with later VPN-protected container workloads. However, the precise original pfSense VPN provider configuration, firewall rules and gateway assignments are not fully preserved.

This document therefore describes the architecture, configuration principles and troubleshooting methodology without claiming that every illustrated configuration was deployed.

---

## Objectives

The primary objective was to understand and implement selective traffic routing through a VPN gateway using pfSense.

Specific goals included:

- Understanding policy-based routing.
- Configuring VPN client connectivity.
- Establishing VPN gateway monitoring.
- Creating firewall rules for selective routing.
- Routing specific devices through a VPN.
- Preserving normal WAN connectivity for other devices.
- Preventing unintended traffic leaks.
- Understanding firewall rule evaluation order.
- Troubleshooting gateway selection.
- Investigating DNS routing behavior.
- Maintaining access to local network resources.
- Evaluating gateway failure behavior.
- Verifying external IP addresses.
- Monitoring VPN connectivity.
- Developing a reliable troubleshooting methodology.

An additional objective was to determine how VPN policy routing could support containerized workloads without introducing unnecessary complexity into the network.

---

## Infrastructure Environment

The project was associated with the homelab's pfSense deployment.

The environment included dedicated firewall hardware, network-attached storage and multiple self-hosted services.

| Component | Role |
|---|---|
| pfSense | Original firewall and routing platform |
| OpenVPN | VPN tunneling technology |
| WireGuard | Alternative VPN tunneling technology |
| VPN provider | External VPN endpoint |
| Intel network interfaces | Firewall network connectivity |
| Unraid / earlier TrueNAS SCALE | Application and storage hosting |
| Docker | Containerized application deployment |
| qBittorrent | VPN-protected download workload |
| DNS resolver | Local name resolution |
| OPNsense | Subsequent firewall platform |
| Gluetun | Later container-level VPN gateway |

The broader firewall infrastructure included troubleshooting network interface compatibility on a Lanner appliance and evaluating Intel i350-based networking hardware.

The homelab later adopted OPNsense for routing and firewall services.

VPN routing remained relevant after the transition, although its implementation changed.

---

## System Architecture

The intended architecture separates ordinary internet traffic from traffic requiring VPN protection.

```text
                         Internet
                            |
                   +--------+--------+
                   |                 |
                   v                 v
              ISP Gateway       VPN Provider
                   ^                 ^
                   |                 |
                   |            Encrypted Tunnel
                   |                 |
                   +--------+--------+
                            |
                            v
                     +-------------+
                     | pfSense     |
                     | Firewall    |
                     +-------------+
                            |
                     Policy Routing
                            |
               +------------+------------+
               |                         |
               v                         v
        Standard LAN Traffic       VPN-Routed Traffic
               |                         |
               v                         v
        Normal Devices             Selected Devices
        and Services               and Applications
```

The firewall evaluates traffic against configured rules.

Matching traffic can be assigned to a specific gateway rather than automatically following the system's default route.

This provides selective VPN routing while preserving the existing network architecture.

---

## Understanding Policy-Based Routing

Traditional routing determines the next hop primarily through the routing table.

Policy-based routing introduces additional selection criteria.

In pfSense, firewall rules can specify a gateway for matching traffic.

This allows traffic to be directed through a gateway other than the system default.

For example, a firewall rule can match traffic originating from a particular host and assign it to a VPN gateway.

Other hosts continue using the normal WAN gateway.

The important distinction is that the routing decision is influenced by firewall policy rather than only by destination-based routing.

---

## Default Routing Versus Policy Routing

A conventional network generally uses a default gateway for traffic destined outside directly connected networks.

```text
LAN Device
    |
    v
pfSense
    |
    v
Default WAN Gateway
    |
    v
Internet
```

With policy-based routing:

```text
LAN Device
    |
    v
pfSense Firewall Rule
    |
    +---- Match VPN Policy ----> VPN Gateway
    |
    +---- No VPN Policy -------> Default WAN
```

The project emphasized that a working VPN tunnel does not automatically mean traffic is using it.

The appropriate firewall policy must select the VPN gateway for the intended traffic.

---

## VPN Client Configuration

pfSense can operate as a VPN client using supported tunneling technologies.

The specific configuration depends on the VPN provider and protocol.

Common parameters include:

- VPN server hostname.
- Remote server port.
- Transport protocol.
- Authentication credentials.
- Certificates or keys.
- Tunnel interface.
- Encryption settings.
- Routing behavior.
- DNS requirements.

The project considered the importance of establishing a functioning VPN connection before configuring policy routing.

If the underlying tunnel is unavailable, firewall policy changes cannot provide reliable VPN connectivity.

---

## OpenVPN Architecture

OpenVPN creates an encrypted tunnel between the firewall and a remote VPN endpoint.

When configured as a client, pfSense establishes the tunnel using the provider's connection settings.

A simplified architecture is:

```text
Local Network
     |
     v
   pfSense
     |
     v
OpenVPN Client
     |
     v
Encrypted Tunnel
     |
     v
VPN Provider
     |
     v
Internet
```

OpenVPN can use UDP or TCP transport.

The choice depends on the provider's configuration and operational requirements.

The project emphasized verifying tunnel establishment independently from routing policy.

---

## WireGuard Considerations

WireGuard provides an alternative VPN architecture using a comparatively compact protocol design.

Its configuration generally involves:

- Local private key.
- Peer public key.
- Tunnel addresses.
- Endpoint hostname and port.
- Allowed IP ranges.
- Optional keepalive settings.

The routing and gateway integration depends on the pfSense release and WireGuard implementation.

The project considered WireGuard as an alternative VPN technology, but the available history does not establish a specific WireGuard policy-routing deployment on the original pfSense firewall.

The investigation emphasized that protocol selection and firewall policy are separate design decisions.

---

## VPN Interface Assignment

In pfSense, an established VPN tunnel can be assigned as an interface where supported by the VPN implementation.

This can simplify gateway management and firewall configuration.

An assigned interface may be used for:

- Gateway creation.
- Interface-specific firewall rules.
- NAT configuration.
- Monitoring.
- Traffic diagnostics.

The project emphasized that a VPN interface being present does not automatically establish correct routing.

The gateway, firewall policies and NAT behavior must also be verified.

---

## VPN Gateway Configuration

A gateway represents a next-hop routing destination.

For policy-based routing, pfSense needs an appropriate gateway associated with the VPN interface.

The gateway can then be selected in matching firewall rules.

Important configuration considerations include:

- Correct VPN interface.
- Appropriate gateway address or dynamic gateway handling.
- Gateway availability.
- Monitoring configuration.
- Routing behavior during tunnel failure.

The project emphasized verifying that the gateway is actually usable rather than relying exclusively on its presence in the interface.

---

## Gateway Monitoring

pfSense can monitor gateway availability using its gateway monitoring infrastructure.

A monitoring target may be used to evaluate whether the gateway remains reachable.

However, selecting a monitoring target requires care.

A target that is unreachable for unrelated reasons may cause a healthy tunnel to appear unavailable.

Conversely, a target that remains reachable through an unintended route may provide misleading results.

The project emphasized distinguishing tunnel status, gateway monitoring status and actual internet connectivity through the VPN.

These are related but not identical conditions.

---

## Firewall Rule Evaluation

pfSense firewall rules are generally evaluated according to their configured order and rule-processing behavior.

For ordinary interface rules, the first matching rule commonly determines the applicable action.

This makes rule ordering particularly important for policy routing.

A specific VPN-routing rule must be evaluated before a broader rule that would otherwise allow traffic through the default gateway.

For example:

| Order | Source | Destination | Gateway |
|---|---|---|---|
| 1 | Selected VPN host | Local networks | Normal routing |
| 2 | Selected VPN host | Internet destinations | VPN gateway |
| 3 | General LAN | Any | Default gateway |

This is an illustrative policy design.

The actual rule structure depends on the network topology, local subnets and desired traffic restrictions.

---

## Source-Based Routing

One of the simplest policy-routing approaches is selecting traffic based on its source IP address.

For example, a dedicated host or container network could be assigned a static address.

A firewall rule can then match that address and select the VPN gateway.

This provides a straightforward method for isolating a specific workload.

However, the approach depends on consistent source addressing.

The project emphasized the importance of DHCP reservations or static addressing when firewall policies rely on individual device IP addresses.

---

## Firewall Aliases

pfSense supports aliases that group addresses, networks and ports into reusable objects.

An alias can represent multiple devices requiring VPN routing.

For example:

```text
VPN_CLIENTS
    |
    +-- Media Download Host
    +-- Test Virtual Machine
    +-- Selected Workstation
```

A firewall rule can reference the alias instead of maintaining separate rules for each device.

This improves maintainability and reduces configuration duplication.

The project considered aliases as a way to make policy routing easier to audit and modify.

---

## Local Network Exclusions

Traffic between local devices generally should not be routed through an external VPN unless there is a specific reason.

For example, a VPN-routed workstation may still need access to:

- NAS shares.
- Home Assistant.
- Local DNS.
- Network printers.
- Management interfaces.
- Other internal services.

If a broad policy-routing rule captures all destinations, it may interfere with local connectivity.

The project emphasized placing appropriate local-network exclusions before the VPN-routing rule.

The exact exclusions depend on the network topology.

---

## VLAN Considerations

The homelab's broader networking environment included VLAN and firewall segmentation planning.

Policy-based routing can be applied to traffic entering through different network interfaces.

For example, a dedicated VLAN could contain devices that require VPN routing.

```text
                 pfSense
                    |
          +---------+---------+
          |                   |
          v                   v
      Main LAN             VPN VLAN
          |                   |
          v                   v
     Default WAN          VPN Gateway
```

This can simplify policy management by grouping devices according to network purpose.

However, VLAN segmentation and VPN routing are separate functions.

A VLAN does not automatically provide VPN protection.

The firewall must enforce the intended routing and access policies.

---

## Outbound NAT

Network Address Translation is an important component of VPN policy routing.

Traffic leaving through the VPN interface may need its source address translated to an address appropriate for that tunnel.

The required NAT behavior depends on the VPN provider and tunnel configuration.

In pfSense, outbound NAT may be managed through automatic, hybrid or manual configurations.

The project emphasized verifying NAT behavior when traffic reaches the VPN gateway but cannot communicate with external destinations.

A routing rule alone does not guarantee that return traffic can reach the original client.

---

## NAT Troubleshooting

A useful diagnostic distinction is whether traffic is:

1. Matching the intended firewall rule.
2. Entering the VPN interface.
3. Receiving appropriate source translation.
4. Reaching the remote destination.
5. Receiving return traffic.

If packets leave through the tunnel but no responses return, NAT configuration may be one possible cause.

Other causes include VPN provider restrictions, remote routing and firewall policy.

The project emphasized using packet captures and firewall state information to identify the actual failure point.

---

## DNS Routing Considerations

DNS is a particularly important consideration in selective VPN routing.

A device may send ordinary internet traffic through the VPN while continuing to use a resolver outside the tunnel.

Whether this constitutes a privacy concern depends on the intended threat model and DNS architecture.

The homelab has used local DNS infrastructure, including Unbound.

The project considered how local DNS resolution interacts with policy routing.

A client using the firewall as its DNS resolver does not necessarily cause the firewall's own upstream DNS queries to follow that client's VPN policy.

Firewall-originated traffic is governed by different routing considerations.

The investigation emphasized treating DNS routing as a separate configuration requirement.

---

## DNS Leak Prevention

A VPN-routed device may use:

- A local DNS resolver.
- A VPN provider's DNS service.
- An external public resolver.
- Application-specific DNS configuration.

The preferred option depends on the privacy requirements.

A common design is to provide clients with a controlled resolver and restrict unwanted direct DNS traffic.

However, DNS-over-HTTPS and other encrypted DNS mechanisms complicate simple port-based restrictions.

The project emphasized verifying actual resolver behavior rather than assuming that all DNS requests follow the VPN.

---

## VPN Failure Behavior

A critical design question is what happens when the VPN tunnel becomes unavailable.

Possible outcomes include:

- Traffic stops.
- Traffic follows another available gateway.
- Existing connections fail.
- New connections use an unintended route.

The desired behavior depends on the workload.

For privacy-sensitive traffic, automatic fallback to the ordinary WAN may be unacceptable.

The project emphasized designing for a fail-closed outcome when VPN use is mandatory.

This requires more than simply selecting a VPN gateway in an allow rule.

Gateway monitoring, policy routing, firewall state handling and fallback behavior must be considered together.

---

## Kill-Switch Architecture

A VPN kill switch is intended to prevent protected traffic from escaping through the normal WAN when the VPN becomes unavailable.

A conceptual architecture is:

```text
VPN-Protected Device
         |
         v
      pfSense
         |
         v
    VPN Available?
         |
      +--+--+
      |     |
     Yes    No
      |     |
      v     v
   VPN     Block
  Gateway  Traffic
```

The project emphasized that a kill switch should be validated through controlled failure testing.

A configuration that appears correct while the VPN is connected may behave differently when the tunnel disconnects.

---

## Firewall State Management

pfSense is a stateful firewall.

Established connections are tracked in the firewall state table.

When policy-routing rules change, existing states may continue using previously established paths.

This can make troubleshooting confusing.

For example, a new firewall rule may be correct, but an existing connection may continue using an older route.

The project emphasized distinguishing newly established connections from existing states.

Where appropriate, relevant states can be cleared through supported pfSense administrative controls.

However, clearing states can interrupt active connections and should be performed carefully.

---

## Packet Capture Diagnostics

Packet captures provide direct evidence of traffic behavior.

pfSense includes packet capture functionality through its diagnostic interface.

Captures can be performed on:

- LAN interfaces.
- VPN interfaces.
- WAN interfaces.
- Relevant VLAN interfaces.

The project emphasized comparing captures at different points in the traffic path.

For example, a packet appearing on the LAN interface but not the VPN interface suggests a different problem from a packet successfully entering the VPN tunnel.

This approach helps isolate firewall, routing and NAT issues.

---

## Command-Line Diagnostics

pfSense provides command-line access for advanced diagnostics.

Because pfSense is based on FreeBSD, its networking tools differ from those of a conventional Linux system.

Useful commands include:

```sh
netstat -rn
```

This displays routing table information.

```sh
ifconfig
```

This displays interface configuration.

```sh
pfctl -sr
```

This displays loaded firewall rules.

```sh
pfctl -ss
```

This displays active firewall states.

These commands can help correlate the configured firewall policy with actual system behavior.

The project emphasized that pfSense's generated firewall rules may be more complex than the simplified rules shown in its administrative interface.

---

## External IP Verification

A practical VPN validation test is checking the public IP address used by a selected client.

A Linux client can query an external IP identification service:

```bash
curl https://api.ipify.org
```

The result can be compared with the expected VPN provider exit address.

The same test should be performed from a device intended to use the normal WAN.

This helps verify selective routing.

However, a public IP check alone does not prove that DNS routing, IPv6 traffic and VPN failure behavior are configured correctly.

The project emphasized using multiple independent tests.

---

## IPv6 Considerations

IPv6 introduces additional routing requirements.

A VPN provider may support IPv4 only, IPv6 only or dual-stack connectivity.

If the VPN policy covers only IPv4 traffic, IPv6 connections may follow a different route.

This can create unintended connectivity paths.

The project emphasized determining whether IPv6 should be routed through the VPN, restricted or handled through a separate policy.

Simply verifying an IPv4 exit address is insufficient for a dual-stack client.

---

## Selective Routing for Download Services

The homelab has used automated download services, including qBittorrent and NZBGet.

These applications have different connectivity requirements.

A VPN may be desirable for selected download traffic while other services continue using normal internet connectivity.

The project considered source-based routing as one possible way to isolate such workloads.

However, policy routing at the firewall operates on the traffic visible to the firewall.

When multiple containers share a host IP address, identifying individual application traffic can become more complicated.

This is an important distinction between device-level VPN routing and container-level VPN isolation.

---

## Container Networking Limitations

Docker containers may use bridge networking, host networking or other network configurations.

In bridge mode, container traffic may be translated through the host before reaching the firewall.

As a result, the firewall may see the host's IP address rather than a unique address for each container.

This can prevent a simple source-IP policy from distinguishing individual applications.

The project emphasized understanding how container networking affects firewall visibility.

A firewall rule cannot selectively route traffic based on a container identity that is not represented in the packets reaching the firewall.

---

## Alternative: Container-Level VPN Routing

The homelab later used Gluetun to provide VPN connectivity for qBittorrent.

This architecture places the VPN tunnel within the container environment rather than relying entirely on firewall-level policy routing.

A simplified architecture is:

```text
                 Unraid Server
                       |
              +--------+--------+
              |                 |
              v                 v
        Normal Docker       VPN Container
          Services             |
              |               v
              |            Gluetun
              |               |
              |               v
              |          VPN Provider
              |               |
              v               v
          Normal WAN       VPN Tunnel
```

This approach can provide more direct isolation for selected containers.

The project highlighted the tradeoffs between centralized firewall policy routing and application-specific VPN networking.

---

## pfSense Versus Gluetun

Both approaches can provide selective VPN connectivity, but they operate at different layers.

| Feature | pfSense Policy Routing | Gluetun Container |
|---|---|---|
| Routing control | Firewall | Container network namespace |
| Typical scope | Devices, networks or visible traffic sources | Selected containers |
| VPN management | Centralized firewall | Application hosting platform |
| Container awareness | Depends on network design | Direct container integration |
| Gateway monitoring | pfSense gateway infrastructure | Container-specific health and tunnel handling |
| Local network integration | Firewall rules and routing | Container networking configuration |
| Failure isolation | Depends on firewall policy | Depends on container configuration |

The project emphasized that neither approach is universally superior.

The correct choice depends on workload requirements and infrastructure design.

---

## Troubleshooting Methodology

The investigation can be organized into a layered diagnostic process.

### Stage 1: Verify VPN Connectivity

Confirm that the VPN client establishes a functioning tunnel.

### Stage 2: Inspect the VPN Gateway

Verify that pfSense recognizes the intended gateway.

### Stage 3: Review Firewall Rules

Confirm that the correct source traffic matches the policy-routing rule.

### Stage 4: Check Rule Ordering

Ensure that broader allow rules do not take precedence over the intended policy.

### Stage 5: Validate Local Network Exclusions

Confirm that internal traffic remains reachable.

### Stage 6: Inspect Outbound NAT

Verify that traffic leaving through the VPN uses appropriate source translation.

### Stage 7: Test External Connectivity

Compare public IP addresses from VPN-routed and ordinary clients.

### Stage 8: Verify DNS Behavior

Determine which resolvers are being used and how their traffic is routed.

### Stage 9: Test VPN Failure Behavior

Confirm that protected traffic does not unexpectedly fall back to the normal WAN.

### Stage 10: Monitor Stability

Observe gateway health, firewall states and application connectivity over time.

This methodology helps isolate routing problems without changing multiple unrelated firewall settings simultaneously.

---

## Technical Challenges

### Gateway Selection

A functioning VPN tunnel does not automatically become the route for selected traffic.

### Firewall Rule Ordering

Broad allow rules can prevent intended policy-routing rules from matching.

### NAT Configuration

Traffic may leave through the VPN but fail because of incorrect source translation.

### DNS Routing

Resolver traffic may not follow the same path as client application traffic.

### VPN Failure Behavior

Gateway loss can produce unintended fallback behavior.

### Firewall States

Existing connections may retain earlier routing decisions.

### IPv6

Dual-stack traffic may require separate routing controls.

### Container Networking

Multiple applications sharing one host IP can complicate source-based routing.

### Hardware and Platform Changes

The homelab's transition from pfSense to OPNsense required reconsidering where VPN policies should be enforced.

These challenges reinforced the importance of validating actual packet flow rather than relying exclusively on interface configuration.

---

## Security Considerations

Selective VPN routing is partly a network security and privacy control.

The project emphasized:

- Restricting VPN policy management to administrators.
- Avoiding unintended WAN fallback.
- Maintaining appropriate firewall segmentation.
- Preventing unauthorized DNS paths where required.
- Protecting VPN credentials and private keys.
- Keeping VPN software updated.
- Restricting access to firewall management interfaces.
- Reviewing NAT and firewall rules.
- Monitoring unexpected traffic paths.
- Testing IPv4 and IPv6 behavior.

The investigation reinforced that a VPN is not a substitute for firewall security.

VPN encryption protects traffic over the tunnel, while firewall rules determine which traffic is permitted and where it is routed.

---

## Reliability and Recovery

A reliable VPN policy-routing deployment should behave predictably during:

- Firewall restarts.
- VPN tunnel interruptions.
- ISP connectivity failures.
- Gateway monitoring failures.
- DNS outages.
- Configuration changes.
- VPN provider endpoint changes.

The project emphasized maintaining a known-good firewall configuration and documenting the intended routing behavior.

pfSense configuration backups are particularly important before making significant firewall or gateway changes.

A rollback procedure should restore normal network connectivity without unintentionally removing required security controls.

---

## Monitoring and Diagnostics

Useful operational indicators include:

- VPN tunnel status.
- Gateway monitoring status.
- Firewall rule counters.
- Active firewall states.
- VPN interface traffic.
- WAN interface traffic.
- DNS resolution behavior.
- Public exit IP address.
- Packet loss.
- Tunnel reconnection events.

The project emphasized monitoring both tunnel availability and actual traffic routing.

A VPN interface remaining connected does not necessarily mean that every intended client is using it.

---

## Relationship to the OPNsense Migration

The homelab subsequently transitioned from pfSense to OPNsense.

OPNsense became responsible for firewall services, routing and other network functions.

The broader infrastructure also adopted container-level VPN isolation for qBittorrent using Gluetun.

This reduced the need to route every VPN-protected application through a dedicated firewall policy.

However, the underlying concepts remain applicable.

Both pfSense and OPNsense support policy-based routing through firewall and gateway configuration.

The project contributed to understanding the tradeoffs between firewall-level traffic control and application-level VPN isolation.

The available project history does not establish that VPN policy routing was the primary reason for changing firewall platforms.

---

## Evaluation Outcome

The project examined selective VPN routing within the earlier pfSense firewall environment.

The investigation focused on VPN gateways, firewall policy, NAT, DNS behavior and routing verification.

It also considered how source-based routing interacts with containerized applications.

The broader homelab later implemented VPN-protected qBittorrent connectivity through Gluetun and transitioned to OPNsense for firewall management.

The available project history does not preserve the exact original pfSense rule set, gateway assignments or final validation results.

Accordingly, the document does not claim that a particular fail-closed routing policy was successfully tested or deployed.

The documented outcome is the architecture evaluation and troubleshooting methodology developed around selective VPN routing.

---

## Lessons Learned

A connected VPN tunnel does not automatically mean client traffic uses it.

Policy-based routing depends on correct firewall rule matching.

Firewall rule order can determine whether traffic uses the VPN or the default WAN.

Local network exclusions are important for preserving access to internal services.

Outbound NAT can be necessary for successful VPN connectivity.

DNS traffic requires separate consideration.

Gateway monitoring and actual tunnel usability are different measurements.

VPN failure behavior must be explicitly tested.

IPv4 and IPv6 traffic may follow different routing paths.

Container networking can limit the visibility of individual application traffic at the firewall.

The project strengthened understanding of firewall administration, VPN architecture, routing policy, NAT, DNS and network troubleshooting.

---

## Future Improvements

Future work may include:

- Documenting the original pfSense VPN configuration.
- Recreating equivalent policies in OPNsense where appropriate.
- Maintaining dedicated aliases for VPN-routed devices.
- Implementing automated VPN gateway monitoring.
- Testing fail-closed routing behavior.
- Verifying DNS routing through controlled tests.
- Evaluating IPv6 VPN support.
- Monitoring VPN exit addresses.
- Establishing alerts for tunnel disconnections.
- Documenting firewall state behavior during gateway failures.
- Comparing WireGuard and OpenVPN performance.
- Evaluating dedicated VPN VLANs.
- Standardizing container-level VPN configuration.
- Maintaining configuration backups and recovery procedures.
- Developing repeatable routing validation tests.

These improvements would support more reliable and maintainable VPN connectivity across the homelab.

---

## Summary

The pfSense VPN Policy Routing project focused on investigating selective VPN traffic routing within a self-hosted network infrastructure.

The work examined VPN client configuration, gateway selection, firewall rules, NAT, DNS behavior and failure handling.

A central finding was that reliable VPN policy routing requires more than establishing an encrypted tunnel.

The firewall must correctly identify the intended traffic, direct it through the appropriate gateway and prevent unintended fallback when VPN protection is mandatory.

The project also highlighted the operational differences between centralized firewall policy routing and container-level VPN isolation.

The broader homelab subsequently transitioned to OPNsense and adopted Gluetun-based VPN networking for selected Docker workloads.

Although the exact original pfSense configuration and final validation results were not established in the available project record, the work provided a structured foundation for diagnosing and designing selective VPN routing.

The project demonstrates knowledge of pfSense, firewall administration, OpenVPN, WireGuard, policy-based routing, NAT, DNS, VLAN architecture, packet analysis and network security.

---

## Technical References

- [pfSense — Policy Routing](https://docs.netgate.com/pfsense/en/latest/multiwan/policy-route.html)
- [pfSense — OpenVPN](https://docs.netgate.com/pfsense/en/latest/vpn/openvpn/index.html)
- [pfSense — WireGuard](https://docs.netgate.com/pfsense/en/latest/vpn/wireguard/index.html)
- [pfSense — Outbound NAT](https://docs.netgate.com/pfsense/en/latest/nat/outbound.html)
- [pfSense — Firewall Rule Processing](https://docs.netgate.com/pfsense/en/latest/firewall/rule-methodology.html)
- [OPNsense — Official Documentation](https://docs.opnsense.org/)
- [Gluetun — Official Repository](https://github.com/qdm12/gluetun)
- [Docker — Networking Documentation](https://docs.docker.com/engine/network/)
