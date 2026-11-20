
# UniFi Docker Port and MongoDB Troubleshooting

## Overview

The UniFi Docker Port and MongoDB Troubleshooting project focused on diagnosing and resolving deployment problems affecting a self-hosted UniFi Network Application running in a Docker environment.

The project was part of a broader effort to integrate UniFi networking equipment into an existing homelab while retaining control over the underlying network infrastructure.

Rather than deploying a dedicated UniFi Cloud Gateway solely for network management, the objective was to operate the UniFi Network Application as a containerized service on the existing Unraid server.

This approach would allow centralized management of UniFi switches and wireless access points without replacing the existing OPNsense firewall.

The deployment introduced several technical challenges involving Docker networking, port allocation, MongoDB connectivity and device adoption.

These problems were particularly important because UniFi devices depend on specific network services to discover, communicate with and maintain their connection to the controller.

The investigation involved understanding how the UniFi application communicates with managed devices, how its database dependencies are configured and how Docker networking affects the visibility of services on the local network.

The project included hands-on troubleshooting of container port conflicts and repeated device adoption failures.

The available project history confirms these problems were encountered, but it does not establish the precise final configuration or whether every issue was permanently resolved.

---

## Objectives

The primary objective was to deploy and troubleshoot a self-hosted UniFi Network Application within the existing Unraid infrastructure.

Specific goals included:

- Hosting the UniFi Network Application in Docker.
- Understanding UniFi controller networking requirements.
- Identifying and resolving Docker port conflicts.
- Evaluating bridge and host networking.
- Configuring persistent application storage.
- Understanding MongoDB dependencies.
- Troubleshooting database connection failures.
- Investigating UniFi device discovery.
- Resolving repeated device adoption attempts.
- Integrating UniFi management with OPNsense.
- Establishing reliable controller availability.
- Developing repeatable diagnostic procedures.
- Improving the maintainability of the network management environment.

An additional objective was to gain practical experience troubleshooting applications that depend on multiple network protocols and backend services.

---

## Infrastructure Environment

The project was implemented within the existing homelab infrastructure.

The primary server used Unraid to host containerized applications, while OPNsense provided routing, firewall services and other network functions.

| Component | Role |
|---|---|
| Unraid | Primary container hosting platform |
| Docker | Application deployment and networking |
| UniFi Network Application | Centralized UniFi device management |
| MongoDB | Database backend for applicable UniFi deployments |
| OPNsense | Firewall, routing and network services |
| Kea DHCP | DHCP service within the OPNsense environment |
| Unbound | DNS resolution |
| UniFi switches | Managed network infrastructure |
| UniFi access points | Wireless network infrastructure |
| Linux command-line tools | Connectivity and diagnostic testing |

The goal was to retain OPNsense as the network gateway while using the UniFi Network Application to manage compatible UniFi equipment.

This separation allows network management to remain independent of the firewall platform.

---

## System Architecture

The proposed architecture separates routing and firewall services from UniFi device management.

OPNsense remains responsible for network traffic, while the UniFi controller provides configuration, monitoring and adoption services for UniFi devices.

```text
                    Internet
                       |
                       v
                 +-----------+
                 | OPNsense  |
                 | Firewall  |
                 +-----------+
                       |
                       v
                 Local Network
                       |
          +------------+------------+
          |            |            |
          v            v            v
     UniFi Switch   UniFi AP    Unraid Server
          |            |            |
          |            |            v
          |            |      +-------------+
          |            |      | Docker      |
          |            |      | Environment |
          |            |      +-------------+
          |            |            |
          |            |            v
          |            |      +-------------+
          |            |      | UniFi       |
          |            |      | Network App |
          |            |      +-------------+
          |            |            |
          |            |            v
          |            |      +-------------+
          |            |      | MongoDB     |
          |            |      | Database    |
          |            |      +-------------+
          |            |            |
          +------------+------------+
                 Management Traffic
```

The database connection is internal to the application infrastructure.

UniFi switches and access points communicate with the Network Application through its management interfaces rather than connecting directly to MongoDB.

This distinction is important when troubleshooting connectivity.

---

## UniFi Network Application

The UniFi Network Application provides centralized management of compatible UniFi networking devices.

Its responsibilities include:

- Device discovery and adoption.
- Network configuration.
- Wireless network management.
- Switch port configuration.
- VLAN configuration.
- Device firmware management.
- Client monitoring.
- Network statistics.
- Configuration persistence.
- Device health reporting.

Although UniFi devices can continue forwarding traffic without constant controller availability in many configurations, controller connectivity is necessary for centralized management and certain features.

The project emphasized understanding which functions depend on the controller and which continue operating independently.

---

## Docker Deployment Considerations

Docker was selected because the existing Unraid environment already used containers for multiple services.

Containerization provides isolation between applications and simplifies dependency management.

However, network management applications introduce requirements that are more complex than those of ordinary web services.

A UniFi deployment may require several TCP and UDP ports.

Some ports provide access to the administrative interface, while others support device discovery, adoption and communication.

The project demonstrated that a controller web interface loading successfully does not necessarily mean the deployment is fully functional.

Device adoption can still fail when the required management ports are unavailable or incorrectly mapped.

---

## UniFi Network Ports

Understanding UniFi's port requirements was central to troubleshooting the deployment.

Commonly used ports include:

| Port | Protocol | Function |
|---|---|---|
| 8443 | TCP | UniFi Network Application management interface in traditional self-hosted deployments |
| 8080 | TCP | Device inform and adoption communication |
| 3478 | UDP | STUN communication |
| 10001 | UDP | Layer 2 device discovery |
| 8843 | TCP | Legacy guest portal HTTPS |
| 8880 | TCP | Legacy guest portal HTTP |
| 6789 | TCP | Legacy throughput testing |
| 5514 | UDP | Remote syslog in supported configurations |

The exact requirements depend on the UniFi Network Application version, container image and enabled features.

Not every deployment requires every listed port.

The investigation emphasized verifying the requirements for the installed application rather than exposing all possible ports indiscriminately.

The most important distinction was between administrative access and device management communication.

---

## Docker Port Conflicts

One of the problems encountered during deployment involved conflicts between container port assignments and services already running on the Unraid host.

Docker cannot bind multiple services to the same host IP address, transport protocol and port combination.

For example, if another container already occupies TCP port 8080 on the same host address, a UniFi container cannot bind that same endpoint.

This can prevent the container from starting or cause individual service mappings to fail.

The problem becomes more complicated when multiple containers expose common web administration ports.

A diagnostic command on a Linux host is:

```bash
sudo ss -tulpn
```

This displays listening network sockets and their associated processes where permissions allow.

Docker's published ports can be inspected using:

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

These commands help identify whether a required port is already assigned.

The project reinforced the importance of checking the complete host networking environment before modifying container port mappings.

---

## Host Ports Versus Container Ports

Docker distinguishes between the port used by an application inside its container and the port published on the host.

For example:

```yaml
ports:
  - "8444:8443"
```

This mapping forwards connections to TCP port 8444 on the host to TCP port 8443 inside the container.

Changing the external port can resolve conflicts for certain services.

However, UniFi adoption introduces an important complication.

Managed devices must be able to contact the controller using the address and port advertised by the application.

Changing a published port does not automatically change the address or port used by UniFi devices.

The project emphasized that port remapping must be evaluated in the context of the application's actual communication protocol.

---

## Device Inform Port

TCP port 8080 is particularly important in traditional UniFi Network Application deployments.

UniFi devices use the inform mechanism to communicate with the controller.

The inform URL typically resembles:

```text
http://CONTROLLER_IP:8080/inform
```

The controller address must be reachable from the managed device.

If the controller runs inside a Docker container, the published network endpoint must correctly forward traffic to the application.

A device may initially appear in the controller but fail to complete adoption if subsequent inform communication cannot reach the correct destination.

This was particularly relevant to the repeated adoption behavior encountered during the project.

---

## Docker Bridge Networking

Docker bridge networking provides container isolation and allows selected ports to be published on the host.

A simplified architecture is shown below.

```text
                 UniFi Device
                      |
                      v
                Local Network
                      |
                      v
                Unraid Host IP
                      |
                      v
             Docker Port Mapping
                      |
                      v
              UniFi Container
                      |
                      v
               UniFi Service
```

Bridge networking can work well for UniFi when all required ports and advertised addresses are configured correctly.

However, discovery traffic may require additional attention because broadcast and multicast behavior does not always cross network boundaries transparently.

