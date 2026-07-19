# Incident Report — ALT-2024-005
## Internal Network Scanning / Unauthorized Reconnaissance

| Field | Detail |
|---|---|
| **Alert ID** | ALT-2024-005 |
| **Date** | 2024-06-22 |
| **Analyst** | Leonardo Romero Garcia |
| **Severity** | Medium |
| **Status** | Contained — Under Investigation |
| **Alert Source** | IDS / Suricata |
| **NIST Phase** | Detection → Investigation → Containment |

---

## 1. Executive Summary

Suricata IDS generated an alert following the detection of systematic internal network scanning activity originating from **LAPTOP-MKT-03**, a marketing department endpoint. Analysis indicates the scan originated from a virtual machine installed on the laptop rather than the host OS itself — consistent with an attacker staging reconnaissance tooling within a guest environment to reduce host-level detection. The scan targeted **1,247 hosts** across the `192.168.0.0/16` subnet and probed high-value service ports commonly associated with remote access, file sharing, and database services.

This activity is assessed as internal reconnaissance likely intended to map Liberty Co.'s network topology in support of lateral movement or additional payload deployment. A definitive attacker attribution requires further investigation — insider threat and external compromise of the endpoint are both active hypotheses.

---

## 2. Alert Details

| Field | Value |
|---|---|
| **Source Host** | `LAPTOP-MKT-03` |
| **Source IP** | `192.168.20.88` (registered as public IP — anomalous for internal host) |
| **Scan Scope** | `192.168.0.0/16` — 65,536 addressable hosts |
| **Hosts Contacted** | 1,247 |
| **Ports Probed** | `22` `80` `443` `445` `1433` `3306` `3389` |
| **Scan Origin** | Virtual machine installed on LAPTOP-MKT-03 |
| **Detection Source** | Suricata IDS |

---

## 3. Ports of Interest

| Port | Service | Relevance |
|---|---|---|
| 22 | SSH | Remote shell access |
| 80 / 443 | HTTP / HTTPS | Web services and management interfaces |
| 445 | SMB | File sharing — lateral movement via Pass-the-Hash / EternalBlue |
| 1433 | Microsoft SQL Server | Database access — direct relevance to production DB (`10.0.0.45`) |
| 3306 | MySQL | Secondary database services |
| 3389 | RDP | Remote Desktop — direct workstation and server access |

> The port selection is not random. This combination targets the exact services most relevant to lateral movement, credential relay, and data access within a Windows enterprise environment.

---

## 4. Timeline

| Time (approx.) | Event |
|---|---|
| T-unknown | Virtual machine installed on LAPTOP-MKT-03 |
| T-0 | Scanning activity initiated from VM on `LAPTOP-MKT-03` across `192.168.0.0/16` |
| T+unknown | 1,247 hosts contacted — ports 22/80/443/445/1433/3306/3389 probed |
| T+unknown | Suricata IDS alert triggered |
| T+unknown | Scanning activity ceased — connection dropped |
| T+unknown | Alert escalated — endpoint flagged for forensic review |

> **Note:** Duration of scanning activity and whether results were exfiltrated pending full Suricata log and endpoint forensic review.

---

## 5. Affected Assets

| Asset | Impact |
|---|---|
| `LAPTOP-MKT-03` | Source of scanning activity — endpoint integrity unconfirmed |
| Marketing segment | Credentials and local data on this host potentially accessible to threat actor |
| `192.168.0.0/16` subnet | Topology partially mapped by attacker — 1,247 hosts enumerated |
| Production DB (`10.0.0.45`) | Port 1433 scan suggests this host was a target of interest |
| RDP-enabled hosts | Port 3389 enumeration identifies potential lateral movement targets |

---

## 6. CIA Impact Assessment

| Pillar | Assessment |
|---|---|
| **Confidentiality** | **Medium** — Marketing segment credentials and local data potentially accessible; network topology now partially known to the threat actor |
| **Integrity** | **Low** — No evidence of data modification at this stage; scanning activity is passive reconnaissance |
| **Availability** | **Medium** — Attacker-controlled account could be used to modify marketing endpoint credentials; scan results may enable future targeted attacks against identified services |

