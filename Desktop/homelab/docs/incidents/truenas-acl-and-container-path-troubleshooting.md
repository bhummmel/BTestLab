
# TrueNAS ACL and Container Path Troubleshooting

## Overview

The TrueNAS ACL and Container Path Troubleshooting project focused on diagnosing filesystem permission problems, container storage mappings and application data access within a TrueNAS SCALE environment.

The project was associated with the homelab's earlier TrueNAS SCALE deployment, which combined ZFS-based network storage with self-hosted applications.

The primary objective was to understand why containerized applications could encounter permission errors, fail to access mounted datasets or write files to unexpected locations despite apparently correct storage configurations.

TrueNAS SCALE introduces several layers of access control between a physical storage device and an application running inside a container.

These include ZFS datasets, filesystem permissions, access control lists (ACLs), user and group identities, container volume mappings and application-specific directory configurations.

A problem at any of these layers can prevent an otherwise functioning application from accessing its required data.

The investigation was particularly relevant to applications that rely on shared storage, including media servers, automated download managers and services with persistent configuration directories.

The project examined how to distinguish filesystem permission failures from incorrect container paths, missing mounts and application configuration problems.

The broader homelab subsequently migrated from TrueNAS SCALE to Unraid, where container storage configuration continued to be an important administrative responsibility.

The available project history establishes prior TrueNAS container and storage troubleshooting but does not preserve the exact ACL entries, dataset names or error messages from this particular incident.

This document therefore presents the technical investigation and recovery methodology without claiming unverified configuration changes or specific repair outcomes.

---

## Objectives

The primary objective was to troubleshoot access problems between TrueNAS SCALE datasets and containerized applications.

Specific goals included:

- Understanding ZFS dataset permissions.
- Investigating filesystem access control lists.
- Distinguishing POSIX permissions from NFSv4 ACLs.
- Identifying container user and group identities.
- Understanding host-to-container path mappings.
- Diagnosing read and write permission failures.
- Troubleshooting incorrect mount paths.
- Evaluating dataset ownership and inheritance.
- Understanding application data persistence.
- Investigating permissions across shared media directories.
- Avoiding unnecessarily permissive filesystem configurations.
- Developing repeatable access verification procedures.
- Improving storage organization for containerized services.
- Applying lessons learned to the subsequent Unraid environment.

An additional objective was to understand how application configuration, filesystem access and container isolation interact.

---

## Infrastructure Environment

The project was associated with the homelab's previous TrueNAS SCALE platform.

TrueNAS SCALE used OpenZFS for storage management and provided application hosting capabilities.

The environment supported multiple services that depended on persistent filesystem access.

| Component | Role |
|---|---|
| TrueNAS SCALE | NAS operating system and application platform |
| OpenZFS | Dataset and filesystem management |
| ZFS datasets | Persistent storage organization |
| ACLs | Filesystem access control |
| Linux users and groups | Process identity and permission enforcement |
| Container runtime | Application isolation |
| Host path mounts | Access to NAS datasets |
| Jellyfin | Media library access |
| Sonarr | Television media management |
| Radarr | Movie media management |
| Download clients | Download and file import workflows |
| SMB | Network file sharing |
| Unraid | Subsequent NAS and container platform |

The exact application versions and TrueNAS SCALE release used during the original incident are not established.

This distinction matters because TrueNAS SCALE changed its application architecture across major releases.

Earlier versions used Kubernetes-based application management, while later versions introduced Docker-based applications.

The underlying permission principles remain relevant, but the administrative interfaces and deployment procedures differ.

---

## System Architecture

Containerized applications access persistent storage through several layers.

A simplified architecture is shown below.

```text
                 Physical Storage
                        |
                        v
                 +-------------+
                 | ZFS Storage |
                 | Pool        |
                 +-------------+
                        |
                        v
                 +-------------+
                 | ZFS Dataset |
                 +-------------+
                        |
                        v
                 +-------------+
                 | Permissions |
                 | and ACLs    |
                 +-------------+
                        |
                        v
                 +-------------+
                 | Host Path   |
                 | Mount       |
                 +-------------+
                        |
                        v
                 +-------------+
                 | Container   |
                 | Filesystem  |
                 +-------------+
                        |
                        v
                 +-------------+
                 | Application |
                 | Process     |
                 +-------------+
```

