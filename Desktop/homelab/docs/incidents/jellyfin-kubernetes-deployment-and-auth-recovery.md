
# Jellyfin Kubernetes Deployment and Authentication Recovery

## Overview

The Jellyfin Kubernetes Deployment and Authentication Recovery project focused on deploying, managing, and troubleshooting a self-hosted media server within a Kubernetes-based application environment.

The project was associated with the homelab's earlier TrueNAS SCALE infrastructure, which used Kubernetes for application orchestration in certain releases.

Jellyfin was selected as a self-hosted media streaming platform capable of organizing and serving locally stored movies, television programs, and other media without requiring a commercial streaming subscription or cloud-hosted media server.

The investigation involved several interconnected technical areas, including Kubernetes application deployment, persistent storage, container networking, filesystem permissions, application configuration, and administrator authentication recovery.

A central challenge was maintaining access to Jellyfin's administrative interface while preserving existing application data and media library configuration.

In a containerized environment, authentication recovery requires understanding where Jellyfin stores its configuration and user database, how those files are mounted into the application container, and how to perform maintenance without unintentionally destroying persistent state.

The project also examined how Kubernetes introduces additional abstraction between an application and the underlying NAS filesystem.

Unlike a conventional installation, Jellyfin's configuration and media directories may be exposed through Kubernetes volumes, persistent volume claims, or host path mounts.

The broader homelab subsequently migrated from TrueNAS SCALE to Unraid, where Jellyfin continued to form part of the self-hosted media infrastructure.

The available project history establishes the use of TrueNAS SCALE, Jellyfin, Kubernetes-era application management, and later Docker-based media hosting. However, it does not preserve the exact Kubernetes manifests, original authentication failure, or verified recovery procedure.

This document therefore presents the deployment architecture and troubleshooting methodology without claiming unverified administrative actions or repair outcomes.

---

## Objectives

The primary objective was to understand and troubleshoot Jellyfin deployment and administrator access within a Kubernetes-based hosting environment.

Specific goals included:

- Deploying Jellyfin as a containerized application.
- Understanding Kubernetes application architecture.
- Configuring persistent application storage.
- Mapping NAS media directories into Jellyfin.
- Managing container filesystem permissions.
- Investigating application startup failures.
- Understanding Kubernetes Pods and Services.
- Diagnosing networking and port exposure.
- Identifying Jellyfin configuration storage.
- Understanding administrator authentication.
- Investigating login and account access failures.
- Evaluating supported password recovery methods.
- Preserving application data during recovery.
- Understanding application database dependencies.
- Developing repeatable troubleshooting procedures.
- Improving backup and recovery practices.
- Preparing for future platform migrations.

An additional objective was to separate application authentication problems from infrastructure failures such as missing storage mounts or inaccessible configuration files.

---

## Infrastructure Environment

The project was associated with the homelab's earlier TrueNAS SCALE deployment.

TrueNAS SCALE provided ZFS-based storage and, in Kubernetes-based releases, an application platform built around Kubernetes and related container technologies.

| Component | Role |
|---|---|
| TrueNAS SCALE | Original NAS and application hosting platform |
| Kubernetes | Application orchestration |
| Jellyfin | Self-hosted media server |
| OpenZFS | Underlying storage management |
| ZFS datasets | Persistent application and media storage |
| Kubernetes Pods | Application execution environment |
| Kubernetes Services | Application network exposure |
| Persistent volumes | Application data persistence |
| Host path mounts | Access to NAS media directories |
| SQLite | Jellyfin application database technology |
| Linux filesystem permissions | Storage access control |
| Unraid | Subsequent NAS platform |
| Docker | Later Jellyfin container hosting |

The homelab's media infrastructure included automated library management through Sonarr and Radarr, with download clients providing media acquisition workflows.

Jellyfin was responsible for indexing and presenting the resulting media libraries.

The exact TrueNAS SCALE release and Jellyfin chart version used during the original deployment are not established.

This matters because TrueNAS SCALE's application architecture changed significantly between releases.

