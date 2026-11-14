# Wazuh SIEM — Technical Investigation and Maintenance Report

**Report date:** 2026-11-13  
**Wazuh version:** 4.14.7  
**Scope:** Windows device alerts, Windows event noise, Linux vulnerability detection, kernel activation, SIEM availability

## 1. Executive summary

Investigated unusually high Wazuh alert volume, developed targeted Windows device-severity rules, and triaged 17 historical level-12+ alerts. The high alerts comprised one Windows agent queue-flood warning and 16 Linux kernel CVE detections. Both Linux systems were rebooted into newer already-installed kernels, and Wazuh services recovered successfully after host maintenance. A later Omnissiah vulnerability inventory still flagged the newer kernel; therefore kernel vulnerability remediation remains unverified. Printer and scanner tuning was installed and syntax-validated but has not yet produced a confirmed matching live event.

## 2. Environment and scope

| Host | Role | Relevant details |
|---|---|---|
| Omnissiah | Ubuntu physical host, Wazuh agent, libvirt host | Plex/home services, Wazuh VM, Tailscale |
| wazuh | SIEM VM | Manager, Indexer, Dashboard; Wazuh 4.14.7 |
| lab-gateway | Debian 13 VM | Proxmox-hosted overlay gateway, Wazuh agent |
| kosics-surface | Windows endpoint | Windows Security and system events; Wazuh agent |

Remote administration is over a private overlay network. Public-facing documentation intentionally excludes overlay addresses.

## 3. Windows device recognition: rule 60227

### Observation

Wazuh rule `60227`, severity 8, matched Windows Security Event ID `6416` ("A new external device was recognized by the system"). Actual events included OneNote (Desktop), Microsoft Print to PDF, Microsoft XPS Document Writer, Fax, Adobe PDF, Brother MFC-L3770CDW printer, WSD scanner, Soundcore audio endpoints, Surface HID/keyboard/touchpad components, and USB input devices. Rule metadata maps T1092, but this mapping does not establish malicious removable-media communication.

A printer or scanner appearing in a 6416 event while the endpoint was away from home is not evidence of live connectivity; Windows may re-enumerate installed devices during startup or servicing.

### Built-in rule chain

`60000` (Windows EventChannel decoder and provider) → `60001` (Security channel) → `60103` (successful audit) → `60227` (Event ID 6416). Rule 60000 explicitly requires `windows_eventchannel`. A manually pasted JSON sample decoded as generic `json` and did not enter the chain. Prefixing it with `WinEvtLog:` also failed decoding. These failed synthetic tests should not be represented as successful validation.

### Custom tuning

- `100102`: child of 60227; `win.eventdata.className=PrintQueue`; level 3.
- `100103`: child of 60227; `className=Image` and WSD ScannerService device ID; level 3.
- Nonmatching classes such as HIDClass, Keyboard, Mouse, AudioEndpoint, and USB devices retain the original rule's level 8.

The rules were saved in `/var/ossec/etc/rules/local_rules.xml`. `sudo /var/ossec/bin/wazuh-analysisd -t` returned without error; `wazuh-manager` was restarted and confirmed active. After the restart, three Soundcore AudioEndpoint events continued to match 60227 at level 8, consistent with the rule's intended scope. **No observed post-restart PrintQueue or WSD scanner event matched 100102/100103, so efficacy is pending verification.** The generic PrintQueue match may be too broad for a production environment and should be narrowed to known devices after verification.

## 4. Alert volume and Windows health

### Observed alert counts

The local `alerts.json` aggregation produced:

| Rule | Count | Level | Description |
|---|---:|---:|---|
| 60608 | 1,942 | 4 | Windows Error Reporting summary |
| 60227 | 342 | 8 | External device recognition |
| 60642 | 260 | 3 | Software Protection Service scheduling |
| 61102 | 197 | 5 | Windows System error |
| 61104 | 135 | 3 | Service startup type changed |
| 67022 | 85 | 3 | Local logon |
| 67023 | 71 | 3 | Local logoff |
| 510 | 40 | 7 | Host-based rootcheck anomaly |
| 60110 | 37 | 8 | User account changed |

These counts are not equivalent to the dashboard's 8,000+ total at a different time/range. The later 15-minute dashboard view showed 91 alerts and zero level-12+ alerts, but this does not prove the root cause is fixed.

### Windows Error Reporting evidence

