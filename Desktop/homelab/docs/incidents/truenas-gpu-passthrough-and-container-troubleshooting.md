
# TrueNAS GPU Passthrough and Container Troubleshooting

## Overview

The TrueNAS GPU Passthrough and Container Troubleshooting project focused on diagnosing hardware acceleration problems within a TrueNAS SCALE environment, particularly the interaction between NVIDIA graphics hardware, Linux device drivers, containerized applications and virtual machine passthrough.

The project was part of a broader effort to consolidate network-attached storage, media services and GPU-accelerated computing onto a single self-hosted server.

The primary objective was to make physical GPU resources available to applications that could benefit from hardware acceleration while preserving the stability of the underlying NAS.

Potential workloads included media transcoding, local artificial intelligence inference and other computationally intensive services.

The investigation required understanding how TrueNAS SCALE manages GPUs, how Linux exposes hardware devices, and how GPU allocation differs between containers and virtual machines.

A significant architectural consideration was that GPU isolation for virtual machine passthrough is fundamentally different from providing GPU access to containers.

The project also examined application deployment failures, driver initialization, persistent storage, container resource allocation and hardware reliability.

The broader homelab subsequently migrated from TrueNAS SCALE to Unraid, where GPU acceleration continued to be an important component of the infrastructure.

The available project history confirms GPU passthrough and container troubleshooting within the TrueNAS environment. However, the exact original error messages, application configurations and final recovery sequence are not fully documented.

This writeup therefore presents the technical investigation, diagnostic methodology and infrastructure decisions without attributing unverified outcomes to the original deployment.

---

## Objectives

The primary objective was to troubleshoot GPU access for applications and virtual machines hosted on TrueNAS SCALE.

Specific goals included:

- Identifying installed NVIDIA GPU hardware.
- Verifying Linux GPU driver initialization.
- Understanding TrueNAS SCALE GPU isolation.
- Investigating VFIO and IOMMU configuration.
- Distinguishing virtual machine passthrough from container GPU access.
- Diagnosing unavailable GPU allocation options.
- Troubleshooting container startup failures.
- Evaluating GPU-accelerated media transcoding.
- Investigating GPU acceleration for local AI workloads.
- Understanding container device permissions.
- Preserving persistent application data.
- Monitoring GPU resource utilization.
- Evaluating GPU hardware stability.
- Maintaining NAS reliability during troubleshooting.
- Comparing TrueNAS SCALE and Unraid GPU management.

An additional objective was to develop a repeatable process for isolating failures at the hardware, operating system, container runtime and application levels.

---

## Infrastructure Environment

The project took place within the homelab's previous TrueNAS SCALE environment.

TrueNAS SCALE provided storage management through OpenZFS while also supporting application hosting and virtualization.

The broader infrastructure included NVIDIA GPUs, server-class computing hardware and self-hosted services.

| Component | Role |
|---|---|
| TrueNAS SCALE | Original NAS and application platform |
| OpenZFS | Storage management |
| Linux kernel | Hardware and device driver management |
| NVIDIA GPU | Hardware acceleration |
| NVIDIA driver | GPU access from the host operating system |
| Container runtime | Application isolation and deployment |
| KVM/QEMU | Virtual machine infrastructure |
| VFIO | PCIe device assignment |
| IOMMU | DMA isolation for virtual machines |
| Jellyfin | Media server and transcoding workload |
| Ollama | Local language model inference |
| Unraid | Subsequent primary hosting platform |

The homelab has used NVIDIA graphics hardware for accelerated computing, including an RTX 3070 that exhibited hardware instability and another GPU used for AI and media workloads.

The available history does not identify which GPU was involved in every original TrueNAS troubleshooting step.

Consequently, the investigation distinguishes general hardware considerations from confirmed device-specific failures.

---

## System Architecture

The architecture involved several layers between physical GPU hardware and the applications attempting to use it.

```text
                 Physical Server
                        |
                        v
               +----------------+
               | TrueNAS SCALE  |
               | Linux Kernel   |
               +----------------+
                        |
               +--------+--------+
               |                 |
               v                 v
        NVIDIA Driver       VFIO / IOMMU
               |                 |
               v                 v
        Container Runtime     KVM / QEMU
               |                 |
        +------+-------+         v
        |              |     Virtual Machine
        v              v         |
     Jellyfin        Ollama      v
        |              |     Guest GPU Driver
        +------+-------+         |
               |                 |
               v                 v
         Host GPU Access    Assigned GPU
```

