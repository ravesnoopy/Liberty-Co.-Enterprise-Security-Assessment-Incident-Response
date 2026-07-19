# Incident Report — ALT-2024-003
## Credential Dumping / Privilege Escalation / Active Directory Compromise

| Field | Detail |
|---|---|
| **Alert ID** | ALT-2024-003 |
| **Date** | 2024-06-22 |
| **Analyst** | Leonardo Romero Garcia |
| **Severity** | Critical |
| **Status** | Contained — Under Investigation |
| **Alert Source** | SIEM / Wazuh + Windows Event Logs |
| **NIST Phase** | Detection → Containment → Eradication |

---

## 1. Executive Summary

The SIEM (Wazuh) correlated Windows Event Log activity indicating credential dumping and unauthorized privilege escalation within Liberty Co.'s Active Directory environment. Evidence confirms execution of `m1m1k4tz.exe` (Mimikatz variant) and `Impacket secretsdump.py` on a domain-joined host associated with user **Carlos Ruiz**, resulting in LSASS memory dumping and extraction of credential hashes. Subsequently, an unauthorized service account (`svc_backup_adm`) was created and added to the **Domain Admins** group, granting the threat actor full administrative control over the domain.

This incident is assessed as the likely enabler for both **ALT-2024-001** (LockBit ransomware deployment) and **ALT-2024-002** (lateral movement into HR storage).

---

## 2. Alert Details

| Field | Value |
|---|---|
| **Affected User** | Carlos Ruiz (domain account) |
| **Tools Observed** | `m1m1k4tz.exe`, `Impacket secretsdump.py` |
| **Target Process** | LSASS (credential extraction) |
| **Unauthorized Account Created** | `svc_backup_adm` |
| **Group Membership Added** | Domain Admins |
| **Detection Source** | Wazuh SIEM + Windows Event Logs |

---

## 3. Timeline

| Time (approx.) | Event |
|---|---|
| T-0 | `m1m1k4tz.exe` executed on host associated with Carlos Ruiz |
| T+unknown | LSASS memory dump performed — credential hashes extracted |
| T+unknown | `Impacket secretsdump.py` executed — additional credential harvesting |
| T+unknown | `svc_backup_adm` account created in Active Directory |
| T+unknown | `svc_backup_adm` added to Domain Admins group |
| T+unknown | Wazuh SIEM alert triggered — incident escalated |
| T+unknown | Containment initiated — affected host isolated |

> **Note:** Precise timestamps pending full Windows Event Log extraction (Event IDs 4624, 4672, 4720, 4732) and SIEM correlation timeline.

---

## 4. Affected Assets

| Asset | Impact |
|---|---|
| Active Directory | Fully compromised — unauthorized Domain Admin account created |
| Domain Controller | Targeted by `secretsdump.py` — credential hashes exposed |
| Carlos Ruiz account | Likely used as initial access point for tooling execution |
| `svc_backup_adm` | Attacker-controlled account — Domain Admin privileges |
| All domain-joined systems | At risk — Domain Admin access enables unrestricted lateral movement |
| Credential store | All hashes accessible to the threat actor during the compromise window |

---

## 5. CIA Impact Assessment

| Pillar | Assessment |
|---|---|
| **Confidentiality** | **Critical** — LSASS dump exposes credential hashes for all accounts cached on the compromised host; `secretsdump.py` may have targeted the Domain Controller directly |
| **Integrity** | **Critical** — Unauthorized Domain Admin account active; any AD object, GPO, or system configuration may have been modified |
| **Availability** | **High** — Domain Admin access grants the attacker capability to disable accounts, modify DNS, or deploy additional payloads across all domain-joined systems |

---

## 6. False Positive Analysis

Execution of diagnostic or penetration testing tools by an authorized IT administrator was considered.

This hypothesis is **not supported** by available evidence:

- `m1m1k4tz.exe` is not an authorized enterprise tool and its obfuscated naming convention is consistent with attacker operational security practices.
- `Impacket secretsdump.py` has no authorized use case within Liberty Co.'s documented toolset.
- Creation of `svc_backup_adm` and its immediate addition to Domain Admins was not authorized, documented, or preceded by a change request.
- The host in question belongs to a standard user (Carlos Ruiz), not an IT administrator — execution of these tools from this endpoint is anomalous.

**Assessment: False positive practically ruled out.**

---

## 7. Correlation

This incident is assessed as the **central pivot point** of the broader attack chain observed across Liberty Co.'s environment:

- **ALT-2024-002 → ALT-2024-003:** The AiTM phishing attack targeting `jmendez@` (ALT-2024-002) may have provided the initial foothold or credential material used to reach Carlos Ruiz's environment.
- **ALT-2024-003 → ALT-2024-001:** Domain Admin access obtained via `svc_backup_adm` is the most probable enabler for LockBit 3.0 deployment across the accounting segment (ALT-2024-001).
- **ALT-2024-003 → ALT-2024-004:** Privileged access may have facilitated the DNS tunneling activity observed in ALT-2024-004.

> **Status:** Full attack chain correlation under active investigation. Causal links are assessed as probable but require timestamp alignment and log cross-referencing to confirm.

---

## 8. Containment Actions Taken

- `svc_backup_adm` account disabled and removed from Domain Admins group.
- Carlos Ruiz account suspended pending forensic review of his endpoint.
- Affected host isolated from the network.
- Domain Controller placed under enhanced monitoring — all privileged account activity logged and reviewed.
- Emergency credential reset initiated for all Domain Admin accounts.

---

## 9. MITRE ATT&CK Mapping

| Technique ID | Technique Name | Observed Behavior |
|---|---|---|
| T1003.001 | OS Credential Dumping: LSASS Memory | `m1m1k4tz.exe` used to dump LSASS process memory |
| T1003.002 | OS Credential Dumping: Security Account Manager | `Impacket secretsdump.py` used for additional credential extraction |
| T1078.002 | Valid Accounts: Domain Accounts | Compromised domain account used to execute tooling |
| T1136.002 | Create Account: Domain Account | `svc_backup_adm` created as unauthorized domain account |
| T1098.003 | Account Manipulation: Add to Group | `svc_backup_adm` added to Domain Admins |
| T1548 | Abuse Elevation Control Mechanism | Domain Admin privileges abused for lateral movement and payload deployment |

---

## 10. Recommendations

1. **Immediate:** Audit all AD changes (accounts, GPOs, group memberships, DNS records) made during the compromise window.
2. **Immediate:** Reset credentials for all domain accounts — prioritize Domain Admins, service accounts, and accounts cached on the compromised host.
3. **Immediate:** Review Domain Controller event logs for DCSync activity (Event ID 4662) — assess whether full credential database was exfiltrated.
4. **Short-term:** Deploy Microsoft Defender for Identity (MDI) with alerts for LSASS access, DCSync, and unauthorized group membership changes.
5. **Short-term:** Implement Protected Users security group for all privileged accounts to prevent credential caching and Pass-the-Hash.
6. **Long-term:** Enforce Privileged Access Workstations (PAW) — administrative tools should only execute from designated hardened endpoints.
7. **Long-term:** Implement tiered Active Directory model (Tier 0 / Tier 1 / Tier 2) to limit blast radius of future credential compromise.

---

## 11. Open Investigative Leads

- Initial access vector to Carlos Ruiz's endpoint not yet confirmed — phishing, lateral movement from `jmendez@` session, or direct exploitation are the primary hypotheses.
- Whether DCSync was performed against the Domain Controller — pending Event ID 4662 log review.
- Full list of accounts whose hashes were extracted — pending forensic analysis of the LSASS dump artifacts.
- Duration of `svc_backup_adm` activity before detection — any actions performed under this account must be audited in full.

---

*Report status: Active investigation. Document will be updated as additional evidence is collected.*