Three inspected 60608 events on the Surface included `WindowsWcpOtherFailure3` (component store servicing), `CbsPackageServicingFailure2` (cumulative package servicing; code 800736b3), and `RADAR_PRE_LEAK_64` for `SearchFilterHost.exe`. These are consistent with prior DISM/component-store and Windows Update issues, but they do not establish that every 60608 event shares the same cause. The agent previously emitted rule 204 (level 12): event queue flooded, with potential delay or loss of events. Investigate Windows servicing health and Wazuh agent queue behavior before suppressing rule 60608.

## 5. High-severity alert triage

The historical `alerts.json` sample contained 17 alerts at level 12+: one rule 204 (Surface agent event queue flooded), five rule 23506 CVE alerts for Omnissiah's `linux-image-7.0.0-43-generic`, and eleven rule 23506 CVE alerts for lab-gateway's `linux-image-amd64`. They are vulnerability detection results, not confirmed exploit activity.

### Kernel maintenance

| Host | Before | After | Verification |
|---|---|---|---|
| Omnissiah | 7.0.0-42-generic | 7.0.0-43-generic | `uname -r`, Tailscale active, Wazuh VM running |
| lab-gateway | 6.12.115+deb13.1-amd64 | 6.12.116+deb13-amd64 | `uname -r`, Tailscale active |

Package updates had been refreshed with `sudo apt update`; no newer kernel was shown by the filtered `apt list --upgradable` checks at that point. Kernel activation was successful, **not proof of CVE resolution**.

### Operational recovery improvement

The Wazuh VM was initially running but absent from `virsh list --autostart`. Autostart was enabled. After Omnissiah rebooted, `sudo virsh dominfo wazuh-siem` reported `State: running` and `Autostart: enable`. On the Wazuh VM, `wazuh-manager`, `wazuh-indexer`, and `wazuh-dashboard` all returned `active`.

## 6. Vulnerability inventory analysis — Omnissiah

The supplied CSV export (`events-2026-11-13T19_16_27.170Z.csv`) contains **1,280 rows**, all for Omnissiah, covering **640 unique CVE IDs**. Each of the following installed packages is associated with 640 records:

- `linux-image-7.0.0-42-generic` version `7.0.0-42.42` — 640 findings.
- `linux-image-7.0.0-43-generic` version `7.0.0-43.43` — 640 findings.

Severity counts across package-CVE records:

| Severity in export | Records |
|---|---:|
| Critical | 10 |
| High | 304 |
| Medium | 278 |
| Low | 42 |
| Unspecified (`-`) | 646 |
| **Total** | **1,280** |

Five distinct critical CVEs occur against both installed packages: CVE-2026-99153, CVE-2026-99079, CVE-2026-99078, CVE-2026-99154, and CVE-2026-98959. The export does not provide patch availability, exploitability assessment, or a validated fixed version. The older kernel remains installed as a fallback. **Do not conclude that the newer kernel is fixed, or remove fallback kernels, without checking vendor advisories and proposed package removals.** A separate lab-gateway vulnerability inventory was not provided.

The following read-only checks were requested but their outputs were not provided at report time:

```bash
apt-cache policy linux-image-generic linux-image-7.0.0-43-generic
sudo apt-get -s autoremove
```

## 7. Validation and limitations

**Verified:** Rule XML passes Wazuh analysisd configuration test; manager restart; other device classes continue generating level-8 events; both hosts boot newer kernels; Tailscale active; Wazuh VM running with autostart enabled; SIEM Manager/Indexer/Dashboard active; CSV metrics calculated from the supplied export.

**Not verified:** Successful printer/scanner rule 100102/100103 matches; a sustained decrease attributable to rule tuning; current vulnerability remediation for either host; cause/resolution of Surface WER flood; disappearance of agent queue floods; kernel package autoremove safety.

## 8. Recommendations and next validation

1. Capture a new post-restart PrintQueue and WSD scanner event; verify rule ID and level. Narrow the matching criteria to known printers if needed.
2. Compare Wazuh's current vulnerability inventory with vendor-fixed package versions. Record which CVEs remain open and whether patches exist.
3. Review Omnissiah's `apt-cache policy` and simulated `autoremove` output before considering old kernel removal.
4. Export lab-gateway's vulnerability inventory separately.
5. Investigate the Windows servicing failures and `SearchFilterHost.exe` warnings; check for recurrence of Wazuh agent queue flooding.
6. After tuning validation, implement and test SSH brute-force correlation with a controlled lab-only simulation.

## 9. Evidence handling

This report is derived from command output, Wazuh dashboard observations, and the supplied Omnissiah vulnerability CSV. No overlay IP addresses, credentials, or full Windows event payloads are included. Historical logs and inventory snapshots have different timestamps and must not be interpreted as a single live measurement.