Each layer must be configured correctly.

A container may have a valid mount but lack permission to access the underlying dataset.

Alternatively, the filesystem permissions may be correct while the application references a directory that is not mounted inside the container.

The project emphasized diagnosing these conditions separately.

---

## Understanding ZFS Datasets

ZFS datasets provide logical storage organization within a ZFS pool.

Each dataset can have its own properties, including mount points, quotas and access control behavior.

For example, a storage pool might contain datasets organized as follows:

```text
tank/
├── apps/
│   ├── jellyfin/
│   ├── sonarr/
│   └── radarr/
├── media/
│   ├── movies/
│   └── tv/
└── downloads/
    ├── complete/
    └── incomplete/
```

This is an illustrative layout rather than a reproduction of the original TrueNAS dataset structure.

The project examined how dataset boundaries affect container storage access.

A directory that appears to be a normal subdirectory may actually be a separate ZFS dataset with independent permissions and properties.

This distinction is important when troubleshooting recursive permission changes or access across multiple datasets.

---

## POSIX Permissions

Traditional Linux filesystem permissions use three basic access categories:

- Owner.
- Group.
- Other.

Each category can be granted read, write and execute permissions.

For directories, execute permission allows traversal.

A process generally requires execute permission on every parent directory in a path to access a file.

For example, a container may have read permission on a media file but still be unable to open it because an ancestor directory denies traversal.

Permissions can be inspected using:

```bash
ls -ld /mnt/tank/media
```

A more detailed inspection may include:

```bash
stat /mnt/tank/media
```

The project emphasized that checking only the final file's permissions may be insufficient.

Directory traversal permissions are equally important.

---

## Access Control Lists

Access control lists provide more granular permission management than traditional owner, group and other permissions.

TrueNAS SCALE supports ACL management for datasets.

Depending on the dataset configuration and TrueNAS version, permissions may involve POSIX ACLs or NFSv4 ACLs.

These models have different semantics and administrative tools.

NFSv4 ACLs can include detailed access permissions and inheritance behavior.

POSIX ACLs extend traditional Unix permissions with additional user and group entries.

The project emphasized identifying the dataset's ACL model before attempting changes.

Commands designed for POSIX ACLs should not automatically be assumed to provide a complete representation of NFSv4 permissions.

---

## POSIX ACL Inspection

On a filesystem using POSIX ACLs, the `getfacl` utility can display access control entries.

For example:

```bash
getfacl /mnt/tank/media
```

The output may include:

- File owner.
- Owning group.
- Owner permissions.
- Named user permissions.
- Named group permissions.
- ACL mask.
- Other permissions.
- Default ACL entries.

A POSIX ACL mask can restrict the effective permissions of named users and groups.

This means an ACL entry may appear to grant write access while the effective permissions remain more restrictive.

The investigation emphasized inspecting effective access rather than relying exclusively on the presence of an ACL entry.

For datasets using NFSv4 ACLs, the appropriate TrueNAS interface or compatible NFSv4 ACL tooling should be used instead.

---

## NFSv4 ACL Behavior

NFSv4 ACLs provide a more detailed access control model.

They can include permissions for reading data, writing data, deleting objects, modifying attributes and changing permissions.

They also support inheritance flags.

Inheritance determines how access control entries are applied to newly created files and directories.

This can be particularly important for shared media storage.

For example, a download application may create a file that a media management application must later move or rename.

If the new file inherits restrictive permissions, the second application may fail even though it can access the parent directory.

The project emphasized understanding ACL inheritance as part of multi-application storage design.

---

## Dataset Ownership

Every filesystem object has an associated owner and group.

Containerized applications often run under a specific numeric user ID and group ID.

These numeric identities are important because Linux permission enforcement generally operates on numeric IDs rather than display names.

For example, a user named `media` on the host and a user named `app` inside a container may represent the same numeric UID.

Conversely, identically named users may have different numeric identities.

The project emphasized checking actual process identities rather than assuming matching usernames imply matching permissions.

