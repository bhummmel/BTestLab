# Dashboarr Backend and Arr Stack Management

## Overview

The Dashboarr Backend and Arr Stack Management project focused on implementing a centralized management and monitoring interface for a self-hosted media automation infrastructure running on Unraid.

The homelab hosted several interconnected media services, including Sonarr, Radarr, Lidarr, Prowlarr, qBittorrent, NZBGet, Gluetun, and Jellyfin.

Although these applications provided extensive automation capabilities, each maintained its own administrative interface, API, configuration, and monitoring mechanisms.

Managing the complete environment required switching between multiple web interfaces to inspect download activity, identify failed services, review application health, and manage media acquisition.

The project investigated Dashboarr, an open-source mobile application designed to provide a unified interface for self-hosted media services.

A central component of the implementation was the optional Dashboarr backend, which provides service polling, webhook ingestion, push notifications, device pairing, and configuration backup capabilities.

During deployment, the backend displayed a configuration warning indicating that its web interface was not enabled.

The message instructed the administrator to configure the `WEB_UI_PASSWORD` environment variable with a password containing at least eight characters and restart the container.

This introduced an additional configuration requirement beyond simply deploying the Docker image.

The project focused on enabling the backend's administrative interface, understanding its authentication model, and integrating it with the existing media automation infrastructure.

The available project history confirms that the backend was deployed far enough to display its web interface configuration warning. It does not establish that mobile pairing, push notifications, or every service integration was subsequently completed.

---

## Objectives

The primary objective was to simplify administration of the homelab's media automation stack through a centralized management interface.

Specific objectives included:

- Evaluating Dashboarr as a unified media management application.
- Deploying the Dashboarr backend on Unraid.
- Integrating the backend with existing Docker services.
- Enabling the optional administrative web interface.
- Configuring backend authentication.
- Understanding device pairing.
- Investigating service health monitoring.
- Evaluating webhook-based event notifications.
- Understanding mobile push notification delivery.
- Reviewing encrypted configuration backups.
- Evaluating centralized configuration editing.
- Understanding application API integration.
- Maintaining local network access to media services.
- Protecting API keys and administrative credentials.
- Establishing persistent backend storage.
- Developing a repeatable troubleshooting procedure.

The broader objective was to reduce administrative overhead without replacing the individual media applications.

---

## Infrastructure Environment

The project operated within the existing Unraid homelab.

| Component | Function |
|---|---|
| Unraid | NAS operating system and Docker host |
| Docker Engine | Container runtime |
| Docker Compose | Container deployment and configuration |
| Dashboarr | Mobile media management application |
| Dashboarr Backend | Service monitoring and notification backend |
| Sonarr | Television acquisition and library management |
| Radarr | Movie acquisition and library management |
| Lidarr | Music acquisition and library management |
| Prowlarr | Indexer management |
| qBittorrent | BitTorrent download client |
| Gluetun | VPN gateway for qBittorrent |
| NZBGet | Usenet download client |
| Jellyfin | Media streaming server |
| OPNsense | Network firewall and routing |
| SQLite | Backend persistent database |
| Expo Push Service | Mobile push notification delivery |

The Unraid server, `Mouseframe`, hosted the containerized applications.

The Dashboarr backend was intended to operate alongside these services rather than replace them.

---

## System Architecture

Dashboarr uses a mobile application and an optional self-hosted backend.

The mobile application communicates with configured media services, while the backend can monitor services independently and deliver notifications when the mobile application is not running.

```text
                     Mobile Device
                          |
                          v
                   +-------------+
                   | Dashboarr   |
                   | Mobile App  |
                   +-------------+
                          |
                Device Pairing / API
                          |
                          v
                   +-------------+
                   | Dashboarr   |
                   | Backend     |
                   +-------------+
                          |
             +------------+------------+
             |            |            |
             v            v            v
          Polling       Webhooks      SQLite
             |            |          Database
             |            |
             +------------+
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
            Sonarr      Radarr       Lidarr
              |           |           |
              +-----------+-----------+
                          |
                    Media Services
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
           Prowlarr    qBittorrent   NZBGet
                          |
                          v
                      Gluetun VPN

               Backend Notifications
                          |
                          v
                    Expo Push API
                          |
                          v
                     Mobile Device
```