This diagram represents two alternative GPU ownership paths.

A GPU exposed to applications through the host NVIDIA driver is not simultaneously available for exclusive VFIO assignment to a virtual machine.

The project emphasized determining which ownership model was required before modifying the system configuration.

---

## TrueNAS SCALE Application Architecture

TrueNAS SCALE has used different application management technologies across its release history.

Earlier releases used Kubernetes-based application infrastructure, while later releases introduced Docker-based application management.

This distinction matters because GPU configuration procedures, container runtime behavior and troubleshooting commands can differ substantially between versions.

The original deployment's exact TrueNAS SCALE release is not established.

Accordingly, the investigation treated operating system version identification as a prerequisite for selecting appropriate troubleshooting procedures.

A configuration applicable to one TrueNAS release should not automatically be assumed to work on another.

The project reinforced the importance of documenting platform versions when troubleshooting appliance-oriented operating systems.

---

## GPU Passthrough Versus Container GPU Access

A central challenge was distinguishing two different methods of providing hardware acceleration.

### Virtual Machine GPU Passthrough

With virtual machine passthrough, a physical PCIe device is assigned to a guest operating system.

The guest loads its own driver and communicates with the GPU through the virtualization environment.

The host generally cannot use the assigned GPU for ordinary applications while it is isolated for the VM.

### Container GPU Access

Containers share the host operating system kernel.

GPU-enabled containers normally use the host's GPU driver and receive access to the necessary device interfaces through the container runtime.

This arrangement can allow multiple compatible containers to use the same GPU, subject to workload demands and platform support.

The project emphasized that these approaches have different requirements and should not be configured interchangeably.

TrueNAS documentation confirms that GPUs isolated for VM assignment are unavailable for ordinary application allocation. <Cite ref="turn603699search2"/> <Cite ref="turn603699search0"/>

---

## Linux GPU Detection

The first diagnostic stage involved verifying that the Linux host could detect the installed GPU.

PCIe devices can be enumerated using:

```bash
lspci -nn
```

NVIDIA hardware can be filtered using:

```bash
lspci -nn | grep -i nvidia
```

The output identifies the device's PCI address and vendor/device identifiers.

A GPU may expose multiple PCIe functions, including a graphics controller and an audio controller.

The investigation emphasized identifying the complete device before modifying driver or passthrough configuration.

If the GPU does not appear in PCIe enumeration, container configuration is unlikely to be the immediate cause.

Instead, the investigation should focus on hardware installation, firmware settings, PCIe connectivity or device failure.

---

## NVIDIA Driver Initialization

A detected GPU must have a functioning driver before ordinary GPU-accelerated applications can use it.

On a compatible NVIDIA Linux installation, the following command provides information about the GPU and driver:

```bash
nvidia-smi
```

A successful response can identify:

- GPU model.
- Driver version.
- Available GPU memory.
- Memory utilization.
- GPU utilization.
- Active GPU processes.
- Device temperature.
- Power consumption.

However, `nvidia-smi` failing does not by itself establish that the graphics card is defective.

Possible causes include:

- Missing NVIDIA driver.
- Driver initialization failure.
- Kernel module incompatibility.
- GPU assigned to VFIO.
- Unsupported GPU architecture.
- Hardware communication problems.

The project emphasized verifying the host driver before troubleshooting application-level acceleration.

---

## Driver Binding Verification

Linux associates PCIe devices with kernel drivers.

The current driver can be inspected using:

```bash
lspci -nnk
```

For a specific device:

```bash
lspci -nnk -s 0000:01:00.0
```

The PCI address is illustrative and must be replaced with the actual address.

The output can identify whether the GPU is controlled by an NVIDIA driver, another graphics driver or `vfio-pci`.

This is particularly useful when a GPU appears in hardware enumeration but is unavailable to applications.

The investigation emphasized that driver ownership determines which software components can access the device.

---

## VFIO and IOMMU Considerations

VFIO provides the Linux framework used for assigning compatible physical devices to virtual machines.

It works with the system's IOMMU to provide device memory isolation.

For an Intel-based server, IOMMU support commonly involves VT-d.

