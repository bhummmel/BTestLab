# Prowlarr, FlareSolverr, and Indexer Troubleshooting

## Overview

The Prowlarr, FlareSolverr, and Indexer Troubleshooting project focused on diagnosing and improving indexer connectivity within a self-hosted media automation infrastructure running on Unraid.

The project involved integrating Prowlarr with FlareSolverr to address connectivity problems affecting certain web-based torrent indexers, while maintaining compatibility with the existing Sonarr, Radarr, Lidarr, NZBGet, and qBittorrent services.

The original media infrastructure primarily used Usenet for automated downloads, with NZBgeek serving as an established Usenet indexer and NZBGet handling downloads.

The subsequent integration of qBittorrent introduced support for BitTorrent-based acquisition, requiring additional torrent-compatible indexers and corresponding Prowlarr configuration.

During this work, FlareSolverr was installed as a Docker container and configured as an indexer proxy in Prowlarr.

The initial connection test succeeded, confirming that Prowlarr could communicate with the FlareSolverr service.

However, after saving the configuration, Prowlarr displayed the proxy as disabled.

A separate problem involved Prowlarr reporting that all configured indexer proxies were unavailable because of failures.

These symptoms required investigating the distinction between container availability, proxy configuration, indexer assignment, network connectivity, and actual indexer request processing.

A particularly important finding was that Prowlarr's FlareSolverr integration uses matching tags to determine which indexers are associated with a proxy. A successful proxy connection test does not automatically activate that proxy for indexer requests.

The investigation provided practical experience with Docker networking, HTTP APIs, indexer management, reverse-proxy behavior, application logging, and distributed service troubleshooting.

The available project history confirms the initial successful FlareSolverr connection test and the subsequent disabled-proxy and availability warnings. It does not establish that every indexer problem was permanently resolved.

---

## Objectives

The primary objective was to establish reliable indexer connectivity for the homelab's automated media management applications.

Specific goals included:

- Deploying Prowlarr as a centralized indexer manager.
- Integrating additional torrent-compatible indexers.
- Maintaining existing Usenet indexer functionality.
- Deploying FlareSolverr through Docker.
- Configuring FlareSolverr as a Prowlarr indexer proxy.
- Investigating why a successfully tested proxy appeared disabled.
- Understanding Prowlarr's tag-based proxy assignment.
- Diagnosing unavailable indexer proxy warnings.
- Verifying communication between Docker containers.
- Understanding browser-based anti-bot challenges.
- Examining indexer request timeouts.
- Reviewing Prowlarr and FlareSolverr application logs.
- Integrating indexers with Sonarr, Radarr, and Lidarr.
- Maintaining compatibility with qBittorrent and NZBGet.
- Improving reliability and fault isolation.
- Documenting a repeatable troubleshooting methodology.

An additional objective was to distinguish indexer failures from download client failures.

---

## Infrastructure Environment

The project operated within the Unraid-based homelab environment.

| Component | Role |
|---|---|
| Unraid | NAS and application hosting platform |
| Docker Engine | Container runtime |
| Docker Compose | Container orchestration and configuration |
| Prowlarr | Centralized indexer management |
| FlareSolverr | Browser-based challenge-solving proxy |
| Sonarr | Television library automation |
| Radarr | Movie library automation |
| Lidarr | Music library automation |
| NZBGet | Usenet download client |
| qBittorrent | BitTorrent download client |
| Gluetun | VPN gateway for qBittorrent |
| NZBgeek | Existing Usenet indexer |
| OPNsense | Network firewall |
| Unbound | DNS resolution |
| Jellyfin | Media library playback |

The Unraid server hosted the applications as separate Docker containers.

Prowlarr served as the central indexer management layer, reducing the need to configure identical indexers independently across multiple media management applications.

FlareSolverr was introduced as an optional supporting service for compatible indexers encountering supported browser challenges.

---

## System Architecture

The infrastructure separated indexer discovery from media acquisition and library management.