This architecture separates the media applications from the management interface.

The backend maintains its own database and uses the APIs and webhooks of supported services to monitor activity.

---

## Why Dashboarr Was Considered

The existing media infrastructure consisted of several independently managed applications.

Each service exposed its own web interface.

For example:

- Sonarr managed television series.
- Radarr managed movies.
- Lidarr managed music.
- Prowlarr managed indexers.
- qBittorrent managed torrent transfers.
- NZBGet managed Usenet downloads.
- Jellyfin provided media playback.

Although the applications integrated with one another, their administrative interfaces remained separate.

Dashboarr offered a way to consolidate common monitoring and management functions into a single mobile interface.

The project emphasized reducing the need to open individual application dashboards for routine status checks.

---

## Understanding the Dashboarr Architecture

Dashboarr consists of two primary components.

### Mobile Application

The mobile application provides the user-facing management interface.

It can connect directly to configured media services using their APIs.

The application can operate without the optional backend.

### Self-Hosted Backend

The backend adds capabilities that require a continuously running service.

These include:

- Background service polling.
- Webhook ingestion.
- Push notification delivery.
- Device pairing.
- Configuration synchronization.
- Optional encrypted configuration backups.
- A password-protected status interface.

The project emphasized that the backend is an extension of the mobile application rather than a replacement for it.

---

## Backend Technology Stack

The Dashboarr backend is implemented using a Node.js and TypeScript application stack.

Its documented components include:

| Technology | Function |
|---|---|
| Node.js | JavaScript runtime |
| TypeScript | Application implementation |
| Fastify | HTTP API framework |
| SQLite | Persistent state storage |
| Zod | Runtime configuration and data validation |
| Docker | Containerized deployment |
| Expo | Mobile push notification transport |

The backend maintains a SQLite database at:

```text
/data/dashboarr.db
```

This database stores backend state, including information required for monitoring and device management.

Persistent storage is therefore necessary to preserve the backend's configuration across container recreation.

---

## Docker Deployment

The backend was deployed as a Docker service on the Unraid server.

The official project supports building the backend image from its source repository.

An illustrative deployment configuration is:

```yaml
services:
  dashboarr-backend:
    image: dashboarr-backend:local
    container_name: dashboarr-backend

    environment:
      NODE_ENV: production
      PORT: "4000"
      HOST: "0.0.0.0"
      TZ: America/New_York
      WEB_UI_PASSWORD: "${DASHBOARR_WEB_PASSWORD}"

    ports:
      - "4000:4000"

    volumes:
      - /mnt/user/appdata/dashboarr-backend:/data

    restart: unless-stopped
```

The example assumes that a compatible image has already been built and tagged locally as `dashboarr-backend:local`.

It is not a claim that this exact image tag or Compose configuration was used in the original deployment.

The persistent volume maps the backend's data directory to Unraid's application storage.

---

## Building the Backend Image

The Dashboarr backend repository includes a Dockerfile for building the service.

The backend build uses files from the repository root, including shared application configuration modules.

A conventional build procedure is:

```bash
git clone https://github.com/RenzoBeux/Dashboarr.git

cd Dashboarr

docker build \
  -f backend/dashboarr-backend/Dockerfile \
  -t dashboarr-backend:local .
```

The resulting image can then be deployed using Docker Compose or Unraid's container management interface.

The project emphasized verifying the repository's current build instructions before updating or rebuilding the service.

---

## Initial Configuration Issue

After deploying the backend, the web interface displayed a message indicating that the administrative UI was not enabled.

The message identified the required environment variable:

```text
WEB_UI_PASSWORD
```

It also specified that the password must contain at least eight characters.

This behavior was intentional.

The backend does not enable the authenticated administrative status interface unless a valid password has been configured.

The project required modifying the container environment and restarting the service.

---

## Enabling the Web Interface

The backend's web interface can be enabled by adding the following environment variable:

```yaml
environment:
  WEB_UI_PASSWORD: "${DASHBOARR_WEB_PASSWORD}"
```

A local `.env` file can supply the password:

```dotenv
DASHBOARR_WEB_PASSWORD=replace_with_a_unique_strong_password
```

The `.env` file should not be committed to a public Git repository.

