# qBittorrent and Gluetun VPN Integration

## Overview

The qBittorrent and Gluetun VPN Integration project focused on deploying a containerized BitTorrent download client on an Unraid server while isolating its network traffic through a dedicated VPN connection.

The objective was to integrate qBittorrent into an existing self-hosted media management infrastructure without routing the entire NAS or its other applications through the VPN.

The implementation used Docker Compose, Gluetun, qBittorrent, and Private Internet Access (PIA), with subsequent investigation into using Proton VPN as an alternative provider.

The broader media infrastructure included Sonarr, Radarr, Lidarr, Prowlarr, NZBGet, and Jellyfin.

The principal architectural decision was to use Gluetun as a dedicated VPN gateway container and configure qBittorrent to share its network namespace.

This approach provides container-level VPN isolation while allowing unrelated services to continue using their normal network connections.

The project also involved troubleshooting qBittorrent WebUI authentication, application connectivity, indexer integration, and VPN provider configuration.

Additional work examined Prowlarr's integration with FlareSolverr, torrent indexer availability, and configuring qBittorrent as a download client for Lidarr.

The resulting architecture demonstrates practical experience with Docker networking, Linux network namespaces, VPN tunneling, firewall isolation, application integration, and network troubleshooting.

The available project history establishes the use of a Gluetun-based Compose configuration and several integration issues. It does not establish that every troubleshooting issue was resolved or that the proposed Proton VPN migration was completed.

---

## Objectives

The primary objective was to provide VPN-isolated torrent downloading within the existing Unraid media infrastructure.

Specific goals included:

- Deploying qBittorrent as a Docker container.
- Deploying Gluetun as a VPN gateway.
- Establishing an authenticated VPN connection.
- Routing qBittorrent traffic through Gluetun.
- Preventing unintended direct internet access.
- Maintaining local access to the qBittorrent WebUI.
- Integrating qBittorrent with Sonarr, Radarr, and Lidarr.
- Connecting Prowlarr to the download client.
- Troubleshooting WebUI authentication.
- Investigating torrent indexer failures.
- Evaluating FlareSolverr integration.
- Comparing PIA and Proton VPN configurations.
- Maintaining persistent application settings.
- Managing download directory permissions.
- Avoiding unnecessary VPN routing for other containers.
- Developing a maintainable Docker Compose deployment.

A secondary objective was to maintain separation between download acquisition, media organization, and media playback services.

---

## Infrastructure Environment

The project was deployed within the Unraid-based homelab environment.

| Component | Role |
|---|---|
| Unraid | NAS operating system and Docker host |
| Docker Engine | Container runtime |
| Docker Compose | Multi-container deployment |
| Gluetun | VPN client and network gateway |
| qBittorrent | BitTorrent download client |
| Private Internet Access | Original VPN provider |
| Proton VPN | Alternative VPN provider investigated |
| OpenVPN | VPN tunneling protocol |
| WireGuard | Alternative VPN protocol |
| Prowlarr | Indexer management |
| FlareSolverr | Browser challenge-solving proxy |
| Sonarr | Television library management |
| Radarr | Movie library management |
| Lidarr | Music library management |
| NZBGet | Usenet download client |
| Jellyfin | Media server |
| OPNsense | Network firewall |
| Unraid user shares | Persistent media and download storage |

The Unraid server, identified on the network as `Mouseframe`, hosted the containerized media stack.

The VPN deployment was designed to operate independently of the OPNsense firewall's primary routing configuration.

This allowed selective traffic isolation without requiring policy-based routing rules for the entire NAS.

---

## System Architecture

The architecture used two containers with a shared network namespace.

```text
                      Internet
                         |
                         v
                 +---------------+
                 | VPN Provider  |
                 | PIA / Proton  |
                 +---------------+
                         |
                    VPN Tunnel
                         |
                         v
                 +---------------+
                 | Gluetun       |
                 | VPN Gateway   |
                 | Firewall      |
                 +---------------+
                         ^
                         |
              Shared Network Namespace
                         |
                 +---------------+
                 | qBittorrent   |
                 | Download      |
                 | Client        |
                 +---------------+
                         |
                         v
                 Unraid Downloads
                         |
                +--------+--------+
                |        |        |
                v        v        v
              Sonarr   Radarr   Lidarr
                |        |        |
                +--------+--------+
                         |
                         v
                    Media Library
                         |
                         v
                      Jellyfin
```