```text
                     External Indexers
                            |
               +------------+------------+
               |                         |
               v                         v
         Usenet Indexers           Torrent Indexers
               |                         |
               |                  Optional Proxy
               |                         |
               |                         v
               |                  +-------------+
               |                  | FlareSolverr |
               |                  +-------------+
               |                         ^
               |                         |
               +------------+------------+
                            |
                            v
                      +----------+
                      | Prowlarr |
                      +----------+
                            |
                +-----------+-----------+
                |           |           |
                v           v           v
              Sonarr      Radarr       Lidarr
                |           |           |
                +-----------+-----------+
                            |
                   Download Selection
                            |
                 +----------+----------+
                 |                     |
                 v                     v
              NZBGet               qBittorrent
                 |                     |
              Usenet              Gluetun VPN
                 |                     |
                 +----------+----------+
                            |
                            v
                      Unraid Storage
                            |
                            v
                         Jellyfin
```

The diagram represents logical application relationships rather than a single network route.

Prowlarr communicates with indexers and synchronizes indexer configurations to the media management applications.

FlareSolverr is used only for applicable indexer requests.

The download clients retrieve content after the media management applications select a release.

---

## Understanding Prowlarr

Prowlarr is an indexer management application within the Servarr ecosystem.

Its primary purpose is to centralize indexer configuration and synchronize supported indexers with applications such as Sonarr, Radarr, and Lidarr.

Prowlarr supports both Usenet and BitTorrent indexers.

Important capabilities include:

- Indexer configuration.
- Indexer testing.
- Application synchronization.
- Search functionality.
- Indexer statistics.
- Indexer health monitoring.
- Proxy configuration.
- Tag-based configuration.
- API integration.
- Diagnostic logging.

The project emphasized that Prowlarr is not itself a download client.

It manages access to indexers and provides search results to the applications responsible for selecting releases.

---

## Usenet Versus BitTorrent Indexers

The original media infrastructure already used NZBgeek and NZBGet.

NZBgeek provides Usenet indexing, while NZBGet retrieves content from configured Usenet servers.

The addition of qBittorrent required understanding the difference between Usenet and BitTorrent acquisition.

| Component | Usenet | BitTorrent |
|---|---|---|
| Indexer | NZBgeek or compatible Newznab service | Torrent indexer or tracker |
| Download client | NZBGet | qBittorrent |
| Download reference | NZB file | Torrent file or magnet URI |
| Transfer mechanism | Usenet server | BitTorrent peers |
| Management | Prowlarr / Servarr | Prowlarr / Servarr |

An existing Usenet indexer does not automatically provide BitTorrent search results.

The project therefore required adding and evaluating torrent-compatible indexers.

---

## Understanding FlareSolverr

FlareSolverr is an HTTP API service that uses browser automation to process certain supported browser challenges.

It is commonly deployed alongside indexer management applications.

FlareSolverr runs a browser environment and attempts to obtain a usable response and associated session information.

Prowlarr can then use that information when processing applicable indexer requests.

However, FlareSolverr is not a universal solution for every website or anti-bot mechanism.

Its compatibility depends on the challenge type, browser behavior, network identity, and the implementation of the target website.

The project emphasized treating FlareSolverr as an optional indexer compatibility component rather than a replacement for normal network troubleshooting.

---

## FlareSolverr Docker Deployment

FlareSolverr was installed as a Docker container on the Unraid server.

An illustrative Compose configuration is:

```yaml
services:
  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr

    environment:
      LOG_LEVEL: info
      TZ: America/New_York

    ports:
      - "8191:8191"

    restart: unless-stopped
```

This example represents a conventional deployment structure, not a verified copy of the original production configuration.

FlareSolverr commonly listens on port `8191`.

The published port allows applications on the host or reachable networks to communicate with the service.

For a production environment, the API should be restricted to trusted applications and networks rather than unnecessarily exposed to the internet.

---

## Docker Container Networking

The project required communication between Prowlarr and FlareSolverr.

Depending on the Docker network configuration, Prowlarr can reach FlareSolverr using a container hostname or the Unraid host's LAN address.

For example, containers on the same user-defined Docker bridge network may use:

```text
http://flaresolverr:8191
```

If Prowlarr accesses FlareSolverr through a published host port, the configuration may instead use:

```text
http://UNRAID_SERVER_IP:8191
```

The correct address depends on the actual network topology.

The project emphasized that `localhost` inside a container normally refers to that container's own network namespace.

It does not automatically refer to the Unraid host or another container.

---

## Initial FlareSolverr Configuration

After deploying FlareSolverr, the service was added to Prowlarr as an indexer proxy.

The configuration involved specifying:

- Proxy name.
- FlareSolverr host URL.
- Optional request timeout.
- Indexer proxy tags.

The connection test succeeded.

