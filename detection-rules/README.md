# Custom Detection Rules

This directory contains custom Wazuh detection rules developed and tested as part of the SIEM home lab.

## Current Rules

### Windows Device Recognition Tuning

**File:** `local_device_tuning.xml`

Custom rules designed to reduce alert severity for recognized printer queues and WSD scanners while preserving visibility into other device-recognition events.

- Parent Wazuh rule: `60227`
- Custom rule IDs: `100102`, `100103`
- Target severity: Level 3
- Configuration syntax validation: Passed
- Live matching verification: Pending

## Future Development

- SSH brute-force detection and correlation
- Additional Windows event tuning
- Detection testing and validation

Rules will be documented with their purpose, matching conditions, validation results, and known limitations.