---

## System Architecture

The Kubernetes-based Jellyfin deployment involved several layers between the client and the underlying media storage.

```text
                  Client Devices
                        |
                        v
               +------------------+
               | Jellyfin Web UI  |
               | / Streaming App  |
               +------------------+
                        |
                        v
               +------------------+
               | Kubernetes       |
               | Service          |
               +------------------+
                        |
                        v
               +------------------+
               | Jellyfin Pod     |
               |                  |
               | Jellyfin Server  |
               +------------------+
                        |
               +--------+--------+
               |                 |
               v                 v
        Configuration         Media Library
        Persistent Volume     Host Path Mount
               |                 |
               v                 v
        Application Data      ZFS Datasets
               |                 |
               +--------+--------+
                        |
                        v
                  TrueNAS SCALE
```

The architecture separates application execution from persistent storage.

This is important because Kubernetes Pods are replaceable.

If a Pod is recreated, its application configuration and user accounts must remain available through persistent storage.

The project emphasized that container lifecycle management and application data preservation are separate responsibilities.

---

## Understanding Kubernetes

Kubernetes is a container orchestration platform designed to manage application deployment, scheduling, networking, and lifecycle operations.

Applications are commonly represented through resources such as:

- Pods.
- Deployments.
- StatefulSets.
- Services.
- PersistentVolumeClaims.
- ConfigMaps.
- Secrets.

The exact resources used by Jellyfin depend on the deployment method.

A Helm chart may generate several Kubernetes resources automatically.

The project emphasized understanding the relationship between these resources rather than treating the application as a single container.

---

## Kubernetes Pods

A Pod is a Kubernetes workload unit containing one or more containers.

For a conventional Jellyfin deployment, the Pod runs the Jellyfin application container.

The Pod may also define:

- Environment variables.
- Container ports.
- Storage mounts.
- Security contexts.
- Resource limits.
- Health checks.
- Hardware device access.

A Pod can be terminated and recreated by Kubernetes.

The project emphasized that application data must not depend on the continued existence of a particular Pod.

Persistent storage is necessary to preserve configuration and user accounts.

---

## Kubernetes Deployments

A Deployment can manage the desired number of application replicas and coordinate updates.

For a conventional single-instance Jellyfin deployment, one replica is typically appropriate.

Jellyfin is not automatically a horizontally scalable application merely because it runs on Kubernetes.

Multiple replicas sharing the same application database and configuration directory can introduce consistency and locking problems.

The project emphasized distinguishing Kubernetes orchestration capabilities from the application architecture itself.

---

## TrueNAS SCALE Application Management

Earlier TrueNAS SCALE releases used a Kubernetes-based application system.

Applications could be installed and managed through the TrueNAS administrative interface.

The platform handled many Kubernetes details automatically.

However, troubleshooting sometimes required examining the underlying Kubernetes resources.

Depending on the TrueNAS SCALE release, administrators could encounter differences in:

- Application catalogs.
- Helm chart configuration.
- Storage definitions.
- Networking options.
- Container security contexts.
- GPU allocation.
- Application update procedures.

The project emphasized matching diagnostic procedures to the installed TrueNAS version.

Commands and configuration methods from newer Docker-based TrueNAS releases are not necessarily applicable to earlier Kubernetes deployments.

---

## Persistent Application Storage

Jellyfin stores application configuration and database files separately from the media content it indexes.

Persistent application storage may contain:

- Server configuration.
- User accounts.
- Authentication information.
- Library definitions.
- Metadata.
- Plugins.
- Application databases.
- Scheduled task configuration.

A conventional Jellyfin container commonly uses a configuration directory such as:

```text
/config
```

The actual host storage location depends on the deployment.

The project emphasized identifying the mounted configuration volume before attempting authentication recovery.

A recovery procedure performed against the wrong directory may have no effect on the running application.

---

## PersistentVolumeClaims

Kubernetes uses PersistentVolumeClaims to request persistent storage.

A claim represents a storage requirement for a workload.