Kernel initialization can be investigated using:

```bash
dmesg | grep -Ei 'DMAR|IOMMU|VFIO'
```

The kernel command line can also be inspected:

```bash
cat /proc/cmdline
```

These commands help establish whether the relevant virtualization infrastructure is active.

However, TrueNAS SCALE manages portions of its system configuration through supported administrative interfaces.

Directly modifying bootloader files or persistent kernel configuration outside supported mechanisms can create maintenance and recovery problems.

The project emphasized using TrueNAS-supported GPU isolation settings whenever possible.

---

## GPU Isolation Conflicts

One of the most important troubleshooting considerations was whether a GPU had been reserved for virtual machine use.

A GPU isolated for a VM may disappear from the list of GPUs available to containerized applications.

This behavior is expected because the GPU cannot simultaneously remain under incompatible host and guest ownership.

For supported TrueNAS SCALE releases, the administrative interface provides GPU isolation settings.

The documented troubleshooting approach includes verifying that the desired GPU is not isolated or assigned to a virtual machine when it is needed by an application.

Changes to GPU isolation may require a system reboot before they take full effect. <Cite ref="turn603699search2"/> <Cite ref="turn603699search0"/>

The project reinforced the importance of checking device allocation before attempting driver reinstalls or application redeployment.

---

## IOMMU Group Inspection

For virtual machine passthrough, IOMMU group topology is an important consideration.

Linux exposes group information through sysfs.

A basic inspection command is:

```bash
find /sys/kernel/iommu_groups/ -type l
```

A more detailed listing can be generated using:

```bash
for group in /sys/kernel/iommu_groups/*; do
    echo "Group ${group##*/}"

    for device in "$group"/devices/*; do
        lspci -nn -s "${device##*/}"
    done
done
```

This identifies devices that share an IOMMU group.

A GPU may share its group with additional PCIe functions.

The investigation emphasized that IOMMU group topology can affect whether a device is suitable for independent VM assignment.

These considerations are generally distinct from ordinary container GPU access.

---

## Container GPU Resource Allocation

For containerized applications, the GPU must be exposed through the platform's supported application configuration.

Depending on the TrueNAS SCALE version, GPU allocation may be configured through application resource settings.

A compatible GPU must be detected, supported by the operating system and available for application use.

In current TrueNAS container management, NVIDIA GPU attachment requires the relevant driver support, while GPU assignment is managed through the container configuration interface. <Cite ref="turn603699search1"/>

The investigation considered several failure scenarios:

- GPU not listed as available.
- GPU already assigned to a VM.
- NVIDIA driver not initialized.
- Container lacking GPU device access.
- Unsupported application image.
- Missing GPU runtime libraries.
- Application unable to initialize hardware acceleration.

The project emphasized identifying which layer was failing before modifying unrelated settings.

---

## Container Runtime Integration

A GPU-enabled container requires more than a functioning host driver.

The container runtime must expose the appropriate GPU device interfaces and supporting libraries.

On a conventional Docker installation, NVIDIA GPU support commonly involves the NVIDIA Container Toolkit.

A Docker configuration may request GPU access using:

```yaml
services:
  gpu-test:
    image: nvidia/cuda:12.4.1-base-ubuntu22.04
    command: nvidia-smi
    gpus: all
```

This example illustrates standard Docker GPU allocation syntax.

It is not a complete TrueNAS SCALE deployment recipe.

The actual supported configuration depends on the TrueNAS version and application management system.

The project emphasized avoiding direct modification of appliance-managed container runtime settings when supported GPU allocation controls are available.

---

## Host Versus Container Diagnostics

A useful troubleshooting technique is comparing GPU availability at different layers.

| Diagnostic Layer | Verification |
|---|---|
| Physical hardware | GPU visible through PCIe enumeration |
| Host kernel | Appropriate driver bound to the device |
| Host driver | NVIDIA management tools function |
| Container runtime | GPU devices can be assigned |
| Container environment | GPU libraries and devices are accessible |
| Application | Hardware acceleration initializes successfully |
| Workload | Actual GPU utilization occurs |

This approach helps identify whether the failure is related to hardware, driver installation, runtime configuration or application settings.

For example, a GPU functioning correctly on the host but unavailable inside a container suggests a different problem from a GPU that cannot initialize its host driver.