After updating the configuration, the container must be recreated or restarted with the new environment.

For a Compose deployment:

```bash
docker compose up -d
```

The backend validates the environment variable during startup.

A missing or insufficiently long password prevents the administrative interface from being enabled.

---

## Understanding the WebUI Authentication Model

The backend's web interface uses password-based authentication.

The password is separate from the credentials used to pair mobile devices.

This separation is important because the two authentication mechanisms serve different purposes.

### Web Interface Password

Controls access to the browser-based administrative interface.

### Device Pairing Secret

Authenticates a paired mobile application when communicating with the backend.

The backend uses an authenticated session cookie for browser access.

The documented session behavior includes:

- HTTP-only cookies.
- SameSite restrictions.
- Session expiration.
- Login rate limiting.
- Session invalidation after backend restart.

The project emphasized maintaining separate administrative and device authentication boundaries.

---

## Accessing the Web Interface

The backend's default HTTP port is `4000`.

For a LAN deployment, the interface can be accessed using:

```text
http://UNRAID_SERVER_IP:4000/
```

The actual address depends on the Docker port mapping and Unraid host configuration.

After enabling the web interface, the administrator can authenticate using the configured password.

The project emphasized restricting access to trusted networks.

The backend's administrative interface should not be exposed directly to the public internet without appropriate transport security and access controls.

---

## Backend Health Monitoring

The backend exposes a health endpoint.

For current backend versions, an unauthenticated request can return a minimal liveness response.

Example:

```bash
curl http://localhost:4000/health
```

Expected response structure:

```json
{
  "ok": true,
  "name": "dashboarr-backend"
}
```

This establishes that the backend HTTP service is responding.

It does not prove that all configured media services are reachable or that push notifications are functioning.

The project emphasized distinguishing process health from complete application functionality.

---

## Administrative Status Dashboard

The optional web interface provides visibility into the backend's operational state.

Documented information includes:

- Configured service instances.
- Service names and types.
- Local and remote service URLs.
- Polling intervals.
- Last successful polling activity.
- Recent polling errors.
- Online and offline status.
- Paired mobile devices.
- Device pairing timestamps.
- Recent webhook events.
- Configuration backup metadata.

This information can help identify failures without opening the mobile application.

The project emphasized using the dashboard as an operational monitoring interface.

---

## Service Instance Management

Dashboarr supports multiple instances of the same service type.

For example, an administrator may configure:

```text
Radarr - Main Library

Radarr - Secondary Library

Sonarr - Television

Lidarr - Music
```

Each instance can maintain its own connection settings.

The backend tracks instances independently.

This allows separate monitoring and notification behavior for different deployments.

The project emphasized that the architecture can accommodate future expansion beyond a single media stack.

---

## API-Based Service Integration

Dashboarr communicates with supported applications through their APIs.

These APIs allow the application to retrieve information about service status and activity.

Typical connection parameters include:

- Service URL.
- API key.
- Authentication credentials where required.
- Local or remote endpoint.
- Service-specific settings.

For example, a Sonarr integration requires the appropriate Sonarr API endpoint and credentials.

The project emphasized that API connectivity must be validated independently for each service.

A functioning backend does not automatically guarantee that every configured application is reachable.

---

## Local Versus Remote Service Addresses

Dashboarr supports local and remote service URLs.

This is useful when the mobile application operates both inside and outside the home network.

For example:

```text
Local URL:
http://192.168.1.100:8989

Remote URL:
https://sonarr.example.com
```

These are illustrative addresses.

The backend normally uses the configured local URL for polling.

A separate backend setting can enable remote URL polling when the backend is hosted outside the local network.

The project emphasized understanding which endpoint each component uses.

An address accessible from a mobile phone is not necessarily accessible from a Docker container.

---

## Docker Networking Considerations

The backend must be able to communicate with the configured media services.

Depending on the deployment, this can be achieved through:

- Docker bridge networking.
- User-defined Docker networks.
- Host networking.
- Published service ports.
- LAN IP addresses.
- Internal DNS resolution.

For containers sharing a user-defined Docker network, service names may be used as hostnames.

For example:

```text
http://sonarr:8989
```

Alternatively, the backend may communicate through the Unraid host's LAN address.