The project considered whether bridge networking was appropriate for the intended device discovery and adoption workflow.

---

## Docker Host Networking

Host networking allows a container to use the host's network namespace.

This can simplify certain network communication requirements because the application is no longer behind Docker's normal port-publishing layer.

However, host networking also removes the ability to remap conflicting ports through ordinary Docker port mappings.

If another application is already listening on a required port, the conflict must be resolved at the application or host level.

Host networking also reduces network isolation.

The evaluation therefore treated host networking as an architectural choice rather than a universal solution.

The preferred network mode depends on the existing services, UniFi requirements and desired isolation.

---

## MongoDB Architecture

Some self-hosted UniFi Network Application container images require an external MongoDB instance.

Other deployment methods may manage their database dependencies differently.

The project investigated the relationship between the UniFi application and its database.

MongoDB stores application data required for persistent operation.

Depending on the UniFi application version and deployment, this can include configuration, device information and historical records.

A typical external-database architecture is:

```text
              +------------------+
              | UniFi Network    |
              | Application      |
              +------------------+
                       |
                Database Connection
                       |
                       v
              +------------------+
              | MongoDB          |
              | Database Service |
              +------------------+
                       |
                       v
              +------------------+
              | Persistent       |
              | Storage Volume   |
              +------------------+
```

The database should remain accessible to the UniFi application without being unnecessarily exposed to the rest of the network.

---

## MongoDB Connection Requirements

A containerized UniFi application may require database configuration values such as:

- MongoDB hostname.
- Database port.
- Database name.
- Authentication username.
- Authentication password.
- Authentication database.
- Application-specific database settings.

The exact configuration variables depend on the selected container image.

For example, the LinuxServer.io UniFi Network Application image uses environment variables to define its MongoDB connection.

A mismatch between the configured database hostname and the actual Docker service name can prevent connectivity.

Similarly, incorrect credentials or an incorrectly configured authentication database can cause startup failures.

The project emphasized consulting the documentation for the specific container image rather than assuming all UniFi Docker distributions use the same configuration.

---

## Docker Service Name Resolution

Docker Compose provides internal DNS resolution for containers connected to the same user-defined network.

This allows one container to contact another using its service name.

For example, a UniFi application could connect to a MongoDB service using a hostname such as:

```text
mongodb
```

This requires both services to share a suitable Docker network.

The service name must also match the actual Compose configuration.

Using `localhost` inside the UniFi container would normally refer to the UniFi container itself, not a separate MongoDB container.

This distinction is a common source of connection failures.

The project reinforced the importance of understanding container network namespaces and internal DNS.

---

## Example Docker Compose Architecture

The following example illustrates the general relationship between a UniFi application and a separate MongoDB service.

It is a reference architecture rather than a reproduction of the original deployment.

```yaml
services:
  mongodb:
    image: mongo:7.0
    container_name: unifi-mongodb
    restart: unless-stopped
    volumes:
      - /mnt/user/appdata/unifi-mongodb:/data/db
    networks:
      - unifi-backend

  unifi-network:
    image: lscr.io/linuxserver/unifi-network-application:latest
    container_name: unifi-network
    restart: unless-stopped
    depends_on:
      - mongodb
    environment:
      - PUID=99
      - PGID=100
      - TZ=America/New_York
      - MONGO_USER=unifi
      - MONGO_PASS=REPLACE_WITH_SECURE_PASSWORD
      - MONGO_HOST=mongodb
      - MONGO_PORT=27017
      - MONGO_DBNAME=unifi
      - MONGO_AUTHSOURCE=admin
    volumes:
      - /mnt/user/appdata/unifi-network:/config
    ports:
      - "8443:8443"
      - "8080:8080"
      - "3478:3478/udp"
      - "10001:10001/udp"
    networks:
      - unifi-backend

networks:
  unifi-backend:
    driver: bridge
```

This example is intentionally incomplete as a deployable database configuration.

The MongoDB container must also be initialized with the appropriate database users and permissions.

The required initialization procedure depends on the chosen image and MongoDB version.

The UniFi Network Application version must also be checked for compatibility with the selected MongoDB release.

The project emphasized that a Compose file can be syntactically valid while still containing incompatible application dependencies.

---

## MongoDB Initialization