---

## Container User and Group IDs

Many container images allow administrators to specify the user and group under which the application runs.

Common environment variables include:

```text
PUID
PGID
```

These variables are supported by certain container image families, including many LinuxServer.io images.

They are not universal Docker settings.

Other containers may use a Docker `user` directive, an application-specific configuration or a fixed internal service account.

A typical LinuxServer.io configuration might resemble:

```yaml
environment:
  - PUID=1000
  - PGID=1000
```

The values must correspond to the intended filesystem access model.

The project emphasized verifying image documentation before assuming a container supports particular identity configuration variables.

---

## Inspecting Container Identity

On a Docker-based deployment, the identity used by a container process can be inspected using:

```bash
docker exec CONTAINER_NAME id
```

The command reports the identity of the process launched by `docker exec`, which may differ from that of the main application if the container uses privilege-dropping mechanisms.

A more specific inspection may require examining the actual application process.

For example:

```bash
docker exec CONTAINER_NAME ps aux
```

The availability and output of these commands depend on the container image.

The project emphasized identifying the identity responsible for the failing filesystem operation.

A container's initial process running as root does not necessarily mean every application component runs with root privileges.

---

## Host Paths Versus Container Paths

One of the central troubleshooting challenges was understanding the difference between paths on the TrueNAS host and paths visible inside a container.

A host directory must be explicitly mounted into the container before the application can access it.

For example:

```text
Host Path:
/mnt/tank/media/movies

Container Path:
/movies
```

The application sees `/movies`, not necessarily the original host path.

This distinction can cause confusion when configuring applications that interact with the same files.

The project emphasized that a valid host directory does not automatically become available at the same path inside a container.

---

## Docker Bind Mount Example

A Docker Compose configuration may define a bind mount as follows:

```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    volumes:
      - /mnt/tank/media/movies:/media/movies:ro
```

This maps a directory on the host into the container.

The `:ro` suffix makes the mount read-only.

That can be appropriate for applications that only need to read media files.

However, applications responsible for moving, renaming or deleting files require appropriate write access.

The project emphasized granting access based on each application's actual responsibilities.

---

## Read-Only Versus Read-Write Mounts

Container storage mounts can be configured with different access modes.

A read-only mount prevents the container from modifying the mounted data through that mount.

A read-write mount permits modifications when the underlying filesystem permissions also allow them.

This distinction is important because container mount permissions and filesystem ACLs are separate controls.

A read-write mount does not override a restrictive filesystem ACL.

Similarly, a permissive filesystem ACL does not override a read-only container mount.

The project emphasized verifying both layers when troubleshooting write failures.

---

## Incorrect Container Paths

An application can fail even when its container has the correct filesystem permissions.

For example, consider the following configuration:

```text
Host:
  /mnt/tank/media/movies

Container:
  /media/movies

Application configured path:
  /mnt/tank/media/movies
```

The application is attempting to access the host path rather than the path exposed inside its container.

Unless that same path also exists inside the container, the application may report that the directory does not exist.

This is a path mapping problem rather than an ACL problem.

The project emphasized checking the application's configured path against the container's actual mount points before changing permissions.

---

## Diagnosing Mount Configuration

On Docker-based systems, container mount information can be inspected using:

```bash
docker inspect CONTAINER_NAME
```

A more focused command is:

```bash
docker inspect \
  --format '{{json .Mounts}}' \
  CONTAINER_NAME
```

This displays the container's configured mounts.

The output can help verify:

- Host source path.
- Container destination path.
- Mount type.
- Read-only status.
- Mount configuration.

For older Kubernetes-based TrueNAS SCALE application environments, equivalent information must be obtained through the supported Kubernetes or TrueNAS application tools.

The project emphasized matching diagnostic commands to the installed application platform.

---

## Path Verification Inside Containers

A useful troubleshooting technique is checking whether the expected directory exists inside the container.

For example:

```bash
docker exec CONTAINER_NAME ls -la /media
```

The output helps determine whether the expected mount is present.

A missing directory suggests a mount configuration or application path problem.

A directory that exists but cannot be opened may indicate a permission problem.