This established that Prowlarr could reach the configured FlareSolverr endpoint and receive an acceptable response to its test.

However, successful connectivity did not establish that the proxy had been assigned to any indexers.

This distinction became central to the troubleshooting process.

---

## The Disabled Proxy Problem

The most distinctive issue encountered during the project was that FlareSolverr passed its connection test but appeared disabled after saving.

At first glance, this suggested a container problem or a failure to preserve the configuration.

However, Prowlarr's proxy activation behavior depends on tag assignments.

According to the official Prowlarr documentation, a FlareSolverr proxy without matching indexer tags is disabled. The proxy is used only when Prowlarr detects an applicable Cloudflare challenge and the proxy and indexer have matching tags. <Cite ref="turn165854search0"/>

This means that a disabled proxy does not necessarily indicate that the FlareSolverr container has crashed.

The service may be running and reachable while Prowlarr has no eligible indexers assigned to use it.

---

## Tag-Based Proxy Assignment

Prowlarr uses tags to associate indexer proxies with indexers.

For example, a FlareSolverr proxy might be assigned the tag:

```text
flaresolverr
```

An indexer intended to use the proxy must have the same tag.

A simplified relationship is:

```text
             Prowlarr
                |
                v
         Indexer Configuration
                |
                | Tag: flaresolverr
                |
                v
         FlareSolverr Proxy
                |
                | Tag: flaresolverr
                |
                v
          Matching Assignment
```

If the proxy and indexer tags do not match, Prowlarr does not associate them.

The project emphasized that tags are functional configuration elements rather than merely organizational labels.

---

## Configuring Matching Tags

A conventional configuration procedure is:

1. Open Prowlarr.
2. Navigate to **Settings → Indexers**.
3. Locate the **Indexer Proxies** section.
4. Edit the FlareSolverr proxy.
5. Assign a tag such as `flaresolverr`.
6. Save the proxy configuration.
7. Open an indexer that requires the proxy.
8. Assign the same tag to that indexer.
9. Save the indexer configuration.
10. Retest the indexer and inspect the application logs.

The proxy should be associated only with indexers that require its functionality.

Prowlarr's documentation confirms that FlareSolverr requests depend on both matching tags and challenge detection. <Cite ref="turn165854search1"/>

The available project history does not establish whether this exact corrective procedure was completed during the original troubleshooting session.

---

## Why a Successful Test Was Misleading

The FlareSolverr connection test and actual indexer request processing evaluate different conditions.

A connection test verifies that Prowlarr can communicate with the proxy service.

An indexer request additionally depends on:

- Indexer configuration.
- Matching tags.
- Proxy activation rules.
- Target website availability.
- Challenge detection.
- Browser processing.
- Network connectivity.
- Request timeouts.
- Returned session information.

Consequently, a successful connection test does not guarantee that an indexer will work.

The project reinforced the importance of understanding what a diagnostic test actually verifies.

---

## Indexer Proxy Availability Warning

A separate problem involved Prowlarr displaying a warning that all indexer proxies were unavailable because of failures.

This warning required a different diagnostic approach from the disabled-proxy state.

A disabled proxy may simply have no matching indexer assignments.

An unavailable proxy may indicate that Prowlarr has encountered communication or request-processing failures.

Potential causes include:

- FlareSolverr container not running.
- Incorrect proxy URL.
- Docker networking problems.
- DNS resolution failures.
- HTTP connection timeouts.
- Browser process failures.
- Resource exhaustion.
- Unsupported browser challenges.
- Incompatible proxy and indexer network paths.
- Repeated unsuccessful requests.

The project emphasized investigating the precise warning and associated logs rather than assuming that all proxy states have the same cause.

---

## Docker Container Diagnostics

The first troubleshooting stage involved verifying container availability.

List running containers:

```bash
docker ps
```

Check FlareSolverr status:

```bash
docker ps --filter name=flaresolverr
```

Inspect recent FlareSolverr logs:

```bash
docker logs --tail 100 flaresolverr
```

Follow logs continuously:

```bash
docker logs -f flaresolverr
```

These commands help determine whether the service is running and receiving requests.

The project emphasized collecting evidence before restarting or recreating the container.

---

## Testing the FlareSolverr API

FlareSolverr exposes an HTTP API.

A basic connectivity check can be performed using:

```bash
curl -i http://UNRAID_SERVER_IP:8191/
```