The Gluetun container established and maintained the VPN connection.

qBittorrent used Gluetun's network namespace rather than receiving an independent Docker network interface.

Consequently, qBittorrent's outbound connections were subject to the routing and firewall rules managed by Gluetun.

The other media applications remained on their own Docker networks.

---

## Understanding Docker Network Namespaces

Linux network namespaces isolate network resources between processes.

Each namespace can have its own:

- Network interfaces.
- IP addresses.
- Routing tables.
- Firewall rules.
- Listening sockets.
- Network connections.

Docker normally creates a separate network namespace for each container.

However, Docker also supports allowing one container to share another container's namespace.

In Docker Compose, this is configured using:

```yaml
network_mode: "service:gluetun"
```

When applied to qBittorrent, the container uses the network stack belonging to Gluetun.

This is the fundamental mechanism behind the VPN isolation architecture.

---

## Why Gluetun Was Selected

Gluetun is a containerized VPN client designed to provide VPN connectivity for other containers.

It supports multiple VPN providers and protocols.

Relevant capabilities include:

- OpenVPN support.
- WireGuard support.
- VPN provider configuration.
- DNS handling.
- Firewall management.
- Connection health monitoring.
- VPN endpoint selection.
- Environment-variable configuration.
- Container network namespace sharing.

The project selected Gluetun to avoid installing and managing a VPN client directly inside the qBittorrent container.

This separated the download application from VPN connection management.

It also made changing VPN providers possible without rebuilding the qBittorrent application.

---

## Original Docker Compose Architecture

The original configuration used Gluetun with Private Internet Access.

The Gluetun service included:

```yaml
services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun

    cap_add:
      - NET_ADMIN

    devices:
      - /dev/net/tun:/dev/net/tun

    volumes:
      - /mnt/user/appdata/gluetun:/gluetun

    environment:
      VPN_SERVICE_PROVIDER: "private internet access"
      VPN_TYPE: "openvpn"
      OPENVPN_USER: "${OPENVPN_USER}"
      OPENVPN_PASSWORD: "${OPENVPN_PASSWORD}"
```

This represents the relevant structure of the original deployment, with credentials replaced by environment-variable references.

The `NET_ADMIN` capability allows Gluetun to configure network interfaces and routing.

The `/dev/net/tun` device provides access to the Linux TUN interface used by OpenVPN.

The persistent volume stores Gluetun application data.

The VPN username and password are supplied through environment variables rather than being embedded directly into the published Compose file.

---

## Complete Example Docker Compose Deployment

The following example demonstrates a Gluetun and qBittorrent deployment using PIA.

It is an illustrative configuration rather than a verified copy of the final production deployment.

```yaml
services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun

    cap_add:
      - NET_ADMIN

    devices:
      - /dev/net/tun:/dev/net/tun

    environment:
      VPN_SERVICE_PROVIDER: "private internet access"
      VPN_TYPE: "openvpn"
      OPENVPN_USER: "${OPENVPN_USER}"
      OPENVPN_PASSWORD: "${OPENVPN_PASSWORD}"
      TZ: "America/New_York"

    volumes:
      - /mnt/user/appdata/gluetun:/gluetun

    ports:
      - "8080:8080"

    restart: unless-stopped

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent

    network_mode: "service:gluetun"

    depends_on:
      - gluetun

    environment:
      PUID: "99"
      PGID: "100"
      TZ: "America/New_York"
      WEBUI_PORT: "8080"

    volumes:
      - /mnt/user/appdata/qbittorrent:/config
      - /mnt/user/arrmedia/downloads:/downloads

    restart: unless-stopped
```

The download directory is illustrative and must be adjusted to match the actual Unraid share configuration.

The `PUID` and `PGID` values are common Unraid examples, not universal requirements.

The example also assumes that the Gluetun firewall configuration permits the intended WebUI access.

For reproducibility, production deployments should consider pinning container image versions instead of using `latest`.

---

## Understanding `network_mode`

