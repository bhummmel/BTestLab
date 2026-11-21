
# ZFS Corruption and Scrub Recovery

## Overview

The ZFS Corruption and Scrub Recovery project focused on investigating filesystem integrity problems, understanding ZFS corruption detection and developing a controlled recovery methodology for a self-hosted network-attached storage environment.

The project was associated with the homelab's earlier TrueNAS SCALE deployment, which used OpenZFS for storage management before the infrastructure was migrated to Unraid.

The primary objective was to understand how ZFS identifies corrupted data, how its scrubbing mechanism verifies stored information and what recovery options are available when filesystem integrity problems occur.

Unlike traditional filesystems that may rely primarily on storage device error reporting, ZFS maintains checksums that allow it to detect inconsistencies between stored data and the information expected by the filesystem.

However, detecting corruption does not necessarily mean that ZFS can repair it.

Successful recovery depends on the availability of valid redundant data, the condition of the underlying storage hardware and the type of corruption encountered.

The project examined the relationship between ZFS checksums, storage redundancy, scrubbing, device errors and data recovery.

It also considered how filesystem integrity problems could affect applications, network shares and the broader homelab environment.

The available project history establishes prior TrueNAS SCALE and ZFS troubleshooting, storage reliability investigations and a subsequent migration to Unraid. It does not preserve the exact corruption reports, scrub results or recovery commands from this incident.

Accordingly, this document presents the technical investigation and recovery methodology without claiming unverified repairs or specific data-loss outcomes.

---

## Objectives

The primary objective was to investigate ZFS filesystem corruption while prioritizing data preservation.

Specific goals included:

- Understanding how ZFS detects data corruption.
- Investigating checksum errors and their possible causes.
- Evaluating the purpose and operation of ZFS scrubs.
- Distinguishing recoverable corruption from permanent data loss.
- Inspecting ZFS pool and device health.
- Identifying potential hardware-related corruption sources.
- Understanding ZFS redundancy and self-healing behavior.
- Developing non-destructive recovery procedures.
- Evaluating the impact of corruption on stored files and applications.
- Establishing reliable backup and recovery practices.
- Improving storage monitoring and maintenance procedures.
- Applying lessons learned to the subsequent NAS architecture.

An additional objective was to distinguish filesystem-level corruption from storage hardware failures and operating system instability.

---

## Infrastructure Environment

The investigation was associated with the previous TrueNAS SCALE storage environment.

TrueNAS SCALE provided Linux-based storage management using OpenZFS.

The homelab also included server-class hardware, multiple storage drives and applications that depended on persistent storage availability.

| Component | Role |
|---|---|
| TrueNAS SCALE | Previous NAS operating system |
| OpenZFS | Filesystem and storage pool management |
| Linux kernel | Operating system and storage driver environment |
| ZFS storage pools | Logical storage organization |
| Hard disk drives | Persistent data storage |
| SATA/SAS controllers | Physical storage connectivity |
| ECC memory | Server memory subsystem |
| SMART monitoring | Physical drive health diagnostics |
| Unraid | Subsequent primary NAS platform |
| Docker | Application hosting after migration |
| Network file sharing | Client access to stored files |

The storage system supported the broader homelab, making data integrity and service availability important operational concerns.

A filesystem integrity problem could potentially affect multiple applications even when the operating system remained functional.

---

## System Architecture

ZFS manages storage through a hierarchy of physical devices, virtual devices, pools and datasets.

A simplified architecture is shown below.

```text
                 Network Clients
                        |
                        v
                +---------------+
                | TrueNAS SCALE |
                | Linux Server  |
                +---------------+
                        |
                        v
                +---------------+
                | ZFS Datasets  |
                | Filesystems   |
                +---------------+
                        |
                        v
                +---------------+
                |   ZFS Pool    |
                |     zpool     |
                +---------------+
                        |
                        v
                +---------------+
                | Virtual       |
                | Devices       |
                | vdevs         |
                +---------------+
                        |
              +---------+---------+
              |         |         |
              v         v         v
            Drive     Drive     Drive
```

The exact redundancy characteristics depend on the configuration of the underlying virtual devices.

For example, a mirrored vdev provides different recovery characteristics from a RAIDZ vdev or a single-disk vdev.

This distinction is important because ZFS cannot reconstruct every corrupted block without a valid alternative copy or sufficient redundancy.

---

## ZFS Data Integrity Model

One of ZFS's defining characteristics is its end-to-end data integrity model.