The address must be adjusted to match the deployment.

This checks whether the service is reachable at the configured endpoint.

However, reaching the HTTP service does not establish that browser challenge processing is functioning correctly.

The project emphasized separating basic API availability from end-to-end functionality.

---

## Inspecting Docker Networks

Docker network configuration can be examined using:

```bash
docker network ls
```

A specific network can be inspected using:

```bash
docker network inspect NETWORK_NAME
```

Container network information can be examined using:

```bash
docker inspect prowlarr
```

```bash
docker inspect flaresolverr
```

These commands help identify whether the containers share a Docker network or must communicate through published host ports.

The project emphasized understanding the actual container topology instead of assuming that all Docker services can communicate using their container names.

---

## DNS Resolution

Prowlarr may use DNS to locate FlareSolverr or external indexer websites.

If the proxy is configured using a Docker hostname, that name must resolve from the Prowlarr container.

An example diagnostic command is:

```bash
docker exec prowlarr getent hosts flaresolverr
```

This command requires `getent` to be available in the container.

If the hostname cannot be resolved, Prowlarr may be unable to reach the service.

The project emphasized distinguishing DNS failures from HTTP request failures.

---

## HTTP Connectivity Testing

A direct request can help establish whether Prowlarr's network environment can reach FlareSolverr.

If `curl` is installed inside the Prowlarr container:

```bash
docker exec prowlarr \
  curl -i http://flaresolverr:8191/
```

The exact hostname depends on the Docker network configuration.

Some minimal container images do not include `curl`.

In that case, a temporary diagnostic container attached to the appropriate Docker network may be used.

The project emphasized avoiding unnecessary modifications to production containers solely to install diagnostic tools.

---

## FlareSolverr Request Timeouts

FlareSolverr may require more time than an ordinary HTTP client because it can launch or control a browser.

Potential sources of delay include:

- Browser initialization.
- Page loading.
- JavaScript execution.
- Network latency.
- Website challenge processing.
- Proxy latency.
- CPU contention.
- Memory pressure.

Prowlarr provides a configurable FlareSolverr request timeout.

The documented setting supports values between 1 and 180 seconds, with a default of 60 seconds. <Cite ref="turn165854search0"/>

The project emphasized that increasing a timeout may help with slow requests but will not resolve an incorrect hostname, missing proxy assignment, or unsupported challenge.

---

## Browser Resource Requirements

FlareSolverr relies on browser automation.

This can require more memory and CPU resources than a lightweight HTTP proxy.

Potential resource-related failures include:

- Browser startup failures.
- Process termination.
- Shared memory exhaustion.
- Excessive memory consumption.
- Slow challenge processing.
- Container restarts.

The project considered container resource availability as part of the diagnostic process.

However, the available project history does not establish resource exhaustion as the cause of the original warning.

---

## Container Shared Memory

Browser-based applications may benefit from an appropriately sized shared memory allocation.

An illustrative Compose configuration is:

```yaml
services:
  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr

    shm_size: "256mb"

    environment:
      LOG_LEVEL: info
      TZ: America/New_York

    restart: unless-stopped
```

The shared memory setting is an example, not a confirmed requirement of the original deployment.

The appropriate allocation depends on the browser implementation, workload, and available host resources.

---

## Prowlarr Application Logs

Prowlarr logs provide information about indexer and proxy failures.

Relevant diagnostic information may include:

- Indexer request failures.
- Proxy connection errors.
- HTTP response codes.
- Timeouts.
- DNS failures.
- Authentication errors.
- Indexer availability changes.
- Application synchronization errors.

If Prowlarr is running as a Docker container, recent logs can be examined using:

```bash
docker logs --tail 200 prowlarr
```

The project emphasized correlating Prowlarr logs with FlareSolverr logs.

An error recorded by Prowlarr may correspond to a request received by FlareSolverr at the same time.

---

## Correlating Logs Between Containers

Distributed applications often require examining multiple logs to understand one failure.

A useful troubleshooting sequence is:

1. Start following Prowlarr logs.
2. Start following FlareSolverr logs.
3. Trigger a test against the affected indexer.
4. Record the time of the request.
5. Observe whether FlareSolverr receives it.
6. Compare the responses and error messages.
7. Determine where the request fails.

This allows the administrator to distinguish several conditions.

For example:

- No request reaches FlareSolverr.
- FlareSolverr receives the request but cannot process it.
- FlareSolverr returns a response that Prowlarr cannot use.
- The indexer remains unavailable despite successful proxy processing.

The project emphasized end-to-end tracing rather than relying on one application's status indicator.

---

## Network Identity and Session Compatibility

Browser-based challenge processing may depend on the network identity used to establish a session.

If FlareSolverr and Prowlarr reach an indexer through different public IP addresses, session information may not be accepted consistently.

Differences can arise from:

- VPN routing.
- HTTP proxies.
- IPv4 versus IPv6.
- Different Docker network paths.
- Multiple WAN connections.

The FlareSolverr troubleshooting documentation specifically identifies consistent network identity between the application and FlareSolverr as an important diagnostic consideration. <Cite ref="turn165854search2"/>

The project emphasized examining the outbound network path of both containers.

---

## Relationship to Gluetun

The broader homelab used Gluetun to provide VPN isolation for qBittorrent.

This introduced an important networking distinction.

qBittorrent shared Gluetun's network namespace, while Prowlarr and FlareSolverr could operate through separate network connections.

Consequently, the public IP address used for torrent transfers was not necessarily the same as the address used for indexer searches.

The project emphasized that a VPN protecting the download client does not automatically route Prowlarr's indexer requests through the same VPN.

The exact routing arrangement of Prowlarr and FlareSolverr during the original troubleshooting session is not preserved.

---

## IPv4 and IPv6 Considerations

Some web services may behave differently depending on whether requests use IPv4 or IPv6.

A mismatch between the address families used by Prowlarr and FlareSolverr can affect session compatibility.

Potential diagnostic steps include:

- Inspecting container network configuration.
- Comparing outbound public addresses.
- Reviewing DNS resolution.
- Testing the same address family from both services.
- Examining proxy configuration.

The project emphasized that IPv6 should be evaluated as a possible variable rather than disabled indiscriminately.

Any configuration changes should be tested against the actual failure.

---

## Indexer Availability

Not every indexer failure is caused by Prowlarr or FlareSolverr.

External indexers may experience:

- Server outages.
- Domain changes.
- API changes.
- Rate limiting.
- Authentication problems.
- Network filtering.
- Certificate errors.
- Temporary maintenance.
- Unsupported challenge mechanisms.

The project emphasized checking whether a problem affects one indexer or multiple unrelated indexers.

A failure isolated to one indexer suggests a different diagnostic path from a failure affecting every configured indexer.

---

## Indexer Health Monitoring

Prowlarr maintains health information about configured indexers.

Repeated failures may cause an indexer to become temporarily unavailable.

The project emphasized examining indexer history and diagnostic messages before repeatedly restarting containers.

A failed indexer may recover when the underlying service becomes available again.

However, persistent configuration errors require correction rather than repeated retries.

---

## Integrating Prowlarr with Sonarr

Sonarr uses indexers to discover television releases.

Prowlarr can synchronize compatible indexers to Sonarr through its application integration.

The integration requires:

- Sonarr server address.
- Sonarr API key.
- Appropriate synchronization settings.
- Compatible indexer categories.
- Network connectivity.

A successful Prowlarr-to-Sonarr connection does not guarantee that every indexer is operational.

The project emphasized verifying both application synchronization and actual search results.

---

## Integrating Prowlarr with Radarr

Radarr uses indexers to discover movie releases.

Prowlarr can manage Radarr's indexer configurations centrally.

Important considerations include:

- API connectivity.
- Indexer categories.
- Synchronization behavior.
- Indexer availability.
- Download client configuration.

The project emphasized avoiding duplicate or conflicting indexer configurations across applications.

---

## Integrating Prowlarr with Lidarr

Lidarr uses indexers to discover music releases.

The broader project involved configuring Lidarr with qBittorrent and managing downloaded music through the Unraid media library.

Prowlarr provided the indexer management component.

The project emphasized that successful indexer searches and successful download client authentication are separate requirements.

An indexer may return results while Lidarr remains unable to submit a download to qBittorrent.

---

## Download Client Integration

Prowlarr can also configure download clients for its own interactive searches and grabs.

The broader media stack used:

- NZBGet for Usenet.
- qBittorrent for BitTorrent.

Each download client required its own host, port, authentication, and category configuration.

The project emphasized distinguishing three stages:

1. Discovering a release through an indexer.
2. Sending the release to a download client.
3. Importing the completed download into a media library.