The most important configuration setting is:

```yaml
network_mode: "service:gluetun"
```

This instructs Docker Compose to attach qBittorrent to Gluetun's network namespace.

As a result:

- qBittorrent does not receive its own independent network interface.
- qBittorrent uses Gluetun's routing table.
- qBittorrent shares Gluetun's network interfaces.
- qBittorrent is subject to Gluetun's firewall rules.
- Ports must be published through the Gluetun service.

This architecture provides stronger isolation than simply configuring a SOCKS proxy inside qBittorrent.

A proxy configured at the application level may not necessarily cover every network operation.

Network namespace sharing places routing and firewall enforcement outside the application itself.

---

## VPN Kill-Switch Behavior

A major design requirement was preventing qBittorrent from using the ordinary internet connection when the VPN became unavailable.

Gluetun implements firewall rules intended to restrict traffic to permitted paths.

Under normal operation, qBittorrent's external traffic follows the VPN tunnel.

If the tunnel becomes unavailable, the firewall should prevent unauthorized traffic from leaving through the ordinary WAN route.

However, this protection depends on the actual Gluetun configuration, firewall state, and container privileges.

The project emphasized verifying kill-switch behavior rather than assuming that namespace sharing alone guarantees protection.

A complete validation should include testing the container's external IP address and observing connectivity when the VPN tunnel is intentionally interrupted.

The available project history does not preserve a completed kill-switch validation test.

---

## Private Internet Access Configuration

The original VPN provider was Private Internet Access.

Gluetun supports PIA through its provider configuration.

The relevant environment variables included:

```yaml
environment:
  VPN_SERVICE_PROVIDER: "private internet access"
  VPN_TYPE: "openvpn"
  OPENVPN_USER: "${OPENVPN_USER}"
  OPENVPN_PASSWORD: "${OPENVPN_PASSWORD}"
```

The VPN credentials must correspond to the provider's supported OpenVPN authentication method.

These credentials may differ from the username and password used to access the provider's website.

The project emphasized distinguishing VPN tunnel credentials from account-management credentials.

---

## Protecting VPN Credentials

The original Compose configuration included VPN authentication information.

For a public technical portfolio, credentials must never be published.

A safer configuration uses a local `.env` file.

Example:

```dotenv
OPENVPN_USER=replace_with_vpn_username
OPENVPN_PASSWORD=replace_with_vpn_password
```

The Compose file can reference these values using:

```yaml
OPENVPN_USER: "${OPENVPN_USER}"
OPENVPN_PASSWORD: "${OPENVPN_PASSWORD}"
```

The `.env` file should be excluded from version control.

For example:

```gitignore
.env
*.secret
```

The project emphasized that environment variables simplify configuration management but do not automatically make secrets inaccessible.

Container inspection and host-level access may still expose them.

---

## TUN Device Access

OpenVPN commonly requires access to the Linux TUN device.

The Docker Compose configuration exposed:

```yaml
devices:
  - /dev/net/tun:/dev/net/tun
```

The TUN device allows software to create a virtual network interface for routed IP traffic.

Without access to the device, Gluetun may fail to establish an OpenVPN tunnel.

On the Unraid host, the device can be checked using:

```bash
ls -l /dev/net/tun
```

The project emphasized distinguishing missing device access from VPN authentication failures.

Both can prevent tunnel establishment, but they require different corrective actions.

---

## Linux Network Capabilities

Gluetun requires permission to configure network interfaces and routing.

The Compose configuration granted:

```yaml
cap_add:
  - NET_ADMIN
```

This capability permits network administration operations within the container's network namespace.

It is necessary for many VPN configurations.

However, it also increases the privileges available to the container.

The project emphasized granting only the capabilities required by the application.

qBittorrent itself did not need an independent `NET_ADMIN` capability.

---

## VPN Connection Diagnostics

Gluetun logs provide information about VPN startup and connection status.

The container can be inspected using:

```bash
docker logs gluetun
```

For continuous monitoring:

```bash
docker logs -f gluetun
```

Relevant information may include:

- VPN provider selection.
- OpenVPN initialization.
- Authentication results.
- Tunnel interface creation.
- Assigned tunnel addresses.
- Firewall initialization.
- DNS configuration.
- Connection failures.
- Reconnection attempts.