The underlying PersistentVolume provides the actual storage.

A simplified architecture is:

```text
Jellyfin Pod
     |
     v
PersistentVolumeClaim
     |
     v
PersistentVolume
     |
     v
ZFS-backed Storage
```

The project emphasized understanding where persistent application data physically resides.

This is particularly important when preparing backups or recovering from a failed application update.

---

## Host Path Storage

Host path mounts expose directories from the Kubernetes node into an application container.

For example:

```text
Host:
/mnt/tank/media/movies

Container:
/media/movies
```

The paths are illustrative.

Inside Jellyfin, the media library must reference the container-visible path.

The application cannot automatically access the original TrueNAS host path unless it is mounted at that location.

The project emphasized distinguishing host filesystem paths from container filesystem paths.

---

## ZFS Dataset Permissions

TrueNAS SCALE uses ZFS datasets for storage organization.

These datasets can have independent permissions and ACL configurations.

Jellyfin requires read access to the media directories it indexes.

It also requires appropriate write access to its persistent configuration and cache directories.

A media dataset being accessible through SMB does not automatically establish that Jellyfin can access it.

The Kubernetes container runs under a process identity that must be permitted by the filesystem.

The project emphasized checking filesystem access before assuming that a library indexing failure is caused by Jellyfin itself.

---

## Container User Identity

Linux filesystem permissions depend on user and group identities.

Kubernetes containers can run with specific user and group IDs through security context configuration.

For example:

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000
```

This is an illustrative Kubernetes configuration.

The correct values depend on the container image, chart, storage driver, and dataset permissions.

Not every security context field has the same effect on every volume type.

The project emphasized verifying the actual process identity and storage permissions instead of assuming that a configured UID automatically resolves every access problem.

---

## Kubernetes Networking

Kubernetes uses Services to provide stable network access to application Pods.

A Jellyfin deployment may expose its web interface through a Service.

The application commonly uses HTTP port:

```text
8096
```

The exact external port depends on the deployment configuration.

A simplified network path is:

```text
Client
   |
   v
NAS IP Address
   |
   v
Kubernetes Service
   |
   v
Jellyfin Pod
   |
   v