The project reinforced the importance of testing each layer independently.

---

## Jellyfin Hardware Transcoding

Jellyfin was one of the relevant GPU-accelerated workloads within the broader homelab.

Hardware transcoding allows supported video processing operations to be performed using dedicated GPU capabilities rather than relying exclusively on the CPU.

NVIDIA GPUs provide NVENC and NVDEC capabilities for supported encoding and decoding operations.

However, codec support depends on GPU generation and driver compatibility.

A GPU appearing in Jellyfin's configuration does not automatically prove that hardware transcoding is functioning.

The project considered the need to verify actual playback behavior and GPU utilization.

A successful test would involve initiating a playback session that requires transcoding and checking whether the expected GPU processing engines are active.

---

## Jellyfin Transcoding Diagnostics

When hardware transcoding fails, several layers must be examined.

Potential causes include:

- Missing GPU device access.
- Incorrect container runtime configuration.
- Unsupported codec.
- Incompatible FFmpeg build.
- Incorrect Jellyfin hardware acceleration settings.
- NVIDIA driver problems.
- GPU memory limitations.
- Application permissions.

The investigation emphasized separating media playback failures from GPU initialization failures.

For example, a video that plays through direct playback does not establish that transcoding is functional.

A controlled test must actually invoke the hardware transcoding pipeline.

---

## Local AI Inference

The broader homelab also includes Ollama and Open WebUI for local language model inference.

GPU acceleration can substantially improve performance for compatible inference workloads.

The project considered how GPU allocation affects containerized AI applications.

Relevant factors include:

- GPU VRAM capacity.
- Model parameter count.
- Quantization.
- CUDA compatibility.
- Driver version.
- Container GPU access.
- Runtime memory requirements.
- Concurrent workloads.

A language model may run successfully using CPU inference even when GPU acceleration is unavailable.

Therefore, successful model execution alone does not establish that the GPU is being used.

The investigation emphasized verifying GPU memory allocation and utilization during inference.

---

## GPU Memory Constraints

GPU memory capacity can limit the size of models and workloads that run efficiently.

Media transcoding and AI inference both consume GPU resources.

Running these workloads simultaneously can create contention.

Potential effects include:

- Increased inference latency.
- Reduced transcoding throughput.
- GPU memory exhaustion.
- Application initialization failures.
- Unexpected workload fallback to CPU processing.

The project considered whether GPU resources should be shared between applications or dedicated to specific workloads.

The appropriate configuration depends on the hardware and expected usage patterns.

---

## Container Startup Failures

Not every container startup problem is caused by GPU passthrough.

A container may fail because of configuration, storage or application dependencies.

Potential causes include:

- Invalid image configuration.
- Missing environment variables.
- Incorrect volume mounts.
- Permission errors.
- Unsupported application versions.
- Insufficient memory.
- Network port conflicts.
- Missing GPU runtime support.

The project emphasized examining application logs before attributing startup failures to hardware acceleration.

For Docker-based environments, useful commands include:

```bash
docker ps -a
```

```bash
docker logs CONTAINER_NAME
```

These commands apply where the supported container environment exposes Docker.

Older TrueNAS SCALE application environments require version-appropriate diagnostic procedures.

---

## Persistent Storage and Permissions

Containerized applications require persistent storage for configuration, databases and other state.

TrueNAS SCALE uses ZFS datasets for storage management.

A container may require access to datasets containing application configuration or media files.

However, filesystem permissions and dataset access controls can prevent applications from reading or writing required data.

The investigation considered:

- Dataset ownership.
- User and group IDs.
- Access control lists.
- Container mount configuration.
- Read-only versus read-write access.
- Application data persistence.
- Backup requirements.

The project emphasized that GPU access and filesystem access are independent requirements.

A container may have a functioning GPU while still failing because it cannot access its media library or configuration directory.

---

## GPU Hardware Reliability

The broader homelab included an NVIDIA RTX 3070 that exhibited graphical artifacting.

This introduced an important diagnostic consideration: not every GPU-related failure is caused by software.

Hardware instability can produce symptoms such as:

- Display artifacts.
- Driver crashes.
- Unexpected device resets.
- GPU initialization failures.
- Computational errors.
- System instability under load.

A GPU exhibiting physical hardware problems should not be assumed to become reliable through passthrough or container configuration changes.