A directory that exists but contains unexpected files may indicate that the wrong host path was mounted.

The project emphasized distinguishing these cases before modifying dataset ACLs.

---

## Media Automation Workflows

The broader homelab includes automated media management services such as Sonarr, Radarr and download clients.

These applications frequently interact with the same filesystem data.

A typical workflow involves:

1. A download client writes a completed file.
2. A media manager identifies the completed download.
3. The media manager imports the file.
4. The file is moved, copied or hardlinked into the media library.
5. Jellyfin indexes the resulting media.

Each application must have access to the appropriate directories.

However, access alone is not sufficient.

The applications must also agree on the paths used to identify the files.

---

## Consistent Container Path Design

A common architectural improvement is exposing shared storage through consistent paths.

For example:

```text
Host Storage
/mnt/tank/data/
├── downloads/
│   ├── movies/
│   └── tv/
└── media/
    ├── movies/
    └── tv/
```

Containers could use:

```text
/data/downloads
/data/media
```

This allows different applications to refer to the same underlying directory structure.

A simplified configuration might resemble:

```yaml
volumes:
  - /mnt/tank/data:/data
```

The project emphasized that consistent path design reduces ambiguity and simplifies troubleshooting.

It can also help preserve hardlink compatibility when source and destination directories reside on the same underlying filesystem.

---

## Hardlinks and Dataset Boundaries

Media automation applications may use hardlinks to avoid duplicating large files.

A hardlink allows multiple directory entries to reference the same underlying file data.

However, hardlinks cannot cross filesystem boundaries.

Separate ZFS datasets are separate filesystems for this purpose.

Consequently, placing downloads and media libraries in different ZFS datasets prevents hardlinks between those datasets.

Even when both datasets belong to the same ZFS pool, they remain distinct filesystem boundaries.

The project emphasized that storage layout can affect application behavior independently of permissions.

A media import operation that unexpectedly copies files instead of creating hardlinks may be caused by dataset boundaries rather than an ACL failure.

---

## Diagnosing Hardlink Compatibility

On Linux, filesystem identity can be inspected using:

```bash
stat -c '%d %n' /path/to/source /path/to/destination
```

The reported device identifiers can help determine whether two paths are on the same filesystem.

The `findmnt` utility can also identify filesystem mount boundaries:

```bash
findmnt -T /path/to/source
```

```bash
findmnt -T /path/to/destination
```

These commands help distinguish path mapping problems from filesystem topology limitations.

The project reinforced the importance of understanding storage layout before configuring media automation workflows.

---

## File Ownership After Downloads

Download clients may create files using a different user or group identity from the applications responsible for importing them.

This can produce a situation where:

- The download client writes the file successfully.
- The media manager can see the file.
- The media manager cannot rename or remove it.
- The import operation fails.

The underlying problem may involve file ownership, directory permissions or ACL inheritance.

The project emphasized examining the actual permissions of newly created files rather than only checking the parent directory.

This is particularly important when multiple containers use different service accounts.

---

## Permission Verification

Filesystem permissions can be examined using:

```bash
ls -lan /mnt/tank/data
```

The `-n` option displays numeric user and group IDs.

This is useful when comparing host ownership with container process identities.

For POSIX ACL datasets, additional information can be obtained using:

```bash
getfacl /mnt/tank/data
```

The appropriate ACL inspection method must match the dataset's configured ACL type.

The project emphasized that changing ownership recursively without understanding the ACL structure can create additional problems.

---

## Recursive Permission Changes

A common troubleshooting mistake is applying broad recursive permission changes.

For example:

```bash
chmod -R 777 /path/to/data
```

This grants extensive access and can weaken filesystem security.

It may also fail to resolve problems involving NFSv4 ACL semantics, incorrect container paths or read-only mounts.

Likewise, recursively changing ownership may interfere with other applications that depend on existing identities.

The project emphasized using targeted permission changes rather than attempting to eliminate access controls entirely.

A reliable solution should identify the required users, groups and permissions for each dataset.

---

## ACL Inheritance Troubleshooting

ACL inheritance determines which permissions newly created files and directories receive.

This is important in shared application environments.