A database container starting successfully does not guarantee that the UniFi application can authenticate to it.

MongoDB authentication requires properly configured users and permissions.

For example, a database user may need access to the application database while authenticating through the `admin` database.

The distinction between the application database and authentication database is important.

An incorrect authentication source can produce failures even when the username and password are otherwise correct.

The project identified database initialization as a separate troubleshooting stage from basic container startup.

It also reinforced the importance of using application-specific database accounts rather than unnecessarily granting broad administrative privileges.

---

## MongoDB Version Compatibility

UniFi Network Application releases may support specific MongoDB versions.

The supported range can change as the application evolves.

A MongoDB version that works with one UniFi release may not be compatible with another.

This creates an additional maintenance requirement for containerized deployments.

Updating the UniFi application without considering its database compatibility can cause startup failures.

Likewise, updating MongoDB independently may introduce compatibility problems.

The project emphasized treating application and database versions as a coordinated dependency set.

Before upgrading either component, compatibility should be verified against the current documentation.

---

## Persistent Database Storage

MongoDB requires persistent storage to retain application data across container restarts.

Without an appropriate persistent volume, removing or recreating the database container could result in data loss.

The Unraid environment provides persistent application data directories under:

```text
/mnt/user/appdata/
```

A database deployment should map the appropriate container data directory to persistent host storage.

The project emphasized the difference between container lifecycle and data lifecycle.

Containers can be replaced during updates, but application databases must remain intact.

---

## UniFi Device Discovery

UniFi devices can use multiple mechanisms to locate a controller.

Depending on the device, network topology and firmware, discovery or adoption may involve:

- Layer 2 discovery.
- DHCP-provided controller information.
- DNS-based discovery.
- Previously configured inform addresses.
- Manual inform configuration.
- Application-assisted adoption.

These methods do not all behave identically across VLANs or routed networks.

The project considered how the existing OPNsense environment affected controller discovery.

A device on a different VLAN may not receive the same broadcast discovery traffic as a device on the controller's local subnet.

The evaluation reinforced the importance of understanding the difference between device discovery and subsequent management connectivity.

---

## Repeated Adoption Failures

One of the principal problems encountered was a UniFi device entering a repeated adoption cycle.

This behavior can occur when the controller initially discovers or contacts the device but cannot maintain the required communication path.

Potential causes include:

- Incorrect inform address.
- Unreachable TCP port 8080.
- Firewall restrictions.
- Incorrect controller hostname.
- Stale controller information.
- Device firmware incompatibility.
- Controller version mismatch.
- Interrupted provisioning.
- Conflicting network configuration.

The project emphasized determining which stage of adoption was failing.

A device appearing in the controller does not establish that provisioning and ongoing communication are functioning correctly.

---

## Inform Address Troubleshooting

For devices supporting SSH-based administration, the inform address can be inspected or changed using supported UniFi commands.

A commonly used command is:

```bash
set-inform http://CONTROLLER_IP:8080/inform
```

The controller address should be the actual reachable IP address or hostname.

In a Docker deployment, this is generally the published controller endpoint rather than the container's internal bridge address.

The command may need to be repeated during certain adoption workflows, depending on the device state and controller behavior.

However, repeatedly issuing adoption commands without correcting the underlying connectivity problem does not provide a permanent solution.

The project emphasized verifying the inform endpoint before attempting additional resets or provisioning changes.

---

## Firewall and VLAN Considerations

The existing OPNsense firewall provides network segmentation and routing.

UniFi devices located on separate VLANs may require explicit firewall rules to communicate with the controller.

The required rules depend on the actual controller features and device communication paths.

The project considered the following questions:

- Can the device reach the controller's IP address?
- Is the inform port accessible?
- Is DNS resolving the expected hostname?
- Are discovery packets crossing the intended network boundaries?
- Is return traffic permitted?
- Are the controller and device using the correct VLAN interfaces?

The evaluation emphasized that successful ICMP connectivity does not guarantee application-level connectivity.

A device may respond to ping while the required TCP or UDP services remain inaccessible.

---

## Network Connectivity Testing

Basic connectivity testing can help isolate network problems.

From a Linux system, a TCP connection can be tested using:

```bash
nc -vz CONTROLLER_IP 8080
```

This checks whether a TCP connection can be established to the specified endpoint.