The project emphasized distinguishing hardware faults from driver and runtime problems.

The available history does not establish that the artifacting RTX 3070 was the GPU responsible for the original TrueNAS container issues.

---

## Power and Thermal Considerations

GPU-accelerated workloads can place significant demands on server power delivery and cooling.

The investigation considered:

- GPU power consumption.
- PCIe slot power delivery.
- Auxiliary power connectors.
- Power supply capacity.
- Airflow restrictions.
- GPU operating temperature.
- Sustained computational load.

Server chassis airflow may differ from the cooling environment expected by consumer graphics cards.

A GPU that performs adequately during short tests may behave differently under sustained inference or transcoding workloads.

The project emphasized monitoring hardware conditions during performance testing.

---

## Troubleshooting Methodology

The investigation used a layered approach to fault isolation.

### Stage 1: Identify the TrueNAS SCALE Version

Determine which application management system and GPU configuration interfaces are available.

### Stage 2: Verify Hardware Detection

Confirm that the GPU appears in PCIe enumeration.

### Stage 3: Inspect Driver Ownership

Determine whether the device is controlled by NVIDIA drivers, another host driver or VFIO.

### Stage 4: Check GPU Isolation

Verify that the GPU is assigned to the intended workload category.

### Stage 5: Validate Host GPU Operation

Use supported NVIDIA management tools to inspect device initialization.

### Stage 6: Inspect Container Allocation

Confirm that the application is configured to receive GPU access.

### Stage 7: Review Application Logs

Identify driver, library, permissions or runtime errors.

### Stage 8: Test a Minimal GPU Workload

Use a simple workload to distinguish GPU access problems from application-specific failures.

### Stage 9: Validate the Intended Application

Test actual media transcoding or AI inference.

### Stage 10: Monitor Stability

Observe GPU utilization, memory consumption, temperature and application behavior.

This methodology reduces unnecessary configuration changes and helps isolate failures more efficiently.

---

## Technical Challenges

### Platform Version Differences

TrueNAS SCALE application management has changed across releases.

### GPU Ownership Conflicts

A GPU assigned to a virtual machine cannot simultaneously serve as an ordinary application GPU.

### Driver Initialization

The host must load compatible drivers for container-based NVIDIA acceleration.

### Runtime Configuration

GPU devices and libraries must be exposed to the container.

### Application Compatibility

The application must support the GPU and its acceleration framework.

### Persistent Storage

Containers require correct dataset access and permissions.

### Hardware Instability

Physical GPU defects can resemble software configuration problems.

### NAS Reliability

Experimental GPU changes must not compromise storage availability.

These challenges reinforced the need for version-specific documentation and systematic troubleshooting.

---

## Security Considerations

GPU-enabled containers require access to hardware device interfaces.

This can increase the resources available to a compromised application.

The project emphasized:

- Using supported GPU allocation mechanisms.
- Avoiding unnecessary privileged containers.
- Restricting access to host devices.
- Maintaining current application images.
- Protecting persistent datasets.
- Limiting administrative access.
- Preserving appropriate filesystem permissions.
- Avoiding unsupported kernel modifications.

Virtual machine passthrough also depends on appropriate IOMMU isolation.

The project reinforced that hardware acceleration should be configured with the minimum permissions required by the workload.

---

## Reliability and Recovery

A NAS server is responsible for maintaining persistent storage availability.

GPU acceleration is useful, but it should not compromise the server's primary storage function.

The project emphasized maintaining a recoverable configuration.

Important practices include:

- Recording original GPU settings.
- Maintaining TrueNAS configuration backups.
- Documenting application resource assignments.
- Preserving application data.
- Testing changes incrementally.
- Maintaining remote administration access.
- Monitoring system logs.
- Verifying storage services after reboot.
- Establishing a rollback procedure.

The investigation highlighted the operational risk of making low-level hardware changes on a system that also hosts critical storage.

---

## Migration to Unraid

The homelab subsequently transitioned from TrueNAS SCALE to Unraid.

The migration changed how applications and hardware resources were administered.

Unraid provides a Docker-based application environment and supports GPU access for compatible containers.

The broader infrastructure continued to use NVIDIA acceleration for media processing and local AI services.

The transition also created an opportunity to simplify application deployment and separate storage management from experimental virtualization configurations.

