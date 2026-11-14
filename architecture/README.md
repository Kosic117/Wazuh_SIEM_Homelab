# Lab Architecture

This directory documents the infrastructure and network design of the Wazuh SIEM home lab.

## Environment Overview

- **SIEM Platform:** Wazuh deployed in a virtual machine on an Ubuntu Server host.
- **Virtualization:** KVM/libvirt and Proxmox VE.
- **Monitored Endpoints:** Windows and Linux systems.
- **Remote Connectivity:** Tailscale overlay network.
- **Security Design:** Restricted remote access and network isolation for vulnerable lab systems.

## Planned Documentation

- Network architecture diagram
- Wazuh installation and configuration
- Agent deployment and connectivity
- Network isolation and security controls

Detailed implementation information is available in the [Technical Report](../docs/TECHNICAL_REPORT.md).