A successful result does not guarantee that the UniFi inform protocol is functioning correctly.

However, a failed connection provides evidence of a network or service availability problem.

Additional testing can include DNS resolution:

```bash
getent hosts CONTROLLER_HOSTNAME
```

And route inspection:

```bash
ip route
```

The project emphasized combining connectivity tests with application logs and device state information.

---

## Docker Container Diagnostics

Docker provides tools for inspecting container status and application logs.

Running containers can be listed using:

```bash
docker ps
```

Container logs can be inspected using:

```bash
docker logs unifi-network
```

For continuous monitoring:

```bash
docker logs -f unifi-network
```

The MongoDB container can be examined separately:

```bash
docker logs unifi-mongodb
```

These commands help determine whether failures originate in the application, database or network environment.

The project reinforced the importance of checking both services rather than focusing exclusively on the UniFi web interface.

---

## Database Connectivity Diagnostics

When UniFi cannot connect to MongoDB, the investigation should distinguish between several failure categories.

### Database Container Unavailable

The MongoDB process may not be running.

### Network Resolution Failure

The UniFi container may be unable to resolve the database hostname.

### Port Connectivity Failure

The database may not be listening on the expected interface or port.

### Authentication Failure

The configured credentials or authentication source may be incorrect.

### Database Initialization Failure

The required application database user may not exist.

### Version Incompatibility

The MongoDB release may not be supported by the UniFi application.

The project emphasized identifying the specific failure before changing configuration values.

---

## Troubleshooting Methodology

The investigation used a layered approach to isolate problems.

### Stage 1: Verify Container State

Confirm that the UniFi and MongoDB containers are running.

### Stage 2: Inspect Logs

Review application and database logs for startup or connection errors.

### Stage 3: Check Port Allocation

Identify conflicts involving required host ports.

### Stage 4: Validate Docker Networking

Confirm that the application and database share the intended network.

### Stage 5: Verify MongoDB Configuration

Check database hostnames, credentials and compatibility.

### Stage 6: Test Controller Access

Confirm that the administrative interface is reachable.

### Stage 7: Validate Device Inform Connectivity

Verify that managed devices can reach the controller's advertised inform endpoint.

### Stage 8: Inspect Adoption State

Determine whether discovery, adoption or provisioning is failing.

### Stage 9: Verify OPNsense Rules

Confirm that routing and firewall policies permit required communication.

### Stage 10: Monitor Stability

Observe whether devices remain adopted after controller and device restarts.

This methodology helps prevent unrelated configuration changes from obscuring the original problem.

---

## Security Considerations

A network management application has significant administrative capabilities.

Compromise of the controller could affect the configuration of managed switches and access points.

The project therefore considered several security controls:

- Restricting access to the management interface.
- Using strong administrator credentials.
- Keeping the application updated.
- Limiting MongoDB access to the application network.
- Avoiding public exposure of database ports.
- Restricting controller access across VLANs.
- Protecting persistent configuration data.
- Maintaining secure backups.
- Separating administrative and ordinary client traffic.

The evaluation emphasized that containerization does not automatically provide complete network isolation.

Security depends on the configured network boundaries and service permissions.

---

## Reliability and Recovery

The UniFi Network Application contains configuration information needed for centralized management.

A reliable deployment should preserve this information across container recreation and system maintenance.

Important recovery components include:

- UniFi application configuration.
- MongoDB data.
- Controller backups.
- Docker Compose configuration.
- Network settings.
- Application version information.
- Device adoption state.

Application-level UniFi backups and database-level backups serve different purposes.

A database backup should be performed using a method appropriate for MongoDB to ensure consistency.

Simply copying active database files may not produce a reliable recovery point.

The project reinforced the importance of testing restoration procedures rather than assuming that persistent volumes alone provide adequate protection.

---

## Operational Monitoring

A production controller should be monitored for both application availability and device connectivity.

Relevant indicators include:

- Container uptime.
- Application startup failures.
- MongoDB availability.
- Database storage utilization.
- Controller API responsiveness.
- Device adoption status.
- Device disconnections.
- Repeated provisioning attempts.
- Network port availability.
- Unexpected container restarts.

A functioning web interface does not necessarily mean that managed devices are communicating correctly.

The project emphasized monitoring the controller and its managed devices as separate operational components.