Jellyfin HTTP Server
```

The project emphasized that a running Pod does not guarantee that the application is reachable from the LAN.

Service configuration, port mappings, firewall rules, and application readiness must also be verified.

---

## Inspecting Kubernetes Workloads

On a Kubernetes installation with an appropriately configured `kubectl` client, available Pods can be listed using:

```bash
kubectl get pods -A
```

Services can be listed using:

```bash
kubectl get services -A
```

Persistent volume claims can be listed using:

```bash
kubectl get pvc -A
```

These commands provide an overview of the application environment.

On appliance-managed Kubernetes deployments, administrators should use supported access methods for the installed platform.

The project emphasized inspecting the current workload state before restarting or reinstalling applications.

---

## Identifying the Jellyfin Pod

A Jellyfin Pod can be located by inspecting the namespace and workload names.

For example:

```bash
kubectl get pods -A | grep -i jellyfin
```

Once identified, the Pod can be inspected using:

```bash
kubectl describe pod POD_NAME -n NAMESPACE
```

The output may include:

- Container images.
- Volume mounts.
- Container state.
- Restart counts.
- Resource requests.
- Events.
- Startup failures.

The project emphasized using Kubernetes events to distinguish application crashes from orchestration problems.

---

## Jellyfin Container Logs

Application logs are an important source of diagnostic information.

On Kubernetes, logs can be inspected using:

```bash
kubectl logs POD_NAME -n NAMESPACE
```

For a Pod containing multiple containers, the intended container may need to be specified.

Logs may reveal:

- Application initialization.
- Database access failures.
- Configuration errors.
- Filesystem permission problems.
- Network binding errors.
- Plugin failures.
- Authentication-related events.

The project emphasized reviewing logs before modifying persistent application files.

---

## Understanding Jellyfin Authentication

Jellyfin supports local user accounts and administrator permissions.

Authentication state is maintained within the application's persistent configuration and database storage.

An administrator account is required for managing server settings, users, libraries, and other administrative functions.

A login failure can have several causes.

These include:

- Incorrect credentials.
- Account lockout.
- Corrupted application state.
- Database access problems.
- Incorrect application instance.
- Lost persistent configuration.
- Reverse proxy or client issues.

The project emphasized determining whether the failure was genuinely an authentication problem before attempting recovery.

---

## Administrator Access Recovery

Jellyfin provides account and password management features through its administrative interface.

However, recovery becomes more complicated when the administrator cannot authenticate.

The appropriate recovery method depends on:

- Jellyfin version.
- Available administrative accounts.
- Server accessibility.
- Database state.
- Existing backups.
- Supported recovery mechanisms.

The project emphasized preserving application state before performing account recovery.

Deleting configuration directories or recreating the application without understanding its persistent storage can result in loss of users, libraries, and server settings.

---

## Built-In Password Reset

Jellyfin provides password reset functionality, with behavior depending on the configured authentication method and server version.

For local accounts, a password reset request may involve a recovery file generated on the server.

The recovery process requires authorized access to the server's filesystem or another supported administrative recovery path.

The project emphasized using Jellyfin's supported password reset procedure before considering direct database modifications.

The exact recovery workflow should be verified against the installed Jellyfin version.

---

## Locating Jellyfin Configuration

Before investigating authentication state, the administrator must identify the active configuration volume.

A Kubernetes Pod description can reveal mounted volumes.

For example:

```bash
kubectl describe pod POD_NAME -n NAMESPACE
```

The container's filesystem can also be inspected through an authorized shell session.

For example:

```bash
kubectl exec -it POD_NAME -n NAMESPACE -- /bin/sh
```

The shell available inside the container depends on the image.

The project emphasized confirming that the inspected configuration directory belongs to the running Jellyfin instance.

A stale configuration directory from an earlier installation may contain different account information.

---

## Application Database Considerations

Jellyfin uses SQLite databases for application state.

These databases may contain information about users, libraries, activity, and other server functions.

The exact database structure and file organization can change between Jellyfin versions.

The project emphasized avoiding unsupported direct database modifications.

Editing authentication records without understanding the installed version's schema can damage application state or create inconsistent permissions.

Supported recovery mechanisms and verified backups are preferable.

---

## SQLite Database Integrity

SQLite stores structured application data in database files.

Depending on the application's current activity, SQLite may also use associated journal or write-ahead log files.

This matters when backing up a running application.

Copying only a database file while the application is actively writing may produce an inconsistent backup.

The project emphasized using a supported backup procedure or stopping the application before performing filesystem-level recovery operations.

A recovery attempt should begin with a consistent backup of the complete relevant application state.

---

## Authentication Versus Database Failures

An application may display a login error even when the underlying problem is not an incorrect password.

For example, if Jellyfin cannot read its database, user authentication may fail.

Similarly, if the application starts with an empty configuration directory, the expected administrator account may not exist in that instance.

The project emphasized checking:

- Application logs.
- Active configuration mounts.
- Database availability.
- User account state.
- Application version.
- Storage permissions.

This prevents unnecessary password resets when the actual problem is missing or inaccessible application data.

---

## Application Instance Confusion

Containerized applications can sometimes be recreated with different persistent storage mappings.

This may cause the application to start with a new configuration directory.

The server may then appear to have lost its users or libraries.

In reality, the original configuration may still exist elsewhere on the host.

The project emphasized verifying the active storage mapping before concluding that account data has been deleted.

This distinction is particularly important during Kubernetes application reinstalls and migrations.

---

## Kubernetes Storage Troubleshooting

Persistent storage failures can prevent Jellyfin from initializing correctly.

Potential causes include:

- Missing PersistentVolumeClaims.
- Failed volume mounts.
- Incorrect host paths.
- ZFS dataset permission problems.
- Read-only storage.
- Insufficient capacity.
- Storage driver failures.
- Application security context conflicts.

The project emphasized checking Kubernetes events and storage status before attempting application-level repairs.

A missing volume cannot be corrected through a password reset.

---

## Application Restart Behavior

Kubernetes may restart a container after an application crash.

Repeated restarts can indicate:

- Invalid configuration.
- Database errors.
- Storage access failures.
- Application incompatibility.
- Resource exhaustion.
- Failed startup checks.

The project emphasized identifying the cause of a restart loop before recreating the workload.

Restarting the application may temporarily clear some conditions, but it does not resolve persistent configuration or storage problems.

---

## Resource Allocation

Jellyfin can consume substantial CPU and memory resources during media scanning and transcoding.

Kubernetes workloads may define resource requests and limits.

If limits are too restrictive, the application may experience degraded performance or termination.

The project considered the importance of monitoring resource usage during application startup and library indexing.

However, the available project history does not establish resource exhaustion as the cause of the original authentication issue.

---

## GPU Transcoding Considerations

The broader homelab used NVIDIA GPU acceleration for media processing.

Jellyfin can use compatible hardware for video decoding and encoding.

Within Kubernetes, GPU access requires appropriate host drivers, runtime integration, and device allocation.

A container may start successfully without having access to the expected GPU.

The project emphasized separating GPU transcoding issues from authentication and application startup problems.

Hardware acceleration should be validated after basic application functionality and persistent storage access are established.

---

## Library Path Configuration

Jellyfin libraries reference paths visible inside the application container.

For example:

```text
/media/movies
/media/tv
```

These paths must correspond to mounted directories.

A host directory such as:

```text
/mnt/tank/media/movies
```

does not automatically exist inside the container.

The project emphasized checking the relationship between host paths, Kubernetes volume mounts, and Jellyfin library settings.

Incorrect path mappings can cause media libraries to appear empty even when the files remain intact on the NAS.

---

## Read-Only Media Access

Jellyfin generally needs read access to media files.

In many deployments, media libraries can be mounted read-only.

For example:

```yaml
volumeMounts:
  - name: movies
    mountPath: /media/movies
    readOnly: true