The project emphasized checking Gluetun logs before troubleshooting qBittorrent connectivity.

If the VPN gateway is unavailable, the download client may be unable to reach external services.

---

## Inspecting the VPN Network Interface

The VPN container's interfaces can be inspected using:

```bash
docker exec gluetun ip addr
```

Routing information can be examined using:

```bash
docker exec gluetun ip route
```

These commands help determine whether the VPN interface exists and how traffic is routed.

For an OpenVPN deployment, a tunnel interface such as `tun0` may appear.

The actual interface name depends on the VPN configuration.

The project emphasized verifying the network state rather than relying exclusively on container status.

A running container does not necessarily indicate an established VPN connection.

---

## Verifying the External IP Address

One method of checking VPN routing is to inspect the public IP address visible from inside the VPN network namespace.

For example, if an HTTP client is available inside Gluetun:

```bash
docker exec gluetun wget -qO- https://api.ipify.org
```

Alternatively, a temporary diagnostic container can share Gluetun's network namespace.

```bash
docker run --rm \
  --network container:gluetun \
  curlimages/curl:latest \
  https://api.ipify.org
```

The returned address should correspond to the intended VPN exit connection.

The project emphasized comparing this address with the host's ordinary WAN address.

A difference provides useful evidence of VPN routing, although it does not by itself prove that all possible traffic is protected against leaks.

---

## qBittorrent Container Deployment

qBittorrent was deployed as a separate application container.

Its responsibilities included:

- Managing torrent transfers.
- Maintaining download queues.
- Handling completed downloads.
- Providing a WebUI.
- Communicating with media management applications.
- Writing downloaded files to persistent storage.

The qBittorrent container shared Gluetun's network namespace but maintained its own filesystem mounts and application configuration.

This allowed the download client to be updated or reconfigured independently of the VPN gateway.

---

## qBittorrent WebUI Access

The qBittorrent WebUI provides administrative access through a browser.

In the example configuration, the interface listens on port 8080.

Because qBittorrent shares Gluetun's network namespace, the port must be published on Gluetun.

For example:

```yaml
gluetun:
  ports:
    - "8080:8080"
```

Publishing the port on qBittorrent itself is not the appropriate configuration when using `network_mode: "service:gluetun"`.

The project emphasized understanding the relationship between shared network namespaces and Docker port publishing.

---

## WebUI Authentication Troubleshooting

One issue encountered during the project involved obtaining the correct qBittorrent WebUI credentials.

Modern qBittorrent versions may generate a temporary administrator password when no permanent password has been configured.

The temporary password may appear in the application logs.

For example:

```bash
docker logs qbittorrent
```

The exact behavior depends on the installed qBittorrent version and container image.

The project emphasized checking the application logs and image documentation rather than assuming that a historical default password remains valid.

After gaining access, the administrator should configure a permanent password.

---

## Lidarr Download Client Authentication

Another issue occurred while attempting to add qBittorrent as a download client in Lidarr.

The integration reported incorrect credentials despite using the expected administrator username.

This required distinguishing several possible causes:

- Incorrect WebUI password.
- Temporary password changes.
- Incorrect host or port.
- Network reachability problems.
- WebUI authentication configuration.
- Reverse proxy or host-header restrictions.
- Incorrect download client settings.

The project emphasized validating the qBittorrent WebUI independently before configuring Lidarr.

A successful browser login helps establish that the credentials work, although it does not guarantee that another container can reach the same endpoint.

---

## Docker Networking Between Applications

The media management applications did not share qBittorrent's network namespace.

Consequently, they required a reachable network address for the qBittorrent WebUI.

A common mistake is using:

```text
localhost
```

inside a separate container.

Within Docker, `localhost` normally refers to the current container's own network namespace.

It does not automatically refer to the Unraid host or another container.

For a media management application running on a separate Docker network, the qBittorrent connection must use a reachable address and port.

Depending on the deployment, this may be the Unraid host's LAN address and the port published by Gluetun.

---

## Integrating qBittorrent with Lidarr

Lidarr uses download clients to acquire music releases selected for monitoring.