---

## Technical Challenges Encountered

### Port Conflicts

Existing Docker services occupied network ports required by the UniFi deployment.

### Multiple Network Protocols

The controller required more than a single HTTPS interface.

### Database Dependencies

The application depended on a correctly configured MongoDB environment.

### Container Networking

Internal Docker addresses differed from the externally reachable controller endpoint.

### Device Adoption Loops

Managed devices repeatedly attempted adoption or provisioning.

### Firewall Segmentation

OPNsense policies and VLAN boundaries affected device-to-controller communication.

### Version Compatibility

The UniFi application and MongoDB needed compatible software versions.

These challenges demonstrated the complexity of hosting network management software inside a general-purpose container environment.

---

## Evaluation Outcome

The project involved deploying and troubleshooting the UniFi Network Application within the existing Unraid Docker environment.

The principal issues included container port conflicts and repeated device adoption behavior.

The investigation also examined MongoDB configuration and the relationship between application startup, database connectivity and controller availability.

The work established a structured approach to diagnosing these problems through container logs, network testing and application configuration review.

However, the available project history does not establish the precise final Docker configuration or confirm that every adoption and database issue was permanently resolved.

The documented outcome is the troubleshooting experience and the resulting understanding of UniFi's networking and database dependencies.

---

## Lessons Learned

Network management applications often require more complex networking than ordinary web applications.

A reachable administrative interface does not guarantee that device adoption will function.

Docker port remapping can resolve some conflicts but may complicate applications that advertise fixed management endpoints.

The address used by managed devices must correspond to a reachable controller endpoint.

MongoDB connectivity requires correct networking, authentication and version compatibility.

Container service names and internal DNS are important when applications depend on separate database containers.

Device discovery and device management are distinct communication processes.

Firewall rules and VLAN segmentation must be evaluated in the context of actual application traffic.

The project strengthened practical understanding of Docker networking, database dependencies, UniFi device adoption and network service troubleshooting.

---

## Future Improvements

Future development may include:

- Standardizing the UniFi Docker Compose deployment.
- Documenting all required controller ports.
- Establishing a dedicated Docker network for UniFi and MongoDB.
- Implementing consistent application and database version management.
- Maintaining automated UniFi configuration backups.
- Testing MongoDB recovery procedures.
- Configuring controller health monitoring.
- Establishing reliable DNS-based controller discovery.
- Documenting UniFi device adoption procedures.
- Reviewing OPNsense firewall rules for management traffic.
- Separating UniFi management from ordinary client networks.
- Implementing alerts for controller or database failures.
- Testing controller recovery after Unraid restarts.
- Evaluating a dedicated UniFi console against the self-hosted deployment.

These improvements would help transform the troubleshooting experience into a standardized and maintainable network management deployment.

---

## Summary

The UniFi Docker Port and MongoDB Troubleshooting project focused on diagnosing deployment and connectivity problems affecting a self-hosted UniFi Network Application.

The work involved Unraid, Docker networking, MongoDB, OPNsense and UniFi device management.

The principal challenges included host port conflicts, database configuration requirements and repeated device adoption failures.

The investigation demonstrated that successful container startup and web interface access do not necessarily establish complete application functionality.

Reliable UniFi operation requires correct database connectivity, accessible management ports, appropriate controller addressing and functioning device communication.

Although the final production configuration was not fully established in the available project record, the work provided practical experience with multi-container application deployment and layered network troubleshooting.

The project demonstrates skills in Linux systems administration, Docker, MongoDB, network protocols, firewall configuration, VLAN architecture, application diagnostics and infrastructure troubleshooting.

---

## Technical References

- [Ubiquiti — Required Ports Reference](https://help.ui.com/hc/en-us/articles/218506997-Required-Ports-Reference)
- [Ubiquiti — UniFi Device Adoption](https://help.ui.com/hc/en-us/articles/360012622613-UniFi-Device-Adoption)
- [LinuxServer.io — UniFi Network Application](https://docs.linuxserver.io/images/docker-unifi-network-application/)
- [Docker — Networking Documentation](https://docs.docker.com/engine/network/)
- [MongoDB — Official Documentation](https://www.mongodb.com/docs/)
- [Unraid — Official Documentation](https://docs.unraid.net/)
