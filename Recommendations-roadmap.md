# 10 — Recommendations, Roadmap & Lessons Learned
## Liberty Co. Security Engagement | Final Report

---

## Overview

This document consolidates the security recommendations derived from both phases of the Liberty Co. engagement — the infrastructure assessment (Sprint 1) and the incident response investigations (Sprint 2). Recommendations are prioritized by urgency and mapped to the specific findings that generated them.

---

## Executive Summary of Findings

The Liberty Co. engagement identified a coordinated intrusion that progressed through five distinct phases: phishing-based initial access, internal reconnaissance, Active Directory compromise, DNS-based data exfiltration, and ransomware deployment. The full attack chain exploited weaknesses in identity security, endpoint visibility, and outbound network controls.

Infrastructure controls implemented during Sprint 1 — specifically VLAN segmentation and SIEM deployment — demonstrably limited the blast radius of the LockBit ransomware to the accounting segment. Without these controls, lateral movement to additional segments would have been significantly easier for the threat actor.

---

## Recommendations

### Priority 1 — Immediate (0–30 days)

| # | Recommendation | Source Finding |
|---|---|---|
| 1.1 | Enforce phishing-resistant MFA (FIDO2 / hardware token) for all accounts — prioritize privileged users, finance, and HR | ALT-2024-002 |
| 1.2 | Implement Conditional Access policies blocking authentication from Anonymous IPs and non-compliant devices | ALT-2024-002 |
| 1.3 | Disable and audit all service accounts with Domain Admin membership — apply least privilege | ALT-2024-003 |
| 1.4 | Deploy Protected Users security group for all Tier 0 accounts to prevent credential caching and Pass-the-Hash | ALT-2024-003 |
| 1.5 | Block outbound DNS queries not destined for authorized internal resolvers — enforce DNS-over-resolver at perimeter | ALT-2024-004 |
| 1.6 | Restrict outbound internet access from database and server segments — no direct external DNS resolution | ALT-2024-004 |
| 1.7 | Enforce GPO / MDM policy blocking unauthorized virtualization software on non-IT endpoints | ALT-2024-005 |
| 1.8 | Conduct emergency credential rotation for all domain accounts exposed during the ALT-2024-003 compromise window | ALT-2024-003 |

---

### Priority 2 — Short-term (30–90 days)

| # | Recommendation | Source Finding |
|---|---|---|
| 2.1 | Deploy Microsoft Defender for Identity (MDI) with alerts for LSASS access, DCSync, and unauthorized group membership changes | ALT-2024-003 |
| 2.2 | Implement application allowlisting on all endpoints — block execution of unauthorized binaries | ALT-2024-001 |
| 2.3 | Deploy DNS security monitoring with query frequency anomaly detection and entropy-based subdomain analysis | ALT-2024-004 |
| 2.4 | Configure EDR behavioral rules to detect process name impersonation (e.g., `svchost` variants) | ALT-2024-001 |
| 2.5 | Integrate threat intelligence feeds to flag newly registered domains (NRDs under 30 days) in DNS query logs | ALT-2024-004 |
| 2.6 | Deploy network behavioral analytics to detect internal port sweep patterns below 100-host threshold | ALT-2024-005 |
| 2.7 | Implement network access control (NAC) to block non-compliant and unmanaged devices at the switch level | ALT-2024-005 |
| 2.8 | Establish and test a formal ransomware response playbook including legal notification thresholds and backup restoration procedures | ALT-2024-001 |

---

### Priority 3 — Long-term (90–180 days)

| # | Recommendation | Source Finding |
|---|---|---|
| 3.1 | Implement a tiered Active Directory model (Tier 0 / Tier 1 / Tier 2) to limit blast radius of future credential compromise | ALT-2024-003 |
| 3.2 | Deploy Privileged Access Workstations (PAW) — administrative tooling restricted to designated hardened endpoints | ALT-2024-003 |
| 3.3 | Conduct organization-wide security awareness training with emphasis on AiTM phishing and domain spoofing indicators | ALT-2024-002 |
| 3.4 | Review and harden VLAN segmentation rules — marketing and production DB segments should have no direct communication path | ALT-2024-005 |
| 3.5 | Implement SIEM use cases for impossible travel, token replay, and service account anomalies | ALT-2024-002, ALT-2024-004 |
| 3.6 | Establish a formal Vulnerability Management program with monthly scan cycles and tracked remediation SLAs | Sprint 1 |
| 3.7 | Conduct a tabletop exercise simulating the full attack chain observed in this engagement | All alerts |

---

## Remediation Roadmap

```
Month 1          Month 2          Month 3          Month 4-6
────────────     ────────────     ────────────     ────────────
MFA hardening    MDI deployment   AD tiering       PAW rollout
Conditional      App allowlist    DNS security     Tabletop
Access                            monitoring       exercise
Credential       EDR rules        Awareness        Vuln mgmt
rotation                          training         program
DNS controls     NAC rollout      SIEM use cases   VLAN review
Ransomware       Behavioral       NRD intel
playbook         analytics        feeds
```

---

## Security Posture: Before vs. After Engagement

| Control Area | Sprint 1 (Before) | Sprint 1 (After) | Sprint 2 Gap Identified |
|---|---|---|---|
| Network Segmentation | Flat network — no VLAN isolation | 5 VLANs implemented | Marketing → DB path still reachable |
| Perimeter Firewall | No Default Deny policy | Default Deny configured | Outbound DNS not fully restricted |
| SIEM | No centralized logging | Wazuh deployed | DNS anomaly use cases missing |
| Credential Security | No rotation policy | Credentials rotated | Service accounts over-privileged |
| Endpoint Protection | Microsoft Defender baseline | No change | EDR behavioral rules not configured |
| Identity Security | No MFA enforcement | Not addressed | AiTM-resistant MFA not deployed |
| Incident Response | No formal IR process | IR process initiated | Playbooks not formalized |

---

## Lessons Learned

### What worked
- **VLAN segmentation contained the ransomware** — LockBit encryption was limited to the accounting segment. The Sprint 1 segmentation investment had a direct, measurable impact on limiting damage.
- **SIEM correlation enabled multi-alert detection** — without Wazuh centralizing events, the credential dumping activity in ALT-2024-003 would likely have gone undetected until after additional damage occurred.
- **Escalation process functioned as designed** — all five alerts were triaged and escalated within the same detection window, enabling parallel investigation.

### What failed
- **Detection occurred after impact** — LockBit had already encrypted 847 files and DNS exfiltration was underway before the first alerts fired. Earlier detection controls at the identity and endpoint layers are required.
- **Service accounts were over-privileged** — `svc_backup_adm` and `svc_database` both had access levels inconsistent with their operational scope. Least privilege was not enforced.
- **MFA was insufficient against AiTM** — standard MFA was bypassed via session token interception. Phishing-resistant MFA was not in place.
- **Outbound traffic was under-monitored** — DNS tunneling generated 14,200 queries before an alert fired. Behavioral baselines for service accounts were not configured in the SIEM.
- **Marketing endpoint had no virtualization controls** — a non-IT device was able to install and run a VM undetected, enabling internal reconnaissance without triggering host-level alerts.

### Key takeaway
> The controls implemented in Sprint 1 were necessary but not sufficient. This engagement demonstrates that infrastructure hardening creates a foundation — it does not eliminate risk. Sustained security requires continuous monitoring, identity-first controls, and behavioral detection layered on top of network segmentation.

---

*This document represents the final deliverable of the Liberty Co. security engagement. All recommendations should be reviewed by the client's internal IT and legal teams prior to implementation.*