ZFS stores checksums for filesystem data and metadata.

When a block is read, ZFS can verify its contents against the checksum recorded for that block.

If the checksum does not match, ZFS identifies an integrity failure.

This mechanism can detect corruption that might otherwise go unnoticed by the operating system or storage hardware.

Potential sources of corruption include:

- Storage media degradation.
- Faulty data cables.
- Controller or interface problems.
- Memory instability.
- Firmware defects.
- Power-related failures.
- Incorrect or incomplete writes.
- Other hardware or software faults.

However, a checksum mismatch identifies an inconsistency, not necessarily its original cause.

The project emphasized using ZFS integrity information as diagnostic evidence rather than assuming every checksum error indicates a failing disk.

---

## Understanding Silent Data Corruption

Silent data corruption occurs when stored information changes unexpectedly without an immediate, obvious error being reported to the application.

Traditional storage systems may successfully read a block even when its contents no longer match the data originally written.

ZFS checksums provide a mechanism for detecting these inconsistencies.

For example, a file may remain accessible through a network share while containing a block that has been altered by an underlying hardware problem.

Without an integrity mechanism, the corruption might not be discovered until the file is opened or otherwise validated.

ZFS can detect many such failures when affected blocks are read.

The project reinforced the importance of integrity verification as a separate concern from basic storage availability.

A drive responding to read requests does not guarantee that every returned block contains valid data.

---

## ZFS Scrubbing

A ZFS scrub is an integrity verification operation that examines allocated data and metadata within a storage pool.

During a scrub, ZFS reads stored blocks and verifies their checksums.

If a checksum mismatch is detected and valid redundant data is available, ZFS may repair the affected block.

A scrub therefore serves two related purposes:

1. Detecting corruption in stored data.
2. Repairing detected corruption when sufficient valid redundancy exists.

A scrub is not equivalent to a conventional filesystem repair utility.

It does not reconstruct arbitrary missing files or restore data from backups.

It also cannot guarantee recovery when all available copies of a block are corrupted.

The project emphasized understanding these limitations before interpreting scrub results.

---

## Scrub Execution

On a compatible OpenZFS system, a scrub can be initiated using:

```bash
zpool scrub POOL_NAME
```

The pool name must be replaced with the actual storage pool identifier.

Pool status can be inspected using:

```bash
zpool status
```

Additional information about detected errors can be requested using:

```bash
zpool status -v
```

These commands are useful for monitoring scrub activity and reviewing reported integrity problems.

However, initiating a scrub creates substantial storage I/O.

If the system is experiencing severe hardware instability, a scrub may increase stress on failing components.

The project therefore emphasized evaluating drive and controller health before initiating intensive storage operations during a recovery incident.

---

## Understanding Scrub Results

ZFS reports information about scrub progress and the results of completed scans.

Relevant information can include:

- Scrub start and completion times.
- Amount of data examined.
- Detected checksum errors.
- Repaired data.
- Unrecoverable errors.
- Pool health.
- Device-level error counters.

A completed scrub does not automatically establish that every stored file is recoverable.

The result must be interpreted alongside pool status and any reported permanent errors.

For example, a scrub may complete successfully while identifying corrupted data that could not be reconstructed.

Conversely, a scrub may detect and repair errors without resulting in permanent file corruption.

The evaluation emphasized distinguishing between errors detected, errors repaired and errors that remain unresolved.

---

## ZFS Error Counters

The `zpool status` command reports several categories of device errors.

These commonly include read, write and checksum errors.

| Error Type | General Meaning |
|---|---|
| READ | A device failed to complete a requested read operation |
| WRITE | A device failed to complete a requested write operation |
| CKSUM | Data returned by a device did not match the expected checksum |

These categories provide useful information but do not identify every underlying hardware failure.

For example, checksum errors can result from problems elsewhere in the storage path.

Likewise, repeated read errors may indicate media failure, controller instability or connectivity problems.

The project emphasized correlating ZFS error counters with kernel logs and physical drive diagnostics.

---

## Recoverable Versus Unrecoverable Corruption

A critical part of the investigation was understanding the distinction between recoverable and unrecoverable corruption.

### Recoverable Corruption

If ZFS detects a corrupted block and can retrieve a valid alternative copy, it may reconstruct the correct data.

This capability is often described as self-healing.

The availability of a valid copy depends on the storage topology and the type of data involved.

### Unrecoverable Corruption

If no valid copy can be found, ZFS may report permanent errors.