For example, a download directory may be writable by a download client but produce files that a media manager cannot modify.

The parent directory's ACL inheritance configuration may be responsible.

A proper investigation should examine:

- Parent dataset ACL.
- Inheritance flags.
- New file ownership.
- New file ACL entries.
- Application process identity.
- Required file operations.

The project emphasized testing newly created files after modifying ACL inheritance.

Existing files may retain their previous permissions unless an explicit propagation operation is performed.

---

## SMB and Container Access

TrueNAS SCALE also provides SMB file sharing.

An SMB client accesses files using authenticated network credentials.

A local container accesses the underlying filesystem using its process identity and mount configuration.

These are different access paths.

A directory being accessible through an SMB share does not automatically mean that a container can access it.

Likewise, a container successfully reading a directory does not establish that a particular SMB user has permission to access it.

The project emphasized distinguishing network share authentication from local filesystem access.

---

## Dataset ACL Types and SMB

SMB shares commonly use ACL configurations designed to support Windows-style access control.

TrueNAS provides administrative tools for managing these permissions.

However, applications using local host path mounts may require a different access model.

The project considered the importance of designing shared datasets with both SMB users and container service accounts in mind.

A dataset should not be configured exclusively for one access method if multiple applications require access.

The preferred configuration depends on the actual users, applications and required operations.

---

## Application Data Persistence

Containerized applications require persistent storage for configuration and databases.

A container image should generally be treated as replaceable.

Application state must be stored outside the container's ephemeral writable layer.

For example:

```text
ZFS Pool
   |
   +-- apps/
       |
       +-- jellyfin/
       |
       +-- sonarr/
       |
       +-- radarr/
```

Each application can have a dedicated configuration dataset or directory.

This reduces the risk of one application modifying another application's data.

The project emphasized separating application configuration from shared media storage.

---

## Troubleshooting Application Startup Failures

A container may fail to start or initialize because it cannot access its configuration directory.

Potential symptoms include:

- Permission denied errors.
- Database initialization failures.
- Missing configuration files.
- Inability to create directories.
- Unexpected default configuration.
- Application crash loops.

These symptoms can occur even when the container image itself is functioning correctly.

The project emphasized inspecting persistent storage mounts and permissions before reinstalling applications.

Recreating a container without correcting its storage configuration may simply reproduce the same failure.

---

## Container Log Analysis

Application logs can provide evidence of storage access problems.

On Docker-based systems, logs can be inspected using:

```bash
docker logs CONTAINER_NAME
```

Common filesystem-related messages may include:

```text
Permission denied
```

```text
No such file or directory
```

```text
Read-only file system
```

These messages describe different failure conditions.

`Permission denied` suggests an access control problem.

`No such file or directory` may indicate an incorrect path or missing mount.

`Read-only file system` may indicate a read-only mount or filesystem state.

The project emphasized interpreting error messages according to their actual meaning rather than treating all storage failures as permission problems.

---

## ZFS Dataset Mount Points

ZFS datasets can have configured mount points.

These determine where the datasets appear in the host filesystem.

The following command can display dataset names and mount points:

```bash
zfs list -o name,mountpoint
```

A dataset may exist in the ZFS pool but not be mounted at the path expected by the container.

The investigation emphasized verifying actual mount points before configuring application storage.

It also considered the possibility of nested datasets and unexpected filesystem boundaries.

---

## Read-Only Dataset Conditions

A dataset may be configured as read-only.

The ZFS `readonly` property can be inspected using:

```bash
zfs get readonly DATASET_NAME
```

If a dataset is read-only, applications cannot write to it regardless of ordinary filesystem permissions.

This is distinct from a container mount configured as read-only.

The project emphasized examining both the ZFS dataset properties and container mount configuration.

A write failure can originate at either layer.

---

## Storage Capacity and Quotas

A container may report a write failure even when permissions are correct.

Potential causes include:

- Full storage pool.
- Dataset quota reached.
- Filesystem allocation limits.
- Read-only dataset state.
- Storage hardware errors.

ZFS capacity information can be inspected using:

```bash
zfs list -o name,used,available,quota,refquota
```

Pool capacity can be examined using:

```bash
zpool list
```

