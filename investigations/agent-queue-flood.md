# Wazuh Agent Queue Saturation Investigation

**Project:** Wazuh SIEM Home Lab  
**Investigation Type:** Endpoint Monitoring / Agent Health  
**Affected Endpoint:** Windows Surface Pro 7 (`kosics-surface`)  
**Investigation Date:** November 14, 2026  
**Status:** Monitoring / Root Cause Not Yet Confirmed

---

## 1. Executive Summary

During routine monitoring of the Wazuh SIEM home lab, a Windows endpoint generated multiple alerts indicating agent event queue saturation.

The agent progressed through queue utilization warnings, reached full capacity, and reported a flooded event buffer before returning to normal operation.

The event raised concerns regarding monitoring reliability and potential loss of security telemetry.

**Key Findings:**

- The agent reported queue utilization reaching 90%.
- The event queue subsequently reached full capacity.
- Wazuh warned that events may have been lost.
- A buffer flood alert was generated at severity level 12.
- The agent reported recovery approximately 76 seconds after the initial warning.
- An earlier agent disconnection was observed, but its relationship to