In this situation, the affected data cannot be reconstructed solely from the remaining pool contents.

Recovery may require restoring the affected files or datasets from an independent backup.

The project reinforced the importance of maintaining backups even when using a filesystem with advanced integrity protection.

---

## Storage Redundancy and Self-Healing

ZFS redundancy can improve the likelihood of recovering corrupted blocks.

However, different vdev layouts provide different levels of fault tolerance.

### Mirrored Vdevs

Mirrored vdevs store multiple copies of data across separate devices.

If one copy is corrupted and another remains valid, ZFS can use the valid copy for recovery.

### RAIDZ Vdevs

RAIDZ distributes data and parity information across multiple devices.

Depending on the RAIDZ level and the number of failures, ZFS may be able to reconstruct corrupted data.

### Nonredundant Vdevs

A single-disk vdev does not provide the same ability to reconstruct corrupted user data.

ZFS may still detect corruption, but detection alone cannot restore missing information.

The project emphasized that checksum verification and redundancy are complementary features.

Neither should be confused with a complete backup strategy.

---

## Pool Health Assessment

A structured recovery investigation begins with understanding the current state of the storage pool.

The following command provides a detailed status report:

```bash
zpool status -v
```

The output may identify degraded devices, active scrubs and reported data errors.

Pool capacity and allocation information can be examined using:

```bash
zpool list
```

Individual datasets can be inspected using:

```bash
zfs list
```

These commands provide different views of the storage environment.

Pool status focuses on the health of the underlying storage configuration.

Dataset information describes the filesystems and volumes managed within the pool.

The project emphasized collecting this information before making changes to the storage configuration.

---

## Permanent Error Reporting

When ZFS cannot repair corrupted data, it may report permanent errors.

Depending on the circumstances, the status output may identify affected files, objects or datasets.

This information can help determine the scope of the problem.

However, not every permanent error can be mapped directly to a convenient filesystem path.

Some errors may involve metadata or objects that require additional investigation.

The project emphasized preserving diagnostic information before deleting or replacing affected files.

Removing a damaged file may eliminate one visible symptom without resolving the underlying hardware or storage problem.

---

## Hardware Diagnostic Investigation

Filesystem corruption can originate from failures anywhere in the storage path.

A thorough investigation therefore requires hardware diagnostics.

Relevant components include:

- Hard disk drives.
- SATA or SAS data cables.
- Backplanes.
- Storage controllers.
- Power connections.
- System memory.
- Motherboard interfaces.
- Power supplies.

The project emphasized that replacing a disk without identifying the actual failure source may not resolve recurring checksum errors.

A defective cable or controller can produce problems across otherwise healthy drives.

Hardware diagnostics were therefore considered essential to determining whether corruption was an isolated event or a recurring infrastructure problem.

---

## SMART Diagnostics

SMART data provides information about the condition of supported storage devices.

On Linux, the `smartctl` utility can retrieve drive health information.

For example:

```bash
smartctl -a /dev/sdX
```

The correct device must be identified before running commands.

Relevant attributes and error information may include:

- Reallocated sectors.
- Pending sectors.
- Uncorrectable errors.
- Interface CRC errors.
- Drive temperature.
- Self-test history.
- Device-reported errors.

SMART information should be interpreted alongside ZFS diagnostics.

A drive may report checksum errors at the filesystem level without showing obvious SMART failures.

Conversely, SMART may reveal developing hardware problems before the filesystem reports permanent corruption.

The project reinforced the importance of using multiple diagnostic sources.

---

## Kernel and Storage Driver Logs

Linux kernel messages can provide evidence of storage-related hardware failures.

Useful information may include:

- Device resets.
- SATA link failures.
- I/O timeouts.
- Controller errors.
- PCIe communication faults.
- Filesystem warnings.
- Unexpected device disconnections.

On a system using persistent journaling, kernel messages can be examined using:

```bash
journalctl -k
```

Errors from the current boot can be filtered using:

```bash
journalctl -k -b -p err
```

The exact commands available on TrueNAS SCALE depend on the operating system version and administrative environment.

The project emphasized correlating kernel-level errors with ZFS status reports.

This helps distinguish data corruption from problems involving the physical storage interface.

---

## Memory Integrity Considerations

System memory is another potential source of storage-related instability.

ZFS uses RAM for caching, metadata and filesystem operations.

Memory corruption can interfere with these operations.

The homelab's server-class hardware supports ECC memory, which provides protection against certain classes of memory errors.

However, ECC does not eliminate every possible hardware fault.