The project emphasized avoiding assumptions about Docker DNS resolution.

A container hostname is not necessarily reachable from every Docker network.

---

## Polling Architecture

The backend periodically checks configured services.

Polling provides information about service availability and activity without requiring the mobile application to remain open.

The backend can track:

- Service reachability.
- Health status.
- Polling errors.
- Download-related state.
- Changes relevant to notifications.

Each configured instance can be monitored independently.

The project emphasized that polling provides periodic visibility rather than instantaneous event delivery.

The actual notification delay depends on the polling interval and service behavior.

---

## Webhook Integration

Webhooks provide an event-driven alternative to polling.

Instead of waiting for the backend to request updated information, a media application sends an HTTP request when a configured event occurs.

A simplified flow is:

```text
             Sonarr / Radarr
                    |
              Event Occurs
                    |
                    v
               HTTP Webhook
                    |
                    v
             Dashboarr Backend
                    |
               Event Processing
                    |
                    v
             Push Notification
                    |
                    v
               Mobile Device
```

The backend supports webhook integrations for several media applications.

Depending on the service and event type, webhooks may generate notifications or be recorded for diagnostic purposes.

---

## Webhook Authentication

The backend supports authenticated webhook ingestion.

For compatible services, authentication can be supplied through a custom HTTP header.

The documented header is:

```text
X-Dashboarr-Secret
```

Some integrations also support a secret embedded in the webhook URL path.

Header-based authentication is preferable when supported because it reduces the likelihood of credentials appearing in URL logs.

The project emphasized protecting webhook secrets and restricting access to webhook endpoints.

---

## Webhook Configuration

An illustrative Sonarr webhook endpoint is:

```text
http://UNRAID_SERVER_IP:4000/webhooks/sonarr
```

The actual URL must match the backend deployment.

The corresponding authentication secret must also be configured.

The backend can generate webhook configuration information during startup.

The official documentation describes a file containing webhook URLs and credentials:

```text
/data/webhook-urls.txt
```

This file contains sensitive information and should not be included in a public repository or troubleshooting screenshot.

---

## Mobile Device Pairing

The backend uses a pairing process to associate a mobile device with the self-hosted service.

The documented workflow involves:

1. Starting the backend.
2. Obtaining the pairing QR code or token.
3. Opening Dashboarr on the mobile device.
4. Navigating to the backend settings.
5. Scanning the pairing QR code.
6. Completing the pairing exchange.
7. Synchronizing configured services.

The backend issues a shared secret for authenticated communication.

The project emphasized that pairing credentials must be protected similarly to other administrative tokens.

The available project history does not establish that the pairing workflow was completed during the original deployment.

---

## Push Notification Architecture

The backend can deliver notifications through Expo's push notification infrastructure.

The general workflow is:

```text
Media Service
     |
     v
Dashboarr Backend
     |
     v
Expo Push Service
     |
     v
Mobile Operating System
     |
     v
Notification
```

This enables notifications to arrive even when the Dashboarr application is not actively open.

Potential events include:

- Completed downloads.
- Media acquisition activity.
- Service outages.
- New media requests.
- Other supported application events.

The project emphasized that the backend is self-hosted, but push notification delivery is not entirely local.

Expo and the mobile operating system's notification infrastructure remain external dependencies.

---

## Expo Notification Dependency

The Dashboarr backend uses Expo's push notification API.

The official backend documentation identifies a dependency on the shared Expo project's notification security configuration.

The backend sends notifications without an Expo access token under the project's documented configuration.

If the project's enhanced notification security requirements change, the backend may be unable to deliver notifications without corresponding software changes.

The project emphasized documenting external dependencies even when the primary service is self-hosted.

This distinction is important for accurately describing the system as locally managed rather than entirely offline.

---

## Configuration Backups

Dashboarr supports optional encrypted configuration backups.

These backups can preserve application configuration for restoration or migration.

The documented backup system uses:

- Passphrase-based encryption.
- AES-256-GCM.
- PBKDF2-SHA256 key derivation.
- Per-device backup slots.
- Backend storage of encrypted backup envelopes.

The backend stores the encrypted backup data without requiring the user's backup passphrase.

This separates backup storage from the ability to decrypt the backup.

