# Wazuh Follow-up Checklist

## Priority 1 — Confirm security posture

- [x] Boot lab-gateway into kernel `6.12.116+deb13-amd64`; confirm Tailscale active
- [x] Boot Omnissiah into kernel `7.0.0-43-generic`; confirm Tailscale active
- [x] Enable and verify Wazuh VM autostart
- [x] Confirm Wazuh Manager, Indexer, and Dashboard active
- [ ] Review output of `apt-cache policy linux-image-generic linux-image-7.0.0-43-generic` (command run; output not supplied)
- [ ] Review `sudo apt-get -s autoremove` before deleting any kernels (command run; output not supplied)
- [ ] Verify current CVE status and fixed versions against Ubuntu/Debian security advisories
- [ ] Export and review lab-gateway vulnerability inventory

## Priority 2 — Validate alert tuning

- [x] Create custom rule 100102 for printer queues, level 3
- [x] Create custom rule 100103 for WSD scanners, level 3
- [x] Run `sudo /var/ossec/bin/wazuh-analysisd -t` successfully
- [x] Restart manager and confirm active
- [x] Verify unrelated AudioEndpoint events still match 60227, level 8
- [ ] Observe a new PrintQueue event matching 100102, level 3
- [ ] Observe a new WSD scanner event matching 100103, level 3
- [ ] Consider narrowing broad PrintQueue tuning to known printer IDs
- [ ] Record before/after counts over comparable time windows

## Priority 3 — Investigate Windows endpoint health

- [ ] Investigate repeated rule 60608 Windows Error Reporting summaries
- [ ] Resolve/verify Windows component store servicing health and failed updates
- [ ] Check SearchFilterHost.exe memory-resource warnings
- [ ] Check whether Wazuh agent queue-flood rule 204 recurs
- [ ] Review rule 60110 user-account changes for authorized activity

## Priority 4 — Portfolio and detection engineering

- [x] Record technical investigation and maintenance results
- [ ] Add sanitized evidence screenshots with captions
- [ ] Test and document SSH brute-force detection in the isolated lab
- [ ] Record test logs, expected rule IDs, false-positive analysis and remediation

## Useful verification commands

```bash
# Wazuh manager: syntax test
sudo /var/ossec/bin/wazuh-analysisd -t

# Wazuh manager: new device-rule events
sudo tail -n 0 -F /var/ossec/logs/alerts/alerts.json | jq --unbuffered -c 'select(.rule.id == "60227" or .rule.id == "100102" or .rule.id == "100103") | {timestamp,rule:.rule.id,level:.rule.level,class:.data.win.eventdata.className,device:.data.win.eventdata.deviceDescription}'

# Omnissiah host: VM state and autostart
sudo virsh dominfo wazuh-siem | grep -E 'State:|Autostart:'

# Linux: running kernel
uname -r
```

**Command notes:** `sudo` elevates permissions; `-t` checks rule configuration; `tail -n 0 -F` follows only newly appended log lines across rotation; `jq` selects and formats JSON; `virsh dominfo` reports VM configuration; `grep -E` filters output; `uname -r` prints the running kernel.
