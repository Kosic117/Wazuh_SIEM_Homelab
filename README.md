# Wazuh SIEM Home Lab

**Security Monitoring | SIEM Administration | Endpoint Telemetry | Detection Engineering**

## Overview

This project documents the design, deployment, and ongoing development of a self-hosted Wazuh Security Information and Event Management (SIEM) environment.

The goal is to build practical experience in cybersecurity operations by implementing centralized monitoring across Windows and Linux systems, validating security telemetry, and developing detection and investigation workflows.

Rather than focusing solely on installing security software, this project emphasizes infrastructure design, operational troubleshooting, network segmentation, evidence-based validation, and technical documentation.

**Project Status:** Active Development

## Lab Architecture

The SIEM environment runs on a dedicated Ubuntu Server virtual machine using KVM/QEMU and libvirt virtualization.

The deployment includes:

- **SIEM Platform:** Wazuh 4.14.7 (Manager, Indexer, Dashboard)
- **Virtualization:** KVM/QEMU, libvirt
- **Operating Systems:** Ubuntu Server, Debian Linux, Windows 11
- **Endpoint Monitoring:** Four initially onboarded agents
- **Remote Administration:** Authenticated Tailscale overlay network
- **Security Testing:** Separate isolated Proxmox environment

The architecture is designed to preserve segmentation between administrative systems, normal home-server services, and intentionally vulnerable security-testing workloads.

## Completed Milestones

### 1. Infrastructure Preparation

- Validated host resources and virtualization capabilities
- Allocated dedicated LVM storage for the SIEM virtual machine
- Verified installation-media integrity
- Configured virtual networking and guest operating system

### 2. Wazuh Deployment

- Installed the Wazuh manager, indexer, and dashboard
- Verified service availability and resource utilization
- Established secure remote administrative access
- Configured a separate named administrative account

### 3. Endpoint Onboarding

Successfully enrolled and validated four monitored endpoints:

| Endpoint Role | Operating System |
|---|---|
| Server and virtualization host | Ubuntu Linux |
| Primary workstation | Windows 11 |
| Mobile administrative workstation | Windows |
| Security-lab gateway | Debian Linux |

Validated agent connectivity, system inventory, service status, and initial security-event ingestion.

## Current Development

The next phase focuses on security operations and detection validation, including:

- Security Configuration Assessment (SCA)
- Vulnerability detection and analysis
- File Integrity Monitoring (FIM)
- Authentication-event monitoring
- Custom Wazuh detection rules
- Controlled security-event generation and alert investigation

These capabilities will be documented with reproducible test cases, relevant alert evidence, and lessons learned as they are validated.

## Key Engineering Decisions

- **Infrastructure isolation:** Deployed Wazuh in a dedicated virtual machine to separate security monitoring from existing production-like home-server services.
- **Secure administration:** Used an authenticated Tailscale overlay for remote access rather than directly exposing the Wazuh dashboard to the public internet.
- **Network segmentation:** Maintained separation between intentionally vulnerable lab systems and normal home infrastructure.
- **Resource management:** Validated CPU, memory, storage, and service health to ensure the SIEM could coexist with existing workloads.
- **Evidence-based validation:** Verified endpoint connectivity, system inventory, and event ingestion rather than treating successful installation as proof of operational readiness.

## Technical Skills Demonstrated

- Linux and Windows system administration
- SIEM deployment and management
- Virtualization and resource allocation
- Network segmentation and secure remote access
- Endpoint agent deployment
- Centralized telemetry collection
- Troubleshooting and service validation
- Technical reporting and documentation

## Documentation

The full project report includes implementation milestones, architectural decisions, troubleshooting procedures, command-line validation, and supporting screenshots.

**Full technical report:** `docs/Wazuh-SIEM-Project-Report.pdf` *(to be published after sanitization)*

## Security and Scope

All security testing is performed within owned or explicitly authorized systems. Public documentation excludes credentials, sensitive addressing, authentication tokens, and other private infrastructure information.

## Project Roadmap

- [x] Deploy Wazuh SIEM infrastructure
- [x] Establish authenticated remote access
- [x] Onboard Windows and Linux endpoints
- [x] Validate initial security telemetry
- [ ] Complete baseline security posture assessment
- [ ] Validate FIM and authentication monitoring
- [ ] Develop and test custom detection rules
- [ ] Document alert investigations
- [ ] Publish sanitized technical evidence and final architecture diagram

---

*This project is maintained as a practical cybersecurity learning environment and professional technical portfolio.*
