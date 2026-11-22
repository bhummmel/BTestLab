# Technical Skills Matrix

This matrix summarizes the technologies, infrastructure platforms, and operational practices demonstrated throughout the BTestLab homelab portfolio.

The portfolio emphasizes hands-on implementation, system integration, troubleshooting, and documentation in a self-hosted infrastructure environment.

## Systems Administration & Infrastructure

| Technology / Skill | Practical Experience | Supporting Documentation |
|---|---|---|
| Linux Administration | Linux configuration, terminal administration, permissions, services, and troubleshooting | [Unraid Infrastructure](infrastructure/unraid.md) |
| Unraid | NAS management, Docker application hosting, storage configuration, and maintenance | [Unraid Infrastructure](infrastructure/unraid.md) |
| TrueNAS SCALE | Storage administration, service deployment, migration, and recovery | [TrueNAS Recovery & Migration](infrastructure/truenas-recovery-and-migration.md) |
| ZFS | Pool administration, scrubbing, corruption investigation, and recovery | [ZFS Recovery](incidents/zfs-corruption-and-scrub-recovery.md) |
| UPS & Power Management | UPS monitoring, NUT integration, and shutdown planning | [UPS Monitoring](infrastructure/ups-monitoring-and-power-protection.md) |

## Networking & Security

| Technology / Skill | Practical Experience | Supporting Documentation |
|---|---|---|
| OPNsense | Firewall configuration, routing, DHCP, DNS, and network integration | [OPNsense & UniFi Architecture](networking/opnsense-unifi-network-architecture.md) |
| pfSense | Firewall deployment, VPN configuration, DNS troubleshooting, and policy routing | [pfSense VPN & DNS](networking/pfsense-vpn-and-dns-infrastructure.md) |
| UniFi | Network controller deployment, device adoption, and controller troubleshooting | [UniFi Controller Troubleshooting](networking/unifi-controller-troubleshooting.md) |
| VPN Technologies | VPN connectivity, tunnel configuration, and traffic routing | [VPN Policy Routing](incidents/pfsense-vpn-policy-routing.md) |
| Ethernet Infrastructure | Structured cabling, termination, and physical network troubleshooting | [Ethernet Cabling](networking/ethernet-termination-and-structured-cabling.md) |
| Network Troubleshooting | Diagnosing connectivity, firewall, DNS, and performance problems | [SB8200 Troubleshooting](networking/sb8200-internet-performance-troubleshooting.md) |

## Containers, Virtualization & Services

| Technology / Skill | Practical Experience | Supporting Documentation |
|---|---|---|
| Docker | Container deployment, configuration, networking, and persistent storage | [Unraid Infrastructure](infrastructure/unraid.md) |
| Docker Compose | Multi-container application configuration and service integration | [Django & PostgreSQL Stack](services/django-postgres-container-stack.md) |
| Kubernetes | Deployment and troubleshooting of containerized services | [Jellyfin Kubernetes Recovery](incidents/jellyfin-kubernetes-deployment-and-auth-recovery.md) |
| GPU Passthrough | PCIe passthrough configuration and troubleshooting | [GPU Passthrough](incidents/truenas-vfio-and-gpu-passthrough-troubleshooting.md) |
| PostgreSQL | Database-backed service deployment and connectivity troubleshooting | [Nextcloud Database Troubleshooting](incidents/nextcloud-docker-networking-and-postgresql-troubleshooting.md) |
| Media Infrastructure | Jellyfin deployment, transcoding, and automated media workflows | [Jellyfin GPU Transcoding](services/jellyfin-gpu-transcoding-and-media-automation.md) |

## Automation, IoT & Local AI

| Technology / Skill | Practical Experience | Supporting Documentation |
|---|---|---|
| Home Assistant | Home automation, integrations, entity management, and YAML automations | [Home Assistant Infrastructure](home-automation/home-assistant-zwave-zigbee-infrastructure.md) |
| Zigbee & Z-Wave | Device integration, wireless mesh networks, and troubleshooting | [Zigbee & Z-Wave](home-automation/home-assistant-zwave-zigbee-infrastructure.md) |
| MQTT | Message-based communication and integration of connected devices | [Home Assistant Infrastructure](home-automation/home-assistant-zwave-zigbee-infrastructure.md) |
| ESPHome & ESP32 | Embedded device configuration and integration | [ESP32 Smart Pet Feeder](automation/esp32-smart-pet-feeder.md) |
| Local Voice AI | Speech recognition, speech synthesis, and voice assistant integration | [Local Voice Assistant Stack](ai-infrastructure/local-voice-assistant-stack.md) |
| GPU AI Inference | GPU hardware evaluation and local AI deployment planning | [GPU Inference Evaluation](ai-infrastructure/gpu-inference-hardware-evaluation.md) |

## VoIP & Telecommunications

| Technology / Skill | Practical Experience | Supporting Documentation |
|---|---|---|
| FreePBX | PBX deployment, SIP trunks, inbound/outbound routing, and troubleshooting | [FreePBX Deployment](services/freepbx-voipms-deployment.md) |
| FusionPBX | VoIP integration, SIP routing, and analog telephone connectivity | [Analog Telephone & FusionPBX](voip/analog-telephone-and-fusionpbx-integration.md) |
| SIP | Registration, authentication, NAT traversal, and call routing | [SIP Registration Troubleshooting](incidents/freepbx-sip-registration-troubleshooting.md) |
| Analog Telephone Adapters | Grandstream and Cisco ATA configuration and troubleshooting | [Grandstream HT802](voip/grandstream-ht802-analog-line-troubleshooting.md) |

## Troubleshooting & Operational Practices

| Competency | Demonstrated Experience | Supporting Documentation |
|---|---|---|
| Root Cause Analysis | Investigation of service failures, configuration errors, and infrastructure incidents | [ZFS Kernel Panic Recovery](incidents/zfs-kernel-panic-recovery.md) |
| Linux Permissions | Filesystem access, ACL troubleshooting, and container permissions | [TrueNAS ACL Troubleshooting](incidents/truenas-acl-and-container-path-troubleshooting.md) |
| Service Recovery | Diagnosing and restoring malfunctioning applications | [Nextcloud Troubleshooting](incidents/nextcloud-docker-networking-and-postgresql-troubleshooting.md) |
| Hardware Troubleshooting | USB device detection, drivers, and peripheral integration | [ReSpeaker Driver Troubleshooting](incidents/respeaker-driver-and-alsa-troubleshooting.md) |
| Technical Documentation | Architecture write-ups, configuration procedures, incident reports, and lessons learned | [Incident Reports](incidents/zfs-kernel-panic-recovery.md) |

## Professional Focus

The practical experience documented in this portfolio is most relevant to the following career paths:

- Junior Systems Administrator
- IT Infrastructure Technician
- Network Support Technician
- Linux Systems Support
- Technical Support Engineer
- Junior Infrastructure Engineer

The projects represent independent homelab experience and experimentation rather than production enterprise employment. They demonstrate transferable technical skills, systematic troubleshooting, and the ability to design, deploy, maintain, and document interconnected systems.