The project emphasized that not every failed write operation is an ACL problem.

Storage capacity and dataset properties must also be considered.

---

## Troubleshooting Methodology

The investigation can be organized into a layered diagnostic process.

### Stage 1: Identify the Application Platform

Determine whether the TrueNAS SCALE version uses Kubernetes-based or Docker-based application management.

### Stage 2: Identify the Failing Operation

Determine whether the application is attempting to read, write, rename, delete or traverse a directory.

### Stage 3: Verify the Host Dataset

Confirm that the expected ZFS dataset exists and is mounted.

### Stage 4: Inspect Dataset Properties

Check mount points, read-only status, capacity and quotas.

### Stage 5: Verify Container Mounts

Confirm that the correct host directory is exposed at the expected container path.

### Stage 6: Identify the Application Process Identity

Determine the numeric UID and GID used for filesystem operations.

### Stage 7: Inspect Permissions and ACLs

Examine the applicable ACL model and effective access.

### Stage 8: Check Directory Traversal

Verify permissions on every relevant parent directory.

### Stage 9: Test the Required Operation

Perform a controlled read or write test using the application's actual identity.

### Stage 10: Validate the Application Workflow

Confirm that the application can complete its intended operation.

This methodology helps isolate storage configuration failures without unnecessarily changing unrelated permissions.

---

## Technical Challenges

### Multiple Permission Layers

Filesystem permissions, ACLs and container mount modes must all permit the intended operation.

### Different User Identities

Container service accounts may not match host users.

### ACL Model Differences

POSIX and NFSv4 ACLs require different administrative approaches.

### Incorrect Container Paths

Applications may reference host paths that do not exist inside their containers.

### Nested ZFS Datasets

Dataset boundaries can affect permissions, mounts and hardlink behavior.

### Multi-Application Access

Download clients and media managers may require coordinated access to shared directories.

### Application Platform Changes

TrueNAS SCALE releases differ in their application management architecture.

### Persistent Storage

Incorrect mounts can cause application data to be stored in unexpected or nonpersistent locations.

These challenges informed the project's emphasis on understanding the complete storage access path.

---

## Security Considerations

Filesystem permissions are an important part of application isolation.

Granting unnecessary write access can allow one compromised application to modify unrelated data.

The project emphasized:

- Using dedicated service accounts where appropriate.
- Applying least-privilege permissions.
- Avoiding world-writable datasets.
- Restricting application configuration directories.
- Using read-only mounts when possible.
- Protecting application databases.
- Separating shared media from sensitive configuration data.
- Maintaining secure backups.
- Avoiding unnecessary privileged containers.

The evaluation reinforced that resolving an access error should not require abandoning filesystem security.

---

## Reliability and Recovery

A reliable container storage design should be understandable and reproducible.

Important practices include:

- Documenting dataset layouts.
- Recording container mount paths.
- Maintaining consistent user and group identities.
- Preserving application configuration.
- Backing up critical datasets.
- Testing access after updates.
- Avoiding broad recursive permission changes.
- Verifying newly created file permissions.
- Maintaining a rollback procedure for ACL modifications.

The project emphasized preserving existing data and access controls during troubleshooting.

A configuration change that fixes one application but breaks several others is not a complete solution.

---

## Migration to Unraid

The homelab subsequently transitioned from TrueNAS SCALE to Unraid.

This changed the storage and application management environment.

The Unraid deployment uses persistent application data under:

```text
/mnt/user/appdata/
```

The broader media storage layout includes directories such as:

```text
/entertainment/media/movies
/entertainment/media/tv
/entertainment/arrmedia/downloads/Movies
/entertainment/arrmedia/downloads/TV
```

These paths reflect the later homelab organization rather than the original TrueNAS dataset layout.

The migration provided an opportunity to reconsider container volume mappings and application data organization.

However, moving to Unraid does not eliminate the need to manage filesystem permissions or container paths.

The same underlying principles remain relevant.

---

## Storage Design Improvements

The investigation highlighted several design improvements applicable to future container deployments.

### Consistent Path Naming

Applications should use predictable internal paths.

### Shared Filesystem Layout