Failures at one stage should not automatically be attributed to the others.

---

## Diagnostic Decision Matrix

| Symptom | Likely Diagnostic Area |
|---|---|
| FlareSolverr container will not start | Docker configuration, image, resources |
| Prowlarr cannot test FlareSolverr | Network connectivity, URL, port |
| FlareSolverr test succeeds but proxy is disabled | Missing or unmatched tags |
| Proxy is enabled but receives no requests | Indexer assignment, challenge detection |
| FlareSolverr receives requests but times out | Browser processing, network, resources |
| All indexer proxies report unavailable | Proxy failures, connectivity, logs |
| One indexer fails while others work | Indexer-specific problem |
| Prowlarr finds releases but downloads fail | Download client configuration |
| Sonarr does not receive indexers | Application synchronization |
| Indexer works manually but automated searches fail | Categories, application settings, scheduling |

The matrix provided a structured way to classify problems before changing configuration.

---

## Troubleshooting Methodology

The investigation can be organized into the following stages.

### Stage 1: Verify Container Availability

Confirm that Prowlarr and FlareSolverr are running.

### Stage 2: Verify Network Connectivity

Test communication between the containers using the configured hostname and port.

### Stage 3: Confirm Proxy Configuration

Review the FlareSolverr URL and request timeout.

### Stage 4: Verify Matching Tags

Confirm that the proxy and intended indexers share the same tag.

### Stage 5: Test an Affected Indexer

Trigger a request that exercises the relevant proxy path.

### Stage 6: Examine Prowlarr Logs

Identify the reported failure.

### Stage 7: Examine FlareSolverr Logs

Determine whether the request reached the service.

### Stage 8: Compare Network Paths

Verify whether both applications use compatible outbound network identities.

### Stage 9: Evaluate Resource Usage

Inspect browser-related memory and process failures.

### Stage 10: Test Application Synchronization

Confirm that Sonarr, Radarr, and Lidarr receive the intended indexer configurations.

### Stage 11: Test Download Client Integration

Verify that selected releases can be submitted to the appropriate download client.

### Stage 12: Monitor Stability

Observe whether indexer and proxy failures recur.

This methodology emphasizes identifying the failing component rather than changing multiple settings simultaneously.

---

## Security Considerations

Prowlarr and FlareSolverr provide access to network services and administrative APIs.

The project emphasized:

- Restricting administrative interfaces to trusted networks.
- Protecting Prowlarr API keys.
- Avoiding unnecessary public exposure of FlareSolverr.
- Keeping container images updated.
- Reviewing indexer credentials.
- Using appropriate Docker network isolation.
- Monitoring outbound network access.
- Avoiding unnecessary container privileges.
- Protecting application configuration backups.
- Using indexers and services in accordance with applicable laws and terms.

FlareSolverr should not be treated as a security control.

Its purpose is application compatibility, not network isolation or access protection.

---

## Reliability and Recovery

The project highlighted several reliability requirements.

These included:

- Persistent Prowlarr configuration.
- Automatic container restarts.
- Stable Docker networking.
- Consistent service addresses.
- Application health monitoring.
- Indexer failure detection.
- Proxy timeout management.
- Configuration backups.
- Controlled container updates.
- Log retention for troubleshooting.

The project emphasized that a healthy container does not necessarily mean the complete service chain is functional.

Reliability must be evaluated at the application level.

---

## Technical Challenges

### Successful Test but Disabled Proxy

The FlareSolverr service passed its connection test, but Prowlarr displayed it as disabled.

This highlighted the importance of tag-based proxy assignment.

### Proxy Availability Warnings

Prowlarr subsequently reported that all indexer proxies were unavailable because of failures.

This required investigating runtime behavior rather than configuration testing alone.

### Docker Networking

The services needed to communicate through correctly configured hostnames, ports, and Docker networks.

### External Indexer Behavior

Indexer availability depended on external websites outside the homelab's control.

### Browser Processing

FlareSolverr introduced additional browser-related resource and timeout considerations.

### VPN Routing

The broader Gluetun deployment introduced possible differences between the network paths used by download clients and indexer services.

### Multi-Application Integration

Prowlarr needed to communicate with several independent media management applications.

These challenges reinforced the importance of understanding both container infrastructure and application-specific behavior.

---

## Evaluation Outcome