The integration requires:

- qBittorrent host address.
- WebUI port.
- Username.
- Password.
- Download category.
- Compatible filesystem paths.

An illustrative configuration is:

```text
Download Client: qBittorrent

Host: UNRAID_SERVER_IP

Port: 8080

Username: admin

Password: configured_webui_password

Category: lidarr
```

The actual hostname, port, and credentials depend on the deployment.

The project emphasized testing the connection before enabling automated downloads.

---

## Integrating Sonarr and Radarr

The same qBittorrent instance can serve multiple media management applications.

Separate download categories help distinguish content types.

For example:

| Application | Example Category |
|---|---|
| Sonarr | `sonarr` |
| Radarr | `radarr` |
| Lidarr | `lidarr` |

Each application can submit downloads to qBittorrent using its assigned category.

Completed downloads can then be imported into the appropriate media library.

The project emphasized that categories provide organizational separation but do not create independent download clients.

All categories still use the same qBittorrent instance and VPN network connection.

---

## Download Directory Architecture

The homelab maintained separate download and media library directories.

The broader media environment included paths such as:

```text
/entertainment/arrmedia/downloads/Movies
/entertainment/arrmedia/downloads/TV
```

The project required mapping the relevant host directories into qBittorrent and the media management applications.

A simplified architecture is:

```text
Unraid Storage
      |
      +-- Downloads
      |      |
      |      +-- Movies
      |      +-- TV
      |      +-- Music
      |
      +-- Media
             |
             +-- Movies
             +-- TV
             +-- Music
```

The exact folder names and mappings depend on the current Unraid configuration.

The project emphasized maintaining consistent container-visible paths wherever practical.

---

## Remote Path Mapping

Sonarr, Radarr, and Lidarr must locate the files downloaded by qBittorrent.

If qBittorrent reports a path that is not accessible from another container, the media management application may be unable to import the completed download.

For example:

```text
qBittorrent:
/downloads/Movies
```

If Radarr sees the same host directory as:

```text
/data/downloads/Movies
```

the reported path may require translation.

Remote path mappings can provide this translation when necessary.

However, consistent volume mounts are generally easier to maintain.

The project emphasized understanding the distinction between host paths and container-visible paths.

---

## File Ownership and Permissions

Downloaded files must be accessible to the media management applications.

Potential problems include:

- Incorrect UID or GID.
- Restrictive file permissions.
- Inconsistent container user identities.
- Missing write permissions.
- Incorrect directory ownership.
- Inaccessible Unraid shares.

The project emphasized coordinating container permissions across the media stack.

A successful torrent download does not guarantee that Sonarr, Radarr, or Lidarr can import the resulting files.

---

## Prowlarr Integration

Prowlarr was used to manage indexers for the media automation stack.

It provides centralized indexer configuration and integration with applications such as Sonarr, Radarr, and Lidarr.

The project involved configuring torrent indexers in addition to the existing Usenet infrastructure.

NZBgeek was already being used as a Usenet indexer.

However, a Usenet indexer does not automatically provide torrent search results.

The project therefore required evaluating additional torrent-compatible indexers.

---

## Torrent Indexers Versus Usenet Indexers

Usenet and BitTorrent use different acquisition systems.

| Feature | Usenet | BitTorrent |
|---|---|---|
| Download client | NZBGet | qBittorrent |
| Indexer format | NZB | Torrent or magnet |
| Transfer source | Usenet servers | BitTorrent peers |
| Network protocol | NNTP / HTTPS-based access | BitTorrent |
| Media manager integration | Sonarr, Radarr, Lidarr | Sonarr, Radarr, Lidarr |

The project emphasized that installing qBittorrent does not automatically make existing Usenet indexers compatible with torrents.

Torrent acquisition requires compatible indexers and appropriate application configuration.

---

## FlareSolverr Integration

FlareSolverr was deployed to assist with websites that present supported browser-based challenges.

The service was configured in Prowlarr as an indexer proxy.

During testing, the connection test succeeded, but the proxy later appeared disabled.

This highlighted an important distinction between:

- A successful connection test.
- A correctly configured proxy.
- An indexer actually assigned to use that proxy.
- A proxy that remains operational during real requests.