A complete investigation should consider:

- Corrected ECC errors.
- Uncorrectable memory errors.
- Hardware event logs.
- Memory stability.
- System crashes.
- Unexpected kernel behavior.

The project emphasized that storage integrity depends on the reliability of the complete computing platform.

---

## Scrub Performance and System Load

A ZFS scrub can generate substantial disk activity.

The performance impact depends on pool layout, device speed, system workload and the amount of allocated data.

During a scrub, other applications may experience increased storage latency.

This can affect services that depend on the same storage pool.

Potentially affected workloads include:

- Network file transfers.
- Media library scanning.
- Application databases.
- Containerized services.
- Backup operations.
- Virtual machines.

The evaluation emphasized scheduling routine scrubs with consideration for normal system activity.

During an active recovery incident, however, the priority should be understanding hardware condition rather than simply minimizing application performance impact.

---

## Scrub Versus Resilver

ZFS scrubbing and resilvering are related but distinct operations.

### Scrub

A scrub verifies allocated data and metadata and repairs detected errors when valid redundant data is available.

### Resilver

A resilver reconstructs the data required on a device or vdev member, commonly after a disk replacement or reconnection.

The two operations serve different administrative purposes.

A scrub is primarily an integrity verification process.

A resilver is primarily a redundancy restoration process.

The project emphasized understanding this distinction when interpreting ZFS activity and planning storage recovery.

---

## Data Preservation Strategy

Preserving recoverable data was the primary concern during the investigation.

A conservative recovery approach should avoid unnecessary changes until the extent of corruption and the condition of the hardware are understood.

Important principles include:

- Recording pool status before modifying the configuration.
- Preserving kernel and hardware error logs.
- Identifying affected datasets and files.
- Avoiding destructive recovery commands.
- Limiting unnecessary writes.
- Verifying available backups.
- Copying critical recoverable data to independent storage.
- Avoiding repeated high-stress operations on failing hardware.

The project reinforced that an aggressive repair attempt can create additional risk when the underlying hardware is unstable.

---

## Backup-Based Recovery

ZFS redundancy cannot recover every corruption scenario.

Independent backups remain necessary.

When permanent errors affect a file, restoring a verified copy from backup may be the most reliable recovery method.

The backup must be independent of the affected storage pool.

Snapshots stored on the same pool can be useful for recovering earlier file versions, but they are not a substitute for an independent backup.

If the underlying pool becomes inaccessible or loses required data, snapshots on that pool may also become unavailable.

The project emphasized maintaining backups that can survive the failure of the primary storage system.

---

## ZFS Snapshots

ZFS snapshots preserve a point-in-time view of a dataset.

They are useful for recovering from accidental deletion or unwanted file modifications.

However, snapshots do not guarantee protection against physical storage failure.

Snapshots also share the underlying storage infrastructure with their parent datasets.

The project considered snapshots an important data management feature but not a complete corruption recovery solution.

A reliable protection strategy combines appropriate redundancy, snapshots and independent backups.

---

## Recovery Verification

After corruption is addressed, the storage environment must be validated.

A successful scrub or cleared error counter is not sufficient by itself to prove complete recovery.

Verification should include:

1. Reviewing pool health.
2. Confirming that no unresolved permanent errors remain.
3. Validating affected files against known-good copies.
4. Checking physical drive health.
5. Reviewing kernel logs for recurring failures.
6. Confirming dataset accessibility.
7. Testing network file sharing.
8. Verifying dependent application functionality.
9. Reviewing backup integrity.
10. Monitoring the system for recurring errors.

The project emphasized verifying actual data accessibility rather than relying exclusively on a single status indicator.

---

## Relationship to Kernel Panic Troubleshooting

The broader TrueNAS SCALE environment also involved investigation of ZFS-related kernel instability.

Kernel panics and filesystem corruption can sometimes occur within the same storage environment, but they are not equivalent failures.

A kernel panic indicates an operating system-level failure.

A checksum error indicates that data returned from storage does not match the expected checksum.

Either problem may occur independently.

Both may also result from a shared hardware fault.

The project emphasized investigating the relationship between symptoms without assuming a common root cause.

For example, repeated storage controller resets could contribute to both filesystem errors and operating system instability.

However, that possibility would require supporting diagnostic evidence.

---

## Migration to Unraid

The homelab was subsequently migrated from TrueNAS SCALE to Unraid.

This represented a significant change in storage management.

The earlier environment used ZFS as its primary storage architecture.

