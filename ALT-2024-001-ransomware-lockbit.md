# Incident Report — ALT-2024-001
## Ransomware Deployment / LockBit 3.0

| Field | Detail |
|---|---|
| **Alert ID** | ALT-2024-001 |
| **Date** | 2024-06-22 |
| **Analyst** | Leonardo Romero Garcia |
| **Severity** | Critical |
| **Status** | Contained |
| **Alert Source** | CrowdStrike EDR |
| **NIST Phase** | Detection → Containment → Eradication |

---

## 1. Executive Summary

CrowdStrike EDR generated a critical alert following the detection of ransomware activity consistent with **LockBit 3.0** on an endpoint within Liberty Co.'s accounting segment. Analysis indicates that a malicious process masquerading as a legitimate Windows service (`svchost32.exe`) established command-and-control (C2) communication and initiated mass encryption of financial files. A total of **847 files** were confirmed encrypted. Evidence suggests data exfiltration preceded the encryption event.

The false positive probability is assessed as **low**. The combination of C2 beacon traffic, mass file encryption across critical file types, and process impersonation is inconsistent with any authorized administrative activity.

---

## 2. Alert Details

| Field | Value |
|---|---|
| **Malicious Process** | `svchost32.exe` |
| **PID** | 4492 |
| **C2 Domain** | `lockbit3-decryptor[.]onion` |
| **Encrypted Extensions** | `.docx` `.xlsx` `.pdf` `.sql` `.bak` |
| **Files Affected** | 847 |
| **Affected Segment** | Accounting — File Server |

---

## 3. Timeline

| Time (approx.) | Event |
|---|---|
| T-0 | `svchost32.exe` (PID 4492) spawned on accounting endpoint |
| T+unknown | C2 beacon established to `lockbit3-decryptor[.]onion` |
| T+unknown | Mass encryption initiated across 847 files |
| T+unknown | CrowdStrike EDR alert triggered |
| T+unknown | Alert escalated for IR investigation |
| T+unknown | Endpoint isolated — containment applied |

> **Note:** Precise timestamps are pending full log extraction from the SIEM. This timeline will be updated as evidence is correlated.

---

## 4. Affected Assets

| Asset | Impact |
|---|---|
| Accounting file server | Primary target — 847 files encrypted |
| Accounting endpoint(s) | Likely initial execution point |
| C2 channel (outbound) | Active during encryption window |
| Credential store | Potentially exposed — under investigation |
| Cloud storage | Potentially compromised — under investigation |

---

## 5. CIA Impact Assessment

| Pillar | Assessment |
|---|---|
| **Confidentiality** | **High** — 847 financial files assessed as exfiltrated prior to encryption |
| **Integrity** | **High** — LockBit 3.0 confirmed active within the domain; files encrypted and unrecoverable without decryption key |
| **Availability** | **High** — Accounting server compromised via active C2 channel; services disrupted |

---

## 6. False Positive Analysis

An administrative encryption tool or legitimate backup process was considered as an alternative explanation.

This hypothesis is **not supported** by available evidence:

- `svchost32.exe` is not a valid Windows system binary name — the legitimate process is `svchost.exe`. This naming pattern is consistent with process impersonation.
- Outbound communication to a `.onion` domain is not consistent with any authorized business activity.
- Encryption of 847 files across financial file types (`.sql`, `.bak`, `.xlsx`) is not consistent with routine IT operations.

**Assessment: False positive ruled out.**

---

## 7. Correlation

Evidence suggests a potential relationship with **ALT-2024-003** (credential dumping / privilege escalation). The creation of an unauthorized Domain Admin account (`svc_backup_adm`) observed in ALT-2024-003 may have provided the threat actor with the access level required to deploy ransomware across the accounting segment.

> **Status:** Correlation under active investigation. Causal link not yet confirmed — pending timeline alignment between both incidents.

---

## 8. Containment Actions Taken

- Affected endpoint(s) isolated from the network.
- Outbound traffic to `lockbit3-decryptor[.]onion` blocked at perimeter firewall.
- Accounting file server taken offline pending forensic review.
- Credentials potentially exposed during the incident flagged for mandatory rotation.

---

## 9. MITRE ATT&CK Mapping

| Technique ID | Technique Name | Observed Behavior |
|---|---|---|
| T1059 | Command and Scripting Interpreter | Malicious process execution |
| T1036.005 | Masquerading: Match Legitimate Name | `svchost32.exe` impersonating Windows service |
| T1071.001 | Application Layer Protocol: Web Protocols | C2 communication via `.onion` domain |
| T1486 | Data Encrypted for Impact | 847 files encrypted with LockBit 3.0 |
| T1041 | Exfiltration Over C2 Channel | Data exfiltration assessed prior to encryption |

---

## 10. Recommendations

1. **Immediate:** Restore encrypted files from last known clean backup — verify backup integrity before restoration.
2. **Immediate:** Conduct full credential audit across accounting segment and any system reachable from the compromised endpoint.
3. **Short-term:** Implement application allowlisting to prevent execution of unauthorized binaries.
4. **Short-term:** Block outbound connections to `.onion` domains and non-categorized external IPs at the firewall level.
5. **Long-term:** Deploy EDR behavioral rules to detect process name impersonation patterns (e.g., `svchost` variants).
6. **Long-term:** Establish and test a formal ransomware playbook including communication protocols, legal notification thresholds, and backup restoration procedures.

---

## 11. Open Investigative Leads

- Precise infection vector not yet identified — phishing, lateral movement from ALT-2024-003, or VPN abuse are the primary hypotheses.
- Volume and content of exfiltrated data not yet confirmed.
- Full scope of C2 activity window pending SIEM log review.
- Relationship with ALT-2024-003 (privilege escalation) requires timeline correlation.

---

*Report status: Active investigation. Document will be updated as additional evidence is collected.*