The project emphasized examining proxy configuration, tags, logs, and indexer assignments.

A successful test alone does not guarantee that Prowlarr will use the proxy for every indexer.

FlareSolverr also does not guarantee compatibility with every website or challenge mechanism.

---

## Prowlarr Indexer Proxy Troubleshooting

The project encountered a warning indicating that all indexer proxies were unavailable because of failures.

Potential diagnostic areas included:

- Proxy container availability.
- Incorrect service address.
- Docker network connectivity.
- Timeout settings.
- Indexer proxy assignments.
- FlareSolverr errors.
- Unsupported challenge behavior.
- Failed upstream requests.

The project emphasized checking both Prowlarr and FlareSolverr logs.

A connection test from the configuration interface may succeed while subsequent requests fail for different reasons.

---

## Troubleshooting Docker Connectivity

Docker provides several useful diagnostic commands.

List running containers:

```bash
docker ps
```

Inspect the Gluetun container:

```bash
docker inspect gluetun
```

Inspect qBittorrent:

```bash
docker inspect qbittorrent
```

Inspect container logs:

```bash
docker logs --tail 100 gluetun
```

```bash
docker logs --tail 100 qbittorrent
```

Inspect Docker networks:

```bash
docker network ls
```

These commands help distinguish application errors from network configuration problems.

---

## Evaluating Proton VPN

The project subsequently investigated replacing Private Internet Access with Proton VPN.

This required evaluating differences in provider configuration and authentication.

Gluetun supports Proton VPN through provider-specific settings.

The migration required considering:

- Supported VPN protocols.
- OpenVPN credentials.
- WireGuard configuration.
- Server selection.
- Port forwarding availability.
- Provider-specific requirements.
- Firewall behavior.
- Tunnel health monitoring.

The project emphasized that changing VPN providers should not require redesigning the entire qBittorrent deployment.

The Gluetun architecture isolates provider configuration from the download application.

---

## Example Proton VPN Configuration

A Proton VPN OpenVPN configuration may use a structure similar to:

```yaml
environment:
  VPN_SERVICE_PROVIDER: "protonvpn"
  VPN_TYPE: "openvpn"
  OPENVPN_USER: "${PROTON_OPENVPN_USER}"
  OPENVPN_PASSWORD: "${PROTON_OPENVPN_PASSWORD}"
```

The actual credentials must be obtained through Proton VPN's supported manual connection configuration.

They may differ from the account's normal login credentials.

Additional settings may be required for specific server features or port-forwarding configurations.

This example illustrates the provider change; it does not represent a verified final migration.

---

## Port Forwarding Considerations

Port forwarding can improve inbound BitTorrent peer connectivity.

However, VPN providers differ in whether and how they support forwarded ports.

Important considerations include:

- Provider support.
- Selected VPN server.
- Supported protocol.
- Dynamically assigned ports.
- Firewall configuration.
- qBittorrent listening port.
- Port assignment changes after reconnection.

The project emphasized that publishing a Docker port is not equivalent to receiving a forwarded port from the VPN provider.

Docker port publishing controls access between the host and container.

VPN port forwarding controls whether external peers can initiate connections through the VPN provider.

These are separate networking functions.

---

## DNS Leak Considerations

DNS resolution can reveal network activity if requests bypass the intended VPN path.

Gluetun includes DNS handling intended to support VPN-isolated workloads.

The project emphasized verifying that qBittorrent's DNS requests follow the intended configuration.

Potential risks include:

- Incorrect resolver settings.
- Application-specific DNS behavior.
- IPv6 routing differences.
- Misconfigured firewall exceptions.
- VPN disconnections.

A public IP check alone does not establish that DNS traffic is using the expected resolver.

---

## IPv6 Considerations

IPv6 introduces additional routing and firewall considerations.

A VPN deployment must account for both IPv4 and IPv6 behavior.

If the VPN supports only IPv4 but the application can reach the internet over IPv6 through another interface, unintended traffic paths may exist.

The project emphasized evaluating IPv6 behavior as part of VPN isolation testing.

The correct configuration depends on the provider, Gluetun version, Docker networking, and host environment.

---

## Troubleshooting Methodology