The subsequent Unraid environment uses a parity-protected array with separate storage pools for other workloads.

The migration introduced different operational considerations involving filesystem integrity, parity protection and application data placement.

The available project history does not establish that ZFS corruption was the sole reason for the migration.

The transition is therefore documented as part of the broader storage infrastructure evolution.

---

## ZFS and Unraid Storage Comparison

The migration provided an opportunity to evaluate different approaches to storage management.

| Characteristic | ZFS | Traditional Unraid Array |
|---|---|---|
| Data integrity | End-to-end checksums | Depends on individual disk filesystem |
| Redundancy | Mirrors, RAIDZ and other configurations | Dedicated parity disks |
| Scrubbing | Verifies checksums and can repair with redundancy | Parity checks verify parity consistency |
| Expansion | Depends on pool and vdev configuration | Individual array disks can be added |
| Disk filesystems | Managed within ZFS pools | Independently formatted array disks |
| Recovery | Depends on pool topology and available redundancy | Individual data disks may remain independently readable |
| Backup requirement | Still required | Still required |

A traditional Unraid parity check does not provide the same end-to-end data integrity guarantees as a ZFS scrub.

Likewise, ZFS redundancy does not eliminate the need for independent backups.

The project reinforced the importance of understanding the actual protection mechanisms provided by each storage architecture.

---

## Application Impact

The storage server supported applications and data used throughout the homelab.

Potentially affected services included:

- Jellyfin media libraries.
- Sonarr and Radarr media management.
- Home Assistant configuration and databases.
- Docker application data.
- Network file shares.
- Download directories.
- Backup storage.

Filesystem corruption can affect applications in different ways.

A corrupted media file may produce playback errors.

A corrupted application database may prevent a service from starting.

Damaged configuration files may cause unexpected application behavior.

The project emphasized identifying critical application data and ensuring that it is included in the backup strategy.

---

## Monitoring and Preventive Maintenance

The investigation highlighted the importance of proactive storage monitoring.

Useful monitoring categories include:

- ZFS pool health.
- Scrub completion.
- Checksum errors.
- Read and write errors.
- SMART attributes.
- Drive temperature.
- Controller errors.
- Memory errors.
- Storage capacity.
- Unexpected reboots.
- Backup completion.

Routine monitoring can help identify developing problems before they become major recovery incidents.

However, monitoring alone does not prevent corruption.

The project emphasized combining monitoring with reliable hardware, appropriate redundancy and verified backups.

---

## Troubleshooting Methodology

The investigation can be organized into a layered recovery workflow.

### Stage 1: Establish the Failure Scope

Determine whether the issue affects individual files, datasets, devices or the entire storage pool.

### Stage 2: Preserve Diagnostic Evidence

Record ZFS status, kernel messages and hardware diagnostics.

### Stage 3: Evaluate Hardware Stability

Inspect drives, cables, controllers, memory and power conditions.

### Stage 4: Review Pool Health

Identify read, write and checksum errors.

### Stage 5: Determine Recovery Capability

Evaluate the pool's redundancy and whether valid copies of affected data remain available.

### Stage 6: Assess Scrub Safety

Determine whether the hardware is stable enough to complete an integrity scan.

### Stage 7: Perform Controlled Recovery

Use supported ZFS recovery mechanisms or restore affected data from verified backups.

### Stage 8: Validate Recovered Data

Confirm that important files and applications function correctly.

### Stage 9: Monitor for Recurrence

Continue reviewing storage and hardware diagnostics after recovery.

This methodology reduces the risk of treating a symptom while leaving the underlying cause unresolved.

---

## Technical Challenges

### Distinguishing Corruption Sources

Checksum errors can originate from several components in the storage path.

### Recoverability

Detected corruption cannot always be repaired.

### Hardware Instability

Failing drives or controllers may prevent a scrub from completing.

### Performance Impact

Integrity scans can produce significant storage I/O.

### Incomplete Error Reporting

Some permanent errors cannot be mapped directly to ordinary file paths.

### Backup Availability

Recovery options are limited when no independent copy of affected data exists.

### Platform Migration

Moving between storage platforms introduces additional data preservation and compatibility requirements.

These challenges informed the project's emphasis on controlled diagnosis and recovery.

---

## Security Considerations

Storage recovery operations may involve sensitive files, application databases and system configuration.

Recovery procedures should preserve appropriate access controls.

Important considerations include:

- Restricting administrative access.
- Protecting diagnostic logs.
- Securing backup media.
- Maintaining appropriate file permissions.
- Avoiding unnecessary network exposure.
- Preserving application credentials.
- Verifying recovery software sources.

The project reinforced that data recovery must preserve confidentiality as well as availability and integrity.

---

## Reliability and Recovery Planning

The project highlighted several important storage reliability principles.

A healthy pool status does not replace independent backup verification.

A successful scrub does not guarantee that future hardware failures will not occur.

Redundancy protects against certain failures but does not protect against every form of corruption or accidental data loss.

A complete recovery strategy requires documented procedures and recoverable copies of critical information.

The evaluation emphasized the importance of:

- Scheduled integrity checks.
- Hardware health monitoring.
- Independent backups.
- Configuration documentation.
- Stable power delivery.
- Recovery testing.
- Clear identification of critical datasets.
- Regular review of storage capacity and hardware condition.

These practices apply regardless of the selected NAS operating system.

---

## Evaluation Outcome

The project examined ZFS corruption detection, scrubbing and the conditions required for successful data repair.

The investigation was associated with the homelab's earlier TrueNAS SCALE storage environment.

The work established a technical framework for interpreting checksum errors, assessing pool health and distinguishing recoverable corruption from permanent data loss.

The broader infrastructure subsequently transitioned to Unraid.

However, the available project information does not establish the exact number of corruption errors, the results of a specific scrub or whether particular files were successfully recovered.

No specific repair outcome is therefore claimed.

The documented outcome is a structured understanding of ZFS integrity management and a recovery methodology centered on data preservation and hardware fault isolation.

---

## Lessons Learned

ZFS checksums provide a mechanism for detecting corruption that may otherwise remain unnoticed.

Detection and repair are separate capabilities.

Successful self-healing depends on the availability of valid redundant data.

Scrubbing is an important integrity verification mechanism but is not a replacement for backups.

Checksum errors do not automatically identify a failing disk.

Storage controllers, cables, memory and power conditions must also be considered.

A scrub can place substantial load on hardware and should be approached cautiously when physical failure is suspected.

Snapshots, redundancy and backups provide different forms of protection.

The project strengthened understanding of filesystem integrity, storage troubleshooting, hardware diagnostics and recovery planning.

---

## Future Improvements

Future work may include:

- Implementing automated storage integrity monitoring.
- Configuring alerts for checksum and device errors.
- Maintaining documented scrub schedules where ZFS is used.
- Evaluating drive health trends.
- Monitoring storage controller errors.
- Maintaining an independent backup destination.
- Testing file restoration procedures.
- Documenting storage device serial numbers and physical locations.
- Establishing recovery procedures for application databases.
- Evaluating filesystem-level integrity tools on Unraid.
- Implementing centralized storage event logging.
- Testing recovery scenarios using non-production datasets.
- Maintaining a formal storage incident response checklist.

These improvements would strengthen the reliability and recoverability of the homelab's storage infrastructure.

---

## Summary

The ZFS Corruption and Scrub Recovery project investigated filesystem integrity failures and recovery mechanisms within a TrueNAS SCALE storage environment.

The work focused on ZFS checksums, scrub operations, storage redundancy, hardware diagnostics and data preservation.

A central finding was that ZFS can detect corruption independently of ordinary storage device error reporting, but successful repair depends on valid redundant data.

The investigation also reinforced the importance of distinguishing filesystem corruption from hardware instability and operating system failures.

The broader homelab subsequently migrated from TrueNAS SCALE to Unraid, providing an opportunity to evaluate alternative storage architectures and their different integrity mechanisms.

Although the exact original scrub results and recovery outcome were not established in the available project record, the project developed a structured approach to diagnosing and recovering from storage integrity problems.

The work demonstrates knowledge of Linux systems administration, OpenZFS, filesystem integrity, storage diagnostics, data recovery, hardware troubleshooting and infrastructure reliability.

---

## Technical References

- [OpenZFS — Official Documentation](https://openzfs.github.io/openzfs-docs/)
- [OpenZFS — zpool Scrub Manual](https://openzfs.github.io/openzfs-docs/man/master/8/zpool-scrub.8.html)
- [OpenZFS — zpool Status Manual](https://openzfs.github.io/openzfs-docs/man/master/8/zpool-status.8.html)
- [TrueNAS — Official Documentation](https://www.truenas.com/docs/)
- [Unraid — Official Documentation](https://docs.unraid.net/)
- [smartmontools — Official Project](https://www.smartmontools.org/)