Applications requiring hardlinks should use compatible paths on the same filesystem.

### Dedicated Configuration Storage

Application databases and configuration files should be separated from shared media directories.

### Explicit Service Identities

Container users and groups should be documented.

### Least-Privilege Access

Applications should receive only the permissions required for their intended operations.

### Recovery Documentation

Dataset and container mount configurations should be recorded for future restoration.

These principles can simplify deployment, migration and troubleshooting.

---

## Evaluation Outcome

The project examined filesystem access problems involving TrueNAS SCALE datasets and containerized applications.

The investigation focused on ZFS permissions, ACL behavior, container identities, host path mounts and shared application storage.

It established a structured approach to distinguishing permission errors from incorrect paths, missing mounts, read-only filesystems and storage topology limitations.

The broader homelab subsequently migrated to Unraid, where persistent application storage and container path management remained important.

The available project history does not establish the exact original ACL modifications or confirm a specific final repair procedure.

The documented outcome is the troubleshooting methodology and the resulting understanding of filesystem access control within containerized NAS environments.

---

## Lessons Learned

A valid container mount does not guarantee filesystem access.

A correct filesystem ACL does not guarantee that an application is using the correct path.

Linux permission enforcement depends on process identities and effective access.

Directory traversal permissions are necessary for accessing nested files.

POSIX ACLs and NFSv4 ACLs have different semantics.

ACL inheritance can affect files created by multiple applications.

Separate ZFS datasets introduce filesystem boundaries that can prevent hardlink operations.

SMB access and local container access are distinct permission paths.

Recursive permission changes can create security and operational problems.

The project strengthened understanding of Linux filesystem permissions, ZFS dataset management, container storage, ACLs and application troubleshooting.

---

## Future Improvements

Future work may include:

- Documenting all container storage mappings.
- Standardizing internal application paths.
- Maintaining a dataset and filesystem inventory.
- Recording application UID and GID assignments.
- Establishing consistent media directory permissions.
- Testing ACL inheritance with non-production files.
- Monitoring application storage errors.
- Implementing automated configuration backups.
- Validating hardlink behavior for media imports.
- Separating application configuration from shared data.
- Maintaining a documented permissions recovery procedure.
- Testing container restoration after host migration.
- Reviewing Unraid share permissions and storage layout.
- Developing a standardized storage troubleshooting checklist.

These improvements would reduce configuration ambiguity and improve the maintainability of the homelab's application storage environment.

---

## Summary

The TrueNAS ACL and Container Path Troubleshooting project focused on diagnosing filesystem access problems within a ZFS-based NAS and container hosting environment.

The work examined POSIX permissions, NFSv4 ACLs, dataset ownership, container user identities, host path mounts and application storage configuration.

A central finding was that container access depends on several independent layers, including the underlying filesystem, dataset permissions, mount configuration and application process identity.

The investigation also highlighted the importance of consistent path design for media automation applications and the impact of ZFS dataset boundaries on hardlink operations.

The broader homelab subsequently migrated from TrueNAS SCALE to Unraid, providing an opportunity to apply these storage design principles to a different hosting platform.

Although the exact original ACL changes and recovery outcome were not established in the available project record, the work provided a structured foundation for diagnosing container storage access failures.

The project demonstrates knowledge of Linux systems administration, OpenZFS, filesystem permissions, access control lists, containerization, persistent storage, media automation and infrastructure troubleshooting.

---

## Technical References

- [TrueNAS — Official Documentation](https://www.truenas.com/docs/)
- [OpenZFS — Official Documentation](https://openzfs.github.io/openzfs-docs/)
- [Docker — Bind Mounts](https://docs.docker.com/engine/storage/bind-mounts/)
- [Docker — Volumes](https://docs.docker.com/engine/storage/volumes/)
- [LinuxServer.io — Understanding PUID and PGID](https://docs.linuxserver.io/general/understanding-puid-and-pgid/)
- [TRaSH Guides — Docker Hardlinks and Atomic Moves](https://trash-guides.info/File-and-Folder-Structure/Hardlinks-and-Instant-Moves/)
- [Unraid — Official Documentation](https://docs.unraid.net/)