The investigation can be organized into a layered diagnostic process.

### Stage 1: Verify Container Startup

Confirm that Gluetun and qBittorrent are running.

### Stage 2: Verify VPN Initialization

Review Gluetun logs for tunnel establishment and authentication errors.

### Stage 3: Inspect Network Interfaces

Confirm that the expected VPN tunnel interface exists.

### Stage 4: Verify Routing

Inspect the routing table inside Gluetun.

### Stage 5: Check the External IP Address

Confirm that traffic from the shared namespace exits through the expected VPN endpoint.

### Stage 6: Verify WebUI Access

Confirm that qBittorrent is reachable through the port published by Gluetun.

### Stage 7: Resolve Authentication

Check qBittorrent logs and configure a known administrator password.

### Stage 8: Test Media Manager Connectivity

Verify that Sonarr, Radarr, and Lidarr can authenticate to qBittorrent.

### Stage 9: Verify Download Paths

Confirm that completed downloads are accessible to the media managers.

### Stage 10: Test Indexer Connectivity

Verify that Prowlarr can communicate with its configured indexers and proxies.

### Stage 11: Validate VPN Isolation

Test external connectivity and firewall behavior under normal and disconnected conditions.

### Stage 12: Review Recovery Behavior

Confirm that the deployment behaves predictably after container and host restarts.

This methodology separates VPN connectivity, application authentication, filesystem access, and indexer problems.

---

## Technical Challenges

### Shared Network Namespace Configuration

qBittorrent required access to Gluetun's network stack without creating an independent external route.

### WebUI Port Exposure

The qBittorrent WebUI port had to be published through Gluetun.

### Authentication Changes

Modern qBittorrent versions may generate temporary administrator passwords, complicating initial setup.

### Lidarr Integration

Lidarr initially rejected the expected qBittorrent credentials.

### Indexer Compatibility

Existing Usenet indexers could not automatically serve as torrent indexers.

### FlareSolverr Behavior

A successful proxy test did not guarantee that the proxy remained usable for actual indexer requests.

### Filesystem Mapping

Downloaded files needed to be accessible to the media management applications.

### VPN Provider Migration

Changing from PIA to Proton VPN required provider-specific configuration.

### VPN Isolation Validation

The architecture required verification that external traffic remained restricted to the intended VPN connection.

These challenges demonstrated the importance of troubleshooting each infrastructure layer independently.

---

## Security Considerations

The project incorporated several security principles.

### Network Isolation

qBittorrent's external traffic was routed through the dedicated VPN namespace.

### Least Privilege

The `NET_ADMIN` capability was assigned to Gluetun rather than unnecessarily granting it to every application.

### Credential Protection

VPN and WebUI credentials were treated as secrets and excluded from public configuration examples.

### Firewall Enforcement

Gluetun's firewall was used to restrict unintended traffic paths.

### Administrative Access

The qBittorrent WebUI required authentication.

### Container Separation

The remaining media management applications did not need to share the VPN namespace.

### Configuration Maintenance

Container versions, credentials, and firewall settings required ongoing review.

The project emphasized that a VPN is one component of network security, not a substitute for correct access controls and application configuration.

---

## Reliability and Recovery

The deployment was intended to operate continuously on the Unraid server.

Important reliability considerations included:

- Automatic container restarts.
- Persistent configuration storage.
- VPN reconnection behavior.
- Health monitoring.
- Stable download paths.
- Credential persistence.
- Application update compatibility.
- Container startup dependencies.
- VPN firewall initialization.
- Provider availability.

The project emphasized that `depends_on` controls startup ordering but does not, by itself, guarantee that the VPN tunnel is ready.

A robust deployment should account for VPN readiness and reconnection behavior.

---

## Evaluation Outcome

The project established a Gluetun-based architecture for routing qBittorrent traffic through a dedicated VPN connection.

The original configuration used Private Internet Access with OpenVPN.

The deployment incorporated Docker network namespace sharing, persistent configuration directories, and integration with the existing Unraid media infrastructure.

Subsequent troubleshooting addressed qBittorrent WebUI credentials, Lidarr authentication, Prowlarr indexer compatibility, and FlareSolverr proxy behavior.