---

## 7. False Positive Analysis

An authorized network administrator performing an internal audit or vulnerability scan from a non-standard endpoint was considered.

This hypothesis requires investigation but is **not strongly supported** by available evidence:

- `LAPTOP-MKT-03` is a marketing department endpoint — not an authorized platform for network scanning or IT administration.
- Scanning originated from a virtual machine, not the host OS — inconsistent with authorized administrative tooling, which would typically run on a managed IT endpoint.
- No change request or authorized scan window has been identified that covers this activity.
- The source IP `192.168.20.88` registering as a public IP for an internal host is anomalous and warrants explanation.

**Assessment: Requires investigation. Benign activity not ruled out — insider threat and external compromise both remain active hypotheses.**

---

## 8. Correlation

- **ALT-2024-004 → ALT-2024-005:** DNS tunneling activity targeting the production DB (`10.0.0.45`) in ALT-2024-004 may have been preceded by port 1433 reconnaissance observed here — suggesting this scan was used to identify the database server as a viable target.
- **ALT-2024-003 → ALT-2024-005:** Domain Admin access obtained via `svc_backup_adm` (ALT-2024-003) could have enabled installation of the virtual machine and scanning tooling on LAPTOP-MKT-03 remotely.
- **ALT-2024-005 → ALT-2024-001:** RDP and SMB port enumeration (3389, 445) is consistent with pre-ransomware reconnaissance to identify reachable endpoints for LockBit deployment.

> **Status:** Correlation under active investigation. This alert may represent the earliest observable phase of the broader attack chain across Liberty Co.'s environment.

---

## 9. Containment Actions Taken

- `LAPTOP-MKT-03` isolated from the network pending forensic review.
- Virtual machine image preserved for forensic analysis.
- Marketing segment user credentials flagged for mandatory rotation.
- Suricata ruleset updated to alert on future port sweep patterns originating from non-IT endpoints.

---

## 10. MITRE ATT&CK Mapping

| Technique ID | Technique Name | Observed Behavior |
|---|---|---|
| T1046 | Network Service Discovery | Systematic port scan across `192.168.0.0/16` targeting 1,247 hosts |
| T1018 | Remote System Discovery | Host enumeration across the internal subnet |
| T1059 | Command and Scripting Interpreter | Scanning tooling executed within guest VM environment |
| T1564.006 | Hide Artifacts: Run Virtual Instance | Virtual machine used to stage scanning activity and reduce host-level detection |
| T1078 | Valid Accounts | Marketing user credentials potentially used to maintain persistence on endpoint |

---

## 11. Recommendations

1. **Immediate:** Perform full forensic acquisition of `LAPTOP-MKT-03` — prioritize VM disk image, network logs, and recently installed software artifacts.
2. **Immediate:** Identify and disable any accounts whose credentials were cached on this endpoint.
3. **Short-term:** Enforce endpoint policy to block unauthorized virtualization software (VMware, VirtualBox, Hyper-V) on non-IT managed devices via GPO or MDM.
4. **Short-term:** Deploy network behavioral analytics capable of detecting internal port sweep patterns — alert threshold should be set significantly below 1,247 hosts.
5. **Long-term:** Implement network access control (NAC) to enforce device compliance before granting internal network access — non-compliant or unmanaged VMs should be blocked at the switch level.
6. **Long-term:** Review VLAN segmentation to ensure marketing endpoints cannot directly reach production DB, SQL, and RDP services without explicit firewall authorization.

---

## 12. Open Investigative Leads

- Identity of the individual who installed the virtual machine on LAPTOP-MKT-03 — insider threat hypothesis not yet ruled out.
- Whether scan results were exfiltrated from the VM before the connection dropped.
- Whether the public IP anomaly (`192.168.20.88`) indicates NAT misconfiguration, VPN split tunneling, or deliberate IP spoofing.
- Full list of hosts that responded to the scan — any responding hosts on sensitive segments require individual review.

---

*Report status: Active investigation. Document will be updated as additional evidence is collected.*