The project successfully deployed FlareSolverr as a Docker service and established initial communication with Prowlarr.

The proxy connection test passed.

However, Prowlarr displayed the proxy as disabled after the configuration was saved.

The investigation identified Prowlarr's tag-based proxy assignment as a key configuration requirement.

A separate availability warning indicated additional proxy failures that required further investigation.

The available project history does not establish a verified permanent resolution for every indexer or proxy warning.

The documented outcome is the integration work, identification of the relevant configuration behavior, and development of a structured troubleshooting process.

---

## Lessons Learned

A successful container connection test does not guarantee that an application integration is fully configured.

Prowlarr uses matching tags to associate indexers with FlareSolverr proxies.

An unassigned FlareSolverr proxy may appear disabled even when the container is functioning correctly.

A disabled proxy and an unavailable proxy are not necessarily the same problem.

Docker container names are only useful for service discovery when the network configuration supports them.

Application logs are essential for distinguishing connectivity failures from request-processing failures.

Browser-based proxy services introduce additional resource and timeout considerations.

External indexer availability cannot be guaranteed by local infrastructure alone.

VPN routing may affect session compatibility when related services use different outbound IP addresses.

Indexer management, download submission, and completed-download import are separate stages of media automation.

The project strengthened understanding of Docker networking, Prowlarr, FlareSolverr, HTTP services, indexer management, application integration, and distributed troubleshooting.

---

## Future Improvements

Future work may include:

- Verifying matching proxy and indexer tags across the deployment.
- Documenting the final FlareSolverr configuration.
- Monitoring Prowlarr proxy health.
- Implementing alerts for repeated indexer failures.
- Reviewing FlareSolverr resource consumption.
- Comparing outbound network identities.
- Standardizing Docker service networking.
- Improving indexer request logging.
- Testing application synchronization after updates.
- Maintaining backups of Prowlarr configuration.
- Removing unreliable or unused indexers.
- Evaluating alternative supported indexer integrations.
- Documenting download client connectivity separately.
- Developing a complete media automation health checklist.

These improvements would increase the reliability and maintainability of the indexer infrastructure.

---

## Summary

The Prowlarr, FlareSolverr, and Indexer Troubleshooting project focused on improving indexer connectivity within an Unraid-based media automation environment.

The infrastructure incorporated Prowlarr, FlareSolverr, Sonarr, Radarr, Lidarr, NZBGet, and qBittorrent.

The work involved deploying FlareSolverr, configuring it as a Prowlarr indexer proxy, and investigating why the proxy appeared disabled despite passing its connection test.

A central technical finding was that Prowlarr requires matching proxy and indexer tags before the FlareSolverr integration becomes eligible for use.

The project also examined broader proxy availability failures, Docker networking, request timeouts, browser resource requirements, and application synchronization.

Although the available project history does not confirm that every indexer issue was permanently resolved, the investigation provided practical experience diagnosing multi-container application integrations.

The project demonstrates knowledge of Linux systems administration, Docker, HTTP APIs, service networking, Prowlarr, FlareSolverr, indexer management, application logging, and self-hosted infrastructure troubleshooting.

---

## Technical References

- [Prowlarr — Official Repository](https://github.com/Prowlarr/Prowlarr)
- [Prowlarr — Official Documentation](https://wiki.servarr.com/prowlarr)
- [Prowlarr — Indexer Proxy Configuration](https://wiki.servarr.com/prowlarr/settings)
- [Prowlarr — Frequently Asked Questions](https://wiki.servarr.com/prowlarr/faq)
- [FlareSolverr — Official Repository](https://github.com/FlareSolverr/FlareSolverr)
- [FlareSolverr — Troubleshooting Documentation](https://github.com/FlareSolverr/FlareSolverr/wiki/Troubleshooting)
- [Docker — Official Documentation](https://docs.docker.com/)
- [Docker — Networking](https://docs.docker.com/engine/network/)
- [LinuxServer.io — Prowlarr](https://docs.linuxserver.io/images/docker-prowlarr/)
- [Sonarr — Official Website](https://sonarr.tv/)
- [Radarr — Official Website](https://radarr.video/)
- [Lidarr — Official Website](https://lidarr.audio/)
- [NZBGet — Official Website](https://nzbget.com/)
- [qBittorrent — Official Website](https://www.qbittorrent.org/)
- [Unraid — Official Documentation](https://docs.unraid.net/)