The project emphasized the importance of protecting the passphrase and maintaining recoverable copies of configuration data.

---

## Browser-Based Configuration Editing

The backend includes an optional browser-based configuration editor.

This functionality allows certain application settings to be modified through the authenticated web interface.

Supported configuration areas include:

- Service instances.
- Service URLs.
- Service credentials.
- Notification settings.
- Selected integration settings.

The editor decrypts a backup in the browser using the supplied passphrase.

The passphrase is not intentionally transmitted to the backend.

However, the security of browser-based decryption depends on the integrity of the served application code and the transport connection.

The project emphasized using HTTPS when accessing the interface outside a trusted local environment.

---

## Configuration Synchronization

The backend can store configuration changes made through the web editor.

These changes are not automatically forced onto a mobile device.

The application provides a mechanism for applying the updated configuration.

This reduces the risk of silently replacing settings on an active device.

The project emphasized understanding the difference between storing a configuration backup and applying that configuration to a running application.

---

## Persistent Storage

The backend uses SQLite for persistent data.

The database is stored in the container's data directory.

A typical Unraid volume mapping is:

```yaml
volumes:
  - /mnt/user/appdata/dashboarr-backend:/data
```

This preserves the database across container recreation.

Without persistent storage, recreating the container could result in the loss of backend state.

The project emphasized treating application databases as persistent infrastructure rather than disposable container files.

---

## SQLite Backup Considerations

SQLite databases require appropriate backup handling.

Copying a database while it is actively being written may not produce a consistent backup unless the correct procedure is used.

Possible approaches include:

- Application-supported backup mechanisms.
- SQLite's online backup API.
- SQLite's `.backup` command.
- Controlled application shutdown before copying files.
- Filesystem snapshots with appropriate consistency guarantees.

An illustrative command is:

```bash
sqlite3 /path/to/dashboarr.db \
  ".backup '/path/to/dashboarr-backup.db'"
```

The command must be executed in an environment with access to the database and the SQLite command-line utility.

The project emphasized verifying backup integrity through restoration testing.

---

## Encrypting Stored Service Credentials

The backend can store credentials used to communicate with media applications.

These may include:

- API keys.
- Usernames.
- Passwords.
- Service authentication information.

The official backend supports optional encryption of stored service credentials through:

```text
CONFIG_ENCRYPTION_KEY
```

An illustrative Compose configuration is:

```yaml
environment:
  CONFIG_ENCRYPTION_KEY: "${DASHBOARR_ENCRYPTION_KEY}"
```

The encryption key should be long, unique, and stored securely.

Losing the key may make previously encrypted credentials unrecoverable.

The project emphasized that database backups and encryption key backups must be managed together.

---

## Environment Variable Management

The backend supports configuration through environment variables.

Relevant settings include:

| Variable | Purpose |
|---|---|
| `PORT` | HTTP listening port |
| `HOST` | Listening interface |
| `DATA_DIR` | Persistent data directory |
| `LOG_LEVEL` | Logging verbosity |
| `PUBLIC_URL` | Public backend address used for pairing |
| `WEB_UI_PASSWORD` | Administrative web interface authentication |
| `CONFIG_ENCRYPTION_KEY` | Encryption of stored service credentials |
| `TRUST_PROXY` | Reverse proxy header handling |
| `OFFLINE_THRESHOLD` | Service failure threshold |
| `BACKEND_USE_REMOTE` | Use remote URLs for backend polling |
| `PUSH_RECEIPTS` | Optional push receipt processing |

The project emphasized documenting configuration variables and their operational consequences.

---

## Troubleshooting the Disabled WebUI

The initial deployment problem was specifically associated with the optional web interface.

A structured diagnostic procedure is:

### Step 1: Inspect Container Status

```bash
docker ps --filter name=dashboarr-backend
```

### Step 2: Review Backend Logs

```bash
docker logs --tail 100 dashboarr-backend
```

### Step 3: Inspect Environment Configuration

```bash
docker inspect dashboarr-backend
```

Because container inspection may expose credentials, its output should not be published without redaction.

### Step 4: Verify `WEB_UI_PASSWORD`

Confirm that the environment variable exists and satisfies the minimum length requirement.

### Step 5: Recreate the Container

For Docker Compose:

```bash
docker compose up -d
```

### Step 6: Verify the Health Endpoint

```bash
curl http://localhost:4000/health
```

### Step 7: Open the Administrative Interface

Navigate to the configured backend address.

### Step 8: Authenticate

Use the configured WebUI password.

This procedure addresses the documented configuration warning.

It does not establish that the procedure was completed successfully during the original session.

---

## Troubleshooting Service Connectivity

After enabling the backend, each configured media application must be reachable.

A useful diagnostic sequence includes:

1. Verify that the target application is running.
2. Confirm its host address and port.
3. Test the service API directly.
4. Verify credentials.
5. Check Docker network connectivity.
6. Inspect backend polling errors.
7. Confirm that the correct local or remote URL is selected.

The project emphasized diagnosing connectivity independently for each application.

---

## Troubleshooting Push Notifications

Push notification delivery involves multiple systems.

Potential failure points include:

- Incorrect device pairing.
- Expired or invalid push token.
- Missing notification permissions.
- Backend service failures.
- Incorrect webhook configuration.
- Polling failures.
- Expo push delivery errors.
- Mobile operating system restrictions.

The project emphasized testing the complete notification chain rather than assuming that a healthy backend guarantees successful delivery.

---

## Reverse Proxy Considerations

The backend can operate behind a reverse proxy.

Possible reverse proxy platforms include:

- Caddy.
- Nginx.
- Nginx Proxy Manager.
- Traefik.

A reverse proxy can provide:

- HTTPS termination.
- Domain-based routing.
- Centralized certificate management.
- Access restrictions.
- Request logging.

The project emphasized using HTTPS when accessing the backend from untrusted networks.

The `PUBLIC_URL` setting should match the address used by mobile devices.

---

## Example Reverse Proxy Configuration

An illustrative Caddy configuration is:

```caddy
dashboarr.example.com {
    reverse_proxy dashboarr-backend:4000
}
```

This assumes that Caddy can resolve and reach the backend container.

The example domain is a placeholder.

The backend should be configured with the corresponding public URL when appropriate.

---

## Security Considerations

The project involved a management application with access to multiple media services.

This created several security considerations.

### Administrative Authentication

The web interface required a dedicated password.

### API Key Protection

Media application API keys needed to be stored and transmitted securely.

### Device Pairing

Mobile devices authenticated through backend-issued secrets.

### Database Protection

Persistent backend data required appropriate filesystem permissions.

### Encryption at Rest

The backend supported optional encryption of stored service credentials.

### HTTPS

Remote administrative access required secure transport.

### Network Segmentation

The backend should only be able to reach services necessary for its operation.

### Secret Management

Environment files and configuration backups needed to be excluded from public repositories.

### Update Management

Backend and mobile application versions required compatibility testing.

The project emphasized that centralizing administration also concentrates access to sensitive service credentials.

---

## Reliability Considerations

The backend was intended to operate continuously alongside the media stack.

Important reliability requirements included:

- Automatic container restarts.
- Persistent SQLite storage.
- Stable network connectivity.
- Reliable service polling.
- Webhook delivery.
- Database backups.
- Credential recovery.
- Configuration synchronization.
- Mobile pairing persistence.
- Application version compatibility.

The project emphasized that monitoring infrastructure must remain available independently of the mobile application.

---

## Technical Challenges

### Web Interface Disabled by Default

The backend required an explicit `WEB_UI_PASSWORD` setting before enabling its administrative interface.

### Container Environment Configuration

Environment variables needed to be supplied correctly through Docker or Unraid.

### Multi-Service Connectivity

The backend required access to several independently hosted applications.

### Authentication Management

Each service maintained its own API credentials.

### Persistent State

The SQLite database required durable storage.

### Mobile Pairing

Device pairing introduced a separate authentication workflow.

### External Push Infrastructure

Push notifications depended on Expo and mobile operating system services.

### Configuration Backup Security

Encrypted backups required reliable passphrase management.

### Reverse Proxy Configuration

Remote access introduced HTTPS and proxy header considerations.

These challenges reinforced the importance of understanding both application configuration and the surrounding infrastructure.

---

## Evaluation Outcome

The project evaluated Dashboarr as a centralized management interface for the existing Unraid media automation stack.