The project also evaluated changing the VPN provider to Proton VPN.

The available project history does not establish that the Proton VPN migration was completed or that every authentication and indexer issue was permanently resolved.

Likewise, a completed VPN kill-switch test is not documented.

The confirmed technical outcome is the architecture, configuration work, and diagnostic methodology developed during the integration.

---

## Lessons Learned

Docker network namespaces provide a practical mechanism for routing selected containers through a VPN.

A dedicated VPN gateway avoids requiring VPN configuration inside every application.

Ports belonging to containers sharing a network namespace must be published through the gateway container.

A running VPN container does not necessarily indicate an established tunnel.

A public IP check is useful but insufficient as a complete leak-prevention test.

qBittorrent authentication behavior can vary between software versions.

Docker containers cannot generally reach one another using `localhost` unless they share a network namespace.

Consistent filesystem mounts simplify completed-download imports.

Usenet and torrent indexers require different download clients and compatible indexer definitions.

A successful proxy connection test does not guarantee reliable upstream requests.

Changing VPN providers requires reviewing authentication, protocol, server selection, and port-forwarding behavior.

The project strengthened understanding of Docker Compose, Linux networking, OpenVPN, VPN isolation, application authentication, and self-hosted media automation.

---

## Future Improvements

Future work may include:

- Completing and documenting the Proton VPN migration.
- Evaluating WireGuard as an alternative to OpenVPN.
- Testing Gluetun firewall behavior during VPN disconnections.
- Implementing automated VPN connectivity monitoring.
- Adding alerts for tunnel failures.
- Verifying DNS and IPv6 isolation.
- Evaluating VPN provider port forwarding.
- Pinning container image versions.
- Standardizing download categories.
- Consolidating container volume mappings.
- Improving Lidarr download client authentication reliability.
- Reviewing Prowlarr indexer and proxy health.
- Implementing secure credential management.
- Documenting a complete disaster recovery procedure.
- Testing the deployment after Unraid host restarts.

These improvements would increase the reliability, security, and maintainability of the media download infrastructure.

---

## Summary

The qBittorrent and Gluetun VPN Integration project focused on implementing a VPN-isolated BitTorrent download client within an Unraid-based homelab.

The architecture used Docker Compose and Linux network namespace sharing to route qBittorrent through a dedicated Gluetun VPN gateway.

The original VPN configuration used Private Internet Access and OpenVPN, with subsequent investigation into Proton VPN.

The project also involved integrating qBittorrent with Lidarr, Sonarr, Radarr, and Prowlarr while troubleshooting authentication, download paths, and indexer proxy behavior.

A central architectural benefit was the separation of VPN connectivity from the download application.

This allowed qBittorrent to use the VPN gateway without requiring the entire NAS or its other containers to use the same tunnel.

The work demonstrates knowledge of Linux networking, Docker Compose, network namespaces, VPN tunneling, firewall isolation, application integration, and infrastructure troubleshooting.

---

## Technical References

- [Gluetun — Official Repository](https://github.com/qdm12/gluetun)
- [Gluetun — Configuration Documentation](https://github.com/qdm12/gluetun-wiki)
- [qBittorrent — Official Website](https://www.qbittorrent.org/)
- [qBittorrent — Official Repository](https://github.com/qbittorrent/qBittorrent)
- [Docker — Networking Documentation](https://docs.docker.com/engine/network/)
- [Docker Compose — Network Configuration](https://docs.docker.com/compose/how-tos/networking/)
- [LinuxServer.io — qBittorrent Container](https://docs.linuxserver.io/images/docker-qbittorrent/)
- [Private Internet Access](https://www.privateinternetaccess.com/)
- [Proton VPN — Manual Configuration](https://protonvpn.com/support/)
- [Prowlarr — Official Repository](https://github.com/Prowlarr/Prowlarr)
- [FlareSolverr — Official Repository](https://github.com/FlareSolverr/FlareSolverr)
- [Sonarr — Official Website](https://sonarr.tv/)
- [Radarr — Official Website](https://radarr.video/)
- [Lidarr — Official Website](https://lidarr.audio/)
- [Unraid — Official Documentation](https://docs.unraid.net/)