```

The corresponding volume definition must also be configured.

This example illustrates the principle rather than a complete Kubernetes manifest.

The project emphasized granting applications only the access necessary for their responsibilities.

Jellyfin's configuration and cache directories still require appropriate write access.

---

## Application Configuration Backups

Authentication recovery is significantly easier when consistent application backups are available.

A useful Jellyfin backup strategy includes:

- Application configuration.
- User databases.
- Library definitions.
- Metadata.
- Plugin configuration.
- Relevant application settings.

The media files themselves may be backed up separately.

The project emphasized documenting where application data resides and how it can be restored.

A backup that omits the active configuration volume may not preserve administrator accounts.

---

## Recovery Before Reinstallation

Reinstalling an application is sometimes considered when troubleshooting becomes difficult.

However, reinstalling Jellyfin without understanding its storage mappings can create additional problems.

Potential consequences include:

- New application configuration.
- Missing users.
- Lost library definitions.
- Repeated media scanning.
- Plugin configuration loss.
- Confusion between old and new application instances.

The project emphasized diagnosing persistent storage before reinstalling the application.

A container can often be replaced without losing application state, provided its persistent volumes are preserved and remounted correctly.

---

## Troubleshooting Methodology

The investigation can be organized into a layered diagnostic process.

### Stage 1: Identify the Deployment Platform

Determine the TrueNAS SCALE release and Kubernetes application management method.

### Stage 2: Verify Workload Status

Confirm that the Jellyfin Pod is running and not repeatedly restarting.

### Stage 3: Inspect Kubernetes Events

Check for volume mounting, scheduling, or startup failures.

### Stage 4: Review Jellyfin Logs

Identify authentication, database, configuration, or filesystem errors.

### Stage 5: Verify Persistent Storage

Confirm that the correct configuration volume is mounted.

### Stage 6: Check Filesystem Permissions

Verify that Jellyfin can read and write its application data.

### Stage 7: Confirm Application Accessibility

Test the web interface through the intended network path.

### Stage 8: Identify the Authentication Failure

Distinguish incorrect credentials from database or configuration problems.

### Stage 9: Preserve Application State

Create a consistent backup before making recovery changes.

### Stage 10: Use Supported Recovery Methods

Follow the recovery procedure appropriate to the installed Jellyfin version.

### Stage 11: Validate Administrative Access

Confirm that the intended administrator can authenticate and manage the server.

### Stage 12: Verify Media Libraries

Ensure that library paths and existing application configuration remain intact.

This methodology minimizes unnecessary changes and reduces the risk of data loss.

---

## Technical Challenges

### Kubernetes Abstraction

Application configuration is distributed across Pods, Services, volumes, and other resources.

### Persistent Storage

The active Jellyfin configuration may reside in a Kubernetes-managed volume rather than an obvious host directory.

### Authentication Recovery

Administrator recovery requires preserving application state and using version-appropriate procedures.

### SQLite Database Management

Unsupported direct database changes can damage application configuration.

### Filesystem Permissions

Incorrect dataset permissions can prevent Jellyfin from reading its configuration or media.

### Application Reinstallation

Recreating a workload with different storage mappings can make existing user accounts appear missing.

### Networking

A running Pod does not guarantee that the Jellyfin web interface is reachable.

### Platform Changes

The transition from Kubernetes-based TrueNAS SCALE applications to Docker-based hosting changed the operational tooling.

These challenges reinforced the importance of understanding the complete deployment architecture.

---

## Security Considerations

Jellyfin provides access to private media libraries and administrative server functions.

The project emphasized:

- Protecting administrator credentials.
- Restricting administrative access.
- Using supported password recovery mechanisms.
- Avoiding unnecessary database modifications.
- Preserving application backups.
- Applying least-privilege filesystem permissions.
- Limiting unnecessary container privileges.
- Securing external application access.
- Keeping Jellyfin updated.
- Reviewing plugin security.
- Protecting authentication-related configuration files.

The investigation reinforced that authentication recovery should restore authorized administrative access without weakening the application's security model.

---

## Reliability and Recovery

A reliable Jellyfin deployment should survive routine container restarts, application updates, and host maintenance.

Important practices include:

- Persistent configuration storage.
- Consistent media mounts.
- Documented application versions.
- Verified backups.
- Controlled upgrades.
- Storage permission validation.
- Application health monitoring.
- Recovery documentation.
- Testing restored application state.

The project emphasized that successful recovery requires more than restarting the container.

The server must retain its users, configuration, and media library definitions.

---

## Migration to Unraid

The homelab subsequently transitioned from TrueNAS SCALE to Unraid.

This changed the application hosting model from Kubernetes-era TrueNAS application management to Docker-based container deployment.

The later environment used persistent application storage under:

```text
/mnt/user/appdata/
```

Media storage was organized through shared directories including:

```text
/entertainment/media/movies
/entertainment/media/tv
```

These paths reflect the later homelab organization rather than the original Kubernetes storage configuration.

The migration provided an opportunity to simplify container management and make application storage mappings more explicit.

However, the same underlying principles remained important.

Jellyfin still required persistent configuration, correct filesystem permissions, accessible media directories, and a recoverable administrator account.

---

## Kubernetes Versus Docker Management

The two hosting models expose different operational interfaces.

| Function | Kubernetes-Based Deployment | Docker-Based Deployment |
|---|---|---|
| Application workload | Pod / Deployment | Container |
| Application networking | Service | Docker network and published ports |
| Persistent storage | PVC / PV / host path | Bind mount or named volume |
| Application logs | `kubectl logs` | `docker logs` |
| Container inspection | `kubectl describe` | `docker inspect` |
| Interactive shell | `kubectl exec` | `docker exec` |
| Application restart | Workload controller operations | Container restart |
| Configuration management | Chart and Kubernetes resources | Compose or container configuration |

The project emphasized that neither deployment model eliminates the need to understand the underlying application.

Kubernetes provides orchestration, but it does not automatically resolve application authentication or storage problems.

---

## Evaluation Outcome

The project examined Jellyfin deployment and administrative recovery within a Kubernetes-based TrueNAS SCALE environment.

The investigation focused on workload management, persistent storage, filesystem permissions, application networking, and authentication state.

The broader homelab subsequently migrated to Unraid, where Jellyfin continued as part of the media hosting infrastructure.

The available project history does not preserve the exact authentication failure or confirm a particular password reset or database recovery operation.

Accordingly, the document does not claim that administrator access was restored through a specific procedure.

The documented outcome is the troubleshooting methodology and the resulting understanding of application recovery in a container orchestration environment.

---

## Lessons Learned

Kubernetes Pods are replaceable, but application data must persist independently.

A running Pod does not guarantee that an application is healthy or accessible.

Persistent volume mappings are critical to preserving Jellyfin configuration.

Application authentication problems can originate from missing or inaccessible database files.

A new configuration directory can make an existing installation appear to have lost its users.

Supported password recovery methods should be preferred over direct database manipulation.

SQLite database backups must account for application activity and consistency.

Media storage and application configuration have different permission requirements.

Reinstalling an application without understanding its persistent volumes can create additional problems.

The project strengthened understanding of Kubernetes, TrueNAS SCALE, Jellyfin administration, persistent storage, Linux permissions, and application recovery.

---

## Future Improvements

Future work may include:

- Documenting the original Kubernetes deployment configuration.
- Maintaining version-specific recovery procedures.
- Implementing scheduled Jellyfin configuration backups.
- Testing restoration into an isolated environment.
- Monitoring application startup failures.
- Verifying persistent volume health.
- Standardizing media library mount paths.
- Maintaining administrator account recovery documentation.
- Monitoring storage capacity and permissions.
- Testing application upgrades before production deployment.
- Improving GPU transcoding monitoring.
- Documenting the migration from Kubernetes to Docker.
- Establishing a standardized application recovery checklist.
- Maintaining secure administrative access procedures.

These improvements would support more reliable and maintainable media hosting infrastructure.

---

## Summary

The Jellyfin Kubernetes Deployment and Authentication Recovery project focused on deploying and troubleshooting a self-hosted media server within the Kubernetes-based application environment used by earlier TrueNAS SCALE releases.

The work examined Kubernetes Pods, Services, persistent storage, ZFS dataset permissions, application configuration, SQLite databases, and administrator authentication.

A central finding was that reliable application recovery depends on understanding where persistent state resides and how the orchestration platform exposes that state to the application.

The investigation also highlighted the importance of distinguishing authentication failures from storage, database, and networking problems.

The broader homelab subsequently migrated to Unraid, where Jellyfin continued operating within a Docker-based application environment.

Although the exact original authentication error and recovery outcome were not established in the available project record, the work provided a structured foundation for diagnosing and recovering containerized applications.

The project demonstrates knowledge of Kubernetes, TrueNAS SCALE, OpenZFS, Jellyfin, Linux systems administration, persistent volumes, application authentication, SQLite, container networking, and infrastructure troubleshooting.

---

## Technical References

- [Jellyfin — Official Documentation](https://jellyfin.org/docs/)
- [Jellyfin — Docker Deployment](https://jellyfin.org/docs/general/installation/container/)
- [Jellyfin — Troubleshooting](https://jellyfin.org/docs/general/administration/troubleshooting/)
- [Jellyfin — Official Repository](https://github.com/jellyfin/jellyfin)
- [Kubernetes — Official Documentation](https://kubernetes.io/docs/)
- [Kubernetes — Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Kubernetes — Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Kubernetes — Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [TrueNAS SCALE — Official Documentation](https://www.truenas.com/docs/)
- [OpenZFS — Official Documentation](https://openzfs.github.io/openzfs-docs/)
- [SQLite — Official Documentation](https://www.sqlite.org/docs.html)
- [Unraid — Official Documentation](https://docs.unraid.net/)