The backend was deployed sufficiently to expose its web interface configuration warning.

The immediate configuration requirement was identified as the `WEB_UI_PASSWORD` environment variable.

The investigation also established the backend's broader architecture, including service polling, webhook ingestion, mobile device pairing, push notifications, persistent SQLite storage, and optional encrypted configuration backups.

The available project history does not confirm completion of mobile pairing, successful push notification delivery, or full integration with every media application.

Accordingly, the documented outcome is the backend deployment and configuration investigation rather than a claim of a fully operational production monitoring system.

---

## Lessons Learned

Centralized media management can reduce administrative overhead in a multi-application environment.

A self-hosted management backend does not necessarily replace the individual applications it monitors.

Optional administrative interfaces may require explicit authentication configuration before becoming available.

Docker environment variables are an important part of application deployment.

A running container does not guarantee that all optional application features are enabled.

API integration requires correct credentials and network connectivity.

Persistent databases must be backed up independently of container images.

Encrypted configuration backups require secure passphrase management.

Push notification systems may depend on external infrastructure even when the backend is self-hosted.

Centralizing service credentials increases the importance of securing the management platform.

The project strengthened understanding of Docker deployment, application configuration, API integration, authentication, SQLite persistence, service monitoring, and centralized infrastructure management.

---

## Future Improvements

Future work may include:

- Confirming successful WebUI authentication.
- Pairing the mobile Dashboarr application.
- Configuring all supported media service instances.
- Validating service polling.
- Enabling webhook notifications.
- Testing mobile push delivery.
- Configuring encrypted backend credentials.
- Enabling encrypted configuration backups.
- Implementing automated SQLite backups.
- Configuring HTTPS access.
- Monitoring backend container health.
- Testing service outage notifications.
- Evaluating Apprise integration for additional notification destinations.
- Documenting mobile device migration.
- Testing disaster recovery procedures.
- Creating a unified operational dashboard for the complete media infrastructure.

These improvements would provide a more complete centralized management and monitoring solution.

---

## Summary

The Dashboarr Backend and Arr Stack Management project focused on consolidating administration and monitoring of a self-hosted media automation infrastructure.

The existing Unraid environment incorporated Sonarr, Radarr, Lidarr, Prowlarr, qBittorrent, NZBGet, Gluetun, and Jellyfin.

Dashboarr was evaluated as a unified management interface, with its optional backend providing additional monitoring, notification, and configuration capabilities.

During deployment, the backend reported that its web interface was disabled until a valid `WEB_UI_PASSWORD` environment variable was configured.

The investigation examined this authentication requirement alongside the backend's architecture, API integrations, SQLite persistence, webhook processing, mobile pairing, and encrypted configuration backup features.

The project demonstrates practical experience with containerized application deployment, environment configuration, service integration, authentication, persistent storage, and infrastructure monitoring.

Although full production integration was not established in the available project record, the work provided a foundation for centralized management of the homelab's media automation services.

---

## Technical References

- [Dashboarr — Official GitHub Repository](https://github.com/RenzoBeux/Dashboarr)
- [Dashboarr — Backend Documentation](https://github.com/RenzoBeux/Dashboarr/blob/main/backend/dashboarr-backend/README.md)
- [Dashboarr — Official Website](https://renzobeux.github.io/Dashboarr/)
- [Dashboarr — Privacy Policy](https://renzobeux.github.io/Dashboarr/privacy-policy.html)
- [Docker — Official Documentation](https://docs.docker.com/)
- [Docker Compose — Official Documentation](https://docs.docker.com/compose/)
- [Node.js — Official Documentation](https://nodejs.org/en/docs)
- [Fastify — Official Documentation](https://fastify.dev/docs/latest/)
- [SQLite — Official Documentation](https://www.sqlite.org/docs.html)
- [Expo — Push Notifications](https://docs.expo.dev/push-notifications/overview/)
- [Sonarr — Official Website](https://sonarr.tv/)
- [Radarr — Official Website](https://radarr.video/)
- [Lidarr — Official Website](https://lidarr.audio/)
- [Prowlarr — Official Repository](https://github.com/Prowlarr/Prowlarr)
- [Unraid — Official Documentation](https://docs.unraid.net/)