However, the available project history does not establish that GPU troubleshooting was the sole reason for migrating away from TrueNAS SCALE.

The migration is therefore documented as part of the broader infrastructure evolution.

---

## Evaluation Outcome

The project investigated GPU passthrough and hardware acceleration problems affecting containerized applications within TrueNAS SCALE.

The work focused on Linux GPU detection, driver binding, VFIO isolation, application GPU allocation and container runtime configuration.

It also examined hardware reliability and the operational implications of hosting GPU workloads on a NAS.

The investigation established a structured approach to distinguishing hardware faults, host driver problems, container runtime failures and application-specific errors.

The available project record does not preserve the exact original error messages or establish that a particular GPU passthrough configuration was successfully restored.

The documented outcome is the troubleshooting experience and the resulting understanding of GPU resource management across virtualization and container platforms.

---

## Lessons Learned

GPU passthrough and container GPU acceleration are different technologies with different ownership requirements.

A GPU isolated for virtual machine use may be unavailable to applications.

Hardware detection does not guarantee successful driver initialization.

A functioning host driver does not automatically provide GPU access inside containers.

Application startup does not establish that hardware acceleration is functioning.

GPU resource allocation must be verified through actual workload testing.

TrueNAS SCALE version differences can significantly affect configuration procedures.

Persistent storage permissions and GPU access must be troubleshot independently.

Hardware instability must be considered before attributing every failure to software.

The project strengthened understanding of Linux GPU drivers, container runtimes, virtualization, hardware acceleration and infrastructure troubleshooting.

---

## Future Improvements

Future work may include:

- Maintaining detailed GPU hardware inventories.
- Recording PCIe device identifiers.
- Documenting driver and runtime versions.
- Establishing repeatable GPU validation tests.
- Benchmarking Jellyfin hardware transcoding.
- Measuring Ollama GPU utilization.
- Monitoring GPU memory and temperature.
- Separating experimental workloads from critical storage services.
- Evaluating dedicated AI compute nodes.
- Maintaining application configuration backups.
- Documenting GPU assignment procedures on Unraid.
- Testing recovery after driver or application updates.
- Implementing centralized GPU monitoring.
- Developing a standardized container troubleshooting checklist.

These improvements would help create a more maintainable and reliable GPU-accelerated homelab environment.

---

## Summary

The TrueNAS GPU Passthrough and Container Troubleshooting project focused on investigating hardware acceleration problems within a Linux-based NAS environment.

The work examined NVIDIA GPU detection, driver initialization, VFIO isolation, IOMMU configuration, container resource allocation and application-level GPU utilization.

A central finding was that GPU availability depends on several independent layers, from physical PCIe hardware through kernel drivers and container runtime configuration to the application itself.

The investigation also highlighted the operational differences between assigning a GPU exclusively to a virtual machine and sharing GPU resources with containerized applications.

The broader homelab subsequently migrated from TrueNAS SCALE to Unraid, where GPU acceleration remained relevant to media and AI services.

Although the exact original repair outcome was not established in the available project record, the work provided a structured foundation for diagnosing GPU access and container deployment failures.

The project demonstrates skills in Linux systems administration, NVIDIA GPU acceleration, containerization, KVM/QEMU, VFIO, IOMMU configuration, hardware diagnostics, application troubleshooting and infrastructure architecture.

---

## Technical References

- [TrueNAS SCALE — GPU Allocation for Applications](https://www.truenas.com/docs/scale/24.04/scaletutorials/apps/)
- [TrueNAS — GPU Isolation for Virtual Machines](https://www.truenas.com/docs/scale/27/systemsettings/advanced/managegpu/)
- [TrueNAS — Container Management and GPU Devices](https://www.truenas.com/docs/scale/27/containers/managingcontainers/)
- [Linux Kernel — VFIO Documentation](https://docs.kernel.org/driver-api/vfio.html)
- [NVIDIA — Container Toolkit Documentation](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/)
- [Docker — GPU Support](https://docs.docker.com/compose/how-tos/gpu-support/)
- [Jellyfin — NVIDIA Hardware Acceleration](https://jellyfin.org/docs/general/post-install/transcoding/hardware-acceleration/nvidia/)
- [Ollama — Official Documentation](https://docs.ollama.com/)
- [Unraid — Official Documentation](https://docs.unraid.net/)
