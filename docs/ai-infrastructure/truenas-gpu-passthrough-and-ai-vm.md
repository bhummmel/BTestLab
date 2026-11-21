# TrueNAS GPU Passthrough & AI VM

## Overview

The TrueNAS GPU Passthrough & AI VM infrastructure was developed to explore virtualization-based GPU acceleration for AI workloads while maintaining workload isolation and operational flexibility.

The environment combines TrueNAS SCALE virtualization, PCI passthrough, NVIDIA GPU acceleration and containerized AI services in order to create dedicated AI environments capable of utilizing physical GPU hardware directly.

The project was designed to evaluate the feasibility of running GPU accelerated AI workloads within virtualized environments while maintaining near-native hardware performance.

---

## Objectives

The primary objective of the project was to provide virtual machines with direct access to physical GPU hardware through PCI passthrough.

GPU passthrough allows virtualized workloads to utilize dedicated NVIDIA hardware rather than relying on virtualized graphics devices or shared resources.

Workload isolation improves infrastructure flexibility by separating experimental AI services from production infrastructure.

The environment also serves as a platform for experimentation with virtualization technologies, VFIO, CUDA acceleration and AI workload deployment strategies.

The project was intended to improve understanding of hardware accelerated virtualization and GPU resource management within modern infrastructure environments.

---

## Infrastructure Environment

The infrastructure operates on TrueNAS SCALE utilizing integrated virtualization capabilities and PCI device assignment.

The environment combines:

* TrueNAS SCALE
* NVIDIA GPU hardware
* VFIO passthrough
* virtual machine infrastructure
* Ollama
* containerized AI services
* CUDA acceleration
* centralized storage resources

The platform was designed to support experimentation with dedicated AI virtual machines while maintaining operational separation from the primary infrastructure environment.

---

## Virtualization Architecture

The virtualization architecture is organized around dedicated virtual machines designed to support GPU accelerated workloads.

Rather than sharing GPU resources between multiple services, passthrough allows an entire GPU to be assigned directly to a single virtual machine.

This approach provides near-native hardware access while allowing AI services to operate within isolated environments.

The architecture prioritizes experimentation, workload separation and operational flexibility.

Virtual machine isolation allows infrastructure testing and AI experimentation to occur without directly affecting production services.

---

## GPU Passthrough Implementation

GPU passthrough is implemented through VFIO, PCI device assignment and IOMMU isolation.

The passthrough process allows the virtual machine to communicate directly with the physical GPU while bypassing traditional virtualization graphics layers.

Direct GPU assignment improves:

* AI inferencing performance
* CUDA workload efficiency
* hardware acceleration compatibility
* virtualization flexibility

The infrastructure was designed to evaluate how closely virtualized AI workloads could approach native hardware performance.

---

## AI Virtual Machine Environment

The virtual machine environment hosts local AI workloads and supporting infrastructure services.

The environment was designed to support:

* local LLM inferencing
* CUDA accelerated workloads
* AI experimentation
* speech processing
* containerized AI services

Virtualization allows AI environments to be modified, rebuilt or replaced without significantly affecting the broader infrastructure platform.

This separation improves experimentation flexibility and simplifies troubleshooting workflows.

---

## NVIDIA Acceleration

NVIDIA GPU acceleration serves as the foundation of the infrastructure.

The RTX 3070 GPU is utilized for:

* CUDA acceleration
* AI inferencing
* speech processing
* hardware accelerated workloads
* virtualization experimentation

Direct hardware access significantly improves AI workload performance compared to CPU-based processing.

GPU acceleration reduces inference latency and improves responsiveness for computationally intensive workloads.

---

## Containerized AI Services

Containerized deployment is utilized extensively within the AI virtual machine environment.

Containerization simplifies service deployment, workload isolation and dependency management while improving operational flexibility.

The environment supports services such as:

* Ollama
* OpenWebUI
* Whisper
* Piper

Containerized deployment allows AI services to be updated, redeployed and modified without requiring significant infrastructure changes.

Persistent storage mapping is utilized to preserve AI models, configurations and application data.

---

## Storage Architecture

Storage infrastructure prioritizes recoverability, flexibility and configuration retention.

Persistent storage mappings are utilized to maintain:

* AI models
* application data
* service configurations
* container state information
* infrastructure documentation

Centralized storage resources simplify backup management and reduce operational complexity.

Storage architecture is designed to support experimentation while maintaining consistency and recoverability.

---

## Networking Architecture

Networking infrastructure is designed to provide secure communication between AI services, supporting infrastructure and automation platforms.

Virtual machine networking allows AI workloads to integrate with Home Assistant, reverse proxy infrastructure and supporting services throughout the homelab environment.

VPN-first remote administration minimizes unnecessary public exposure while maintaining operational accessibility.

Networking architecture prioritizes simplicity, service integration and operational security.

---

## Monitoring & Telemetry

Infrastructure observability is considered an important component of GPU accelerated virtualization.

Planned monitoring infrastructure includes Grafana dashboards, Prometheus metrics collection and GPU telemetry monitoring.

Monitoring objectives include:

* GPU utilization visibility
* virtualization performance monitoring
* VRAM allocation tracking
* AI workload telemetry
* infrastructure stability visibility

Operational telemetry improves troubleshooting efficiency and long-term infrastructure management.

---

## Security Philosophy

The infrastructure prioritizes workload isolation, minimized cloud dependency and local ownership of AI processing resources.

Virtual machine separation reduces the impact of experimental workloads on production infrastructure while improving operational safety.

VPN-first administration and controlled service exposure improve infrastructure security while maintaining accessibility.

The environment emphasizes local processing and operational independence whenever possible.

---

## Troubleshooting Methodology

Troubleshooting procedures focus heavily on staged validation, hardware isolation and infrastructure observability.

Standard troubleshooting procedures typically involve validating:

* GPU visibility
* VFIO functionality
* IOMMU grouping
* PCI device assignment
* NVIDIA driver functionality
* CUDA compatibility
* virtual machine stability
* container runtime functionality
* workload telemetry

Troubleshooting methodology emphasizes systematic validation and root cause analysis rather than temporary workaround implementation.

---

## Common Issues Encountered

Several challenges were encountered throughout deployment and experimentation including:

* VFIO reset failures
* IOMMU grouping conflicts
* PCI passthrough instability
* NVIDIA driver compatibility issues
* virtual machine display conflicts
* CUDA configuration problems
* GPU visibility failures
* container runtime issues

Resolving these issues required staged troubleshooting, hardware validation and dependency isolation testing.

---

## Reliability & Recovery

Virtual machine isolation significantly improves recoverability and experimentation safety.

Backup and recovery planning include:

* VM configuration retention
* persistent storage mapping
* AI model preservation
* container deployment documentation
* infrastructure configuration backups

Recovery planning prioritizes:

* rapid redeployment
* minimized downtime
* workload consistency
* configuration retention
* simplified service restoration

Virtualization allows experimental environments to be rebuilt quickly while minimizing impact on production services.

---

## Lessons Learned

GPU passthrough provides significantly better performance than traditional virtualized graphics solutions for AI workloads.

Workload isolation improves operational flexibility and simplifies experimentation with emerging technologies.

VFIO and PCI passthrough require careful attention to hardware compatibility, virtualization configuration and driver management.

Containerized AI services simplify deployment and experimentation while reducing infrastructure complexity.

Infrastructure observability is critical when troubleshooting GPU accelerated virtualized workloads.

---

## Future Improvements

Future improvements will focus on expanded AI workloads, improved observability and enhanced GPU resource management.

Planned improvements include:

* dedicated AI hardware
* expanded GPU resources
* larger AI models
* Grafana dashboards
* Prometheus telemetry integration
* improved virtualization observability
* expanded AI orchestration
* enhanced workload monitoring

Future development will continue emphasizing GPU accelerated virtualization, local AI processing and infrastructure experimentation.

---

## Summary

The TrueNAS GPU Passthrough & AI VM infrastructure successfully demonstrated GPU accelerated virtualization for AI workloads through direct PCI passthrough and hardware assignment.

The environment emphasizes workload isolation, GPU acceleration, operational flexibility and experimentation with modern virtualization technologies.

The project provided valuable experience with VFIO, CUDA acceleration, GPU resource management and AI infrastructure deployment while establishing a scalable platform for future experimentation.

