# Wazuh SIEM Home Lab — Detection Tuning and Vulnerability Management

**Status:** In progress | **Platform:** Wazuh 4.14.7 | **Environment:** Windows endpoint, Ubuntu server, Debian lab gateway, virtualized SIEM

## Project overview

This portfolio project demonstrates security monitoring, alert triage, custom detection-rule tuning, Linux vulnerability assessment, and resilient SIEM operations in a self-hosted lab. All systems are personally administered; remote management uses a private overlay network. No overlay IP addresses or secrets are included here.

## Architecture

- **Wazuh SIEM VM:** Manager, Indexer, and Dashboard, hosted via libvirt on the Ubuntu server.
- **Omnissiah:** Ubuntu host, Plex and other home services, Wazuh VM host, Wazuh agent.
- **lab-gateway:** Debian 13 VM on a separate Proxmox host, overlay access gateway and Wazuh agent.
- **Windows Surface:** Wazuh agent; primary endpoint used for Windows event analysis.

## Work completed

1. Investigated Windows Security Event ID **6416**, which Wazuh rule **60227** categorizes as level 8 for any newly recognized external device, including routine print queues and WSD scanners.
2. Added local child rules **100102** (PrintQueue) and **100103** (WSD scanner) to reduce matching events to level 3. Ruleset validation passed and the manager restarted successfully. **Live matching remains unverified.**
3. Examined event-volume contributors: **60608** (1,942 Windows Error Reporting alerts), **60227** (342 device alerts), **60642** (260 software protection events), and **61102** (197 system errors). These counts reflect the local alerts.json file at the time of review, not a uniform dashboard window.
4. Triaged **17 level-12+ alerts** in the historical log: 16 Linux kernel CVE alerts and one Windows agent queue-flood alert. Historical counts are not the same as current active vulnerabilities.
5. Booted Omnissiah into **7.0.0-43-generic** and lab-gateway into **6.12.116+deb13-amd64**; verified Tailscale active on both.
6. Enabled Wazuh VM autostart and confirmed it survived the host reboot. Wazuh Manager, Indexer and Dashboard were all active afterward.
7. Exported Omnissiah's vulnerability inventory: **1,280 package-CVE records, 640 unique CVEs**, split equally across the two installed kernel packages. The newer running kernel remains flagged; remediation is **not confirmed**.

## Key security lessons

- A Windows 6416 event is device enumeration, not proof of malicious USB activity or remote printer connectivity.
- Severity tuning should be narrow, tested, and auditable; it must not hide unexpected device classes.
- Alert volume, high-severity alert counts, and vulnerability inventory represent different measurements.
- Installing and booting a newer kernel does not prove that all CVEs have been fixed.
- Virtual machine autostart is an important operational recovery control.

## Repository contents

- `TECHNICAL_REPORT.md` — detailed evidence, commands, findings, and limitations
- `FOLLOW_UP_CHECKLIST.md` — prioritized next actions and acceptance criteria
- `rules/local_device_tuning.xml` — deployed local-rule excerpt (verify on manager before reuse)

## Next steps

Confirm custom rules against new printer/scanner events; verify CVE package fix availability and current vulnerability status; investigate Windows servicing errors and event queue flood; build and validate an SSH brute-force correlation rule.

**Data handling:** Endpoint names are retained as lab identifiers. No Tailscale/overlay addresses, credentials, or private log payloads are published.

## Security Operations Case Studies

Practical security operations exercises conducted within the lab, documenting detection, assessment, remediation, and validation.

| Case Study | Focus Area | Status |
|---|---|---|
| [001 — Linux Vulnerability Assessment and Remediation](vulnerability-management/001-lab-gateway-remediation/README.md) | Vulnerability Management | In Progress |
