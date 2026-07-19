# Incident Report — ALT-2024-002
## Phishing / Adversary-in-the-Middle / Credential Theft

| Field | Detail |
|---|---|
| **Alert ID** | ALT-2024-002 |
| **Date** | 2024-06-22 |
| **Analyst** | Leonardo Romero Garcia |
| **Severity** | High |
| **Status** | Contained — Under Investigation |
| **Alert Source** | Microsoft Defender / Azure AD |
| **NIST Phase** | Detection → Containment → Investigation |

---

## 1. Executive Summary

Microsoft Defender and Azure AD generated alerts following anomalous authentication activity associated with the account **jmendez@libertyCo.com**. Analysis indicates the user was targeted by a phishing campaign leveraging an Adversary-in-the-Middle (AiTM) framework (`evilginx2`) hosted at `45.142.212.100`, designed to intercept session tokens and bypass multi-factor authentication. Evidence indicates successful credential and session token compromise, followed by unauthorized access to HR file storage and lateral movement into Active Directory.

The false positive probability is assessed as **low**.

---

## 2. Alert Details

| Field | Value |
|---|---|
| **Compromised Account** | `jmendez@libertyCo.com` |
| **User Endpoint IP** | `189.203.45.112` |
| **Attacker Infrastructure** | `45.142.212.100` |
| **AiTM Framework** | `evilginx2` |
| **Phishing URL** | `hxxps://microsoft-365-login[.]com/empresa/auth` |
| **Sender (Phishing Email)** | `it-soporte@micro5oft[.]com` |
| **Risk Flags** | Impossible Travel, Anonymous IP, Replicated Session Token |
| **Data Accessed** | `RRHH/Nomina_2024.xlsx` (4.2 MB exfiltrated) |

---

## 3. Timeline

| Time (approx.) | Event |
|---|---|
| T-2 days | Phishing email delivered to `jmendez@` from `it-soporte@micro5oft[.]com` |
| T-0 | User interacts with phishing URL — session token intercepted via `evilginx2` |
| T+unknown | Attacker replays session token from `45.142.212.100` — MFA bypassed |
| T+unknown | Azure AD flags Impossible Travel and Anonymous IP risk signals |
| T+unknown | Unauthorized access to HR storage — `RRHH/Nomina_2024.xlsx` exfiltrated (4.2 MB) |
| T+unknown | Lateral movement into Active Directory observed |
| T+unknown | Alert escalated — credential containment initiated |

> **Note:** Precise timestamps pending full Azure AD sign-in log and Defender audit trail review.

---

## 4. Affected Assets

| Asset | Impact |
|---|---|
| `jmendez@` credentials | Compromised — session token replicated |
| HR file server | `RRHH/Nomina_2024.xlsx` confirmed exfiltrated |
| Active Directory | Unauthorized access observed |
| User endpoint | Assessed as initial phishing target |

---

## 5. CIA Impact Assessment

| Pillar | Assessment |
|---|---|
| **Confidentiality** | **High** — Employee payroll data (`Nomina_2024.xlsx`, 4.2 MB) confirmed exfiltrated |
| **Integrity** | **High** — Compromised account active in AD; unauthorized changes cannot be ruled out |
| **Availability** | **Medium** — Account at risk of lockout or credential replacement by attacker if containment is delayed |

---

## 6. False Positive Analysis

Legitimate travel or VPN usage by the user was considered as an alternative explanation for the geographic anomaly.

This hypothesis is **not supported** by available evidence:

- Azure AD flagged both **Impossible Travel** and **Anonymous IP** simultaneously — consistent with attacker VPN/proxy usage, not legitimate user activity.
- The phishing domain `microsoft-365-login[.]com` and sender `micro5oft[.]com` are confirmed malicious infrastructure.
- `evilginx2` is an offensive AiTM framework with no legitimate enterprise use case.
- Session token replay from a separate IP confirms the credential was weaponized, not merely observed.

**Assessment: False positive ruled out.**

---

## 7. Correlation

Evidence indicates a probable relationship with **ALT-2024-003** (credential dumping / privilege escalation). The unauthorized Domain Admin account (`svc_backup_adm`) created during ALT-2024-003 may have been facilitated by access obtained through the compromised `jmendez@` session.

> **Status:** Correlation under active investigation. Timeline alignment between both incidents is pending.

---

## 8. Containment Actions Taken

- `jmendez@` account suspended pending investigation.
- All active sessions for the account revoked in Azure AD.
- HR file server access restricted to authorized personnel only.
- Phishing domain and attacker IP (`45.142.212.100`) submitted for blocking at perimeter firewall and DNS filtering layer.

---

## 9. MITRE ATT&CK Mapping

| Technique ID | Technique Name | Observed Behavior |
|---|---|---|
| T1566.002 | Phishing: Spearphishing Link | Malicious link delivered via email impersonating IT support |
| T1557 | Adversary-in-the-Middle | `evilginx2` used to intercept credentials and session tokens |
| T1539 | Steal Web Session Cookie | Session token replayed to bypass MFA |
| T1078 | Valid Accounts | Attacker authenticated using legitimate `jmendez@` session |
| T1021 | Remote Services | Lateral movement into Active Directory via compromised account |
| T1041 | Exfiltration Over C2 Channel | `RRHH/Nomina_2024.xlsx` exfiltrated (4.2 MB) |

---

## 10. Recommendations

1. **Immediate:** Force password reset and re-enrollment of MFA for `jmendez@` before reactivating the account.
2. **Immediate:** Audit all AD changes made during the window the compromised session was active.
3. **Short-term:** Enable phishing-resistant MFA (FIDO2 / hardware token) for all privileged and finance-adjacent accounts.
4. **Short-term:** Deploy Microsoft Defender for Identity to detect token replay and lateral movement patterns in real time.
5. **Long-term:** Implement user security awareness training with emphasis on AiTM phishing techniques and domain spoofing indicators.
6. **Long-term:** Configure Conditional Access policies to block authentication from Anonymous IPs and enforce compliant device requirements.

---

## 11. Open Investigative Leads

- Full scope of AD access during the compromised session window not yet determined.
- Whether additional files beyond `Nomina_2024.xlsx` were accessed or exfiltrated — pending HR server access log review.
- Causal relationship between this incident and ALT-2024-003 pending timeline correlation.
- Origin and delivery method of the initial phishing email — inbox rule tampering not yet ruled out.

---

*Report status: Active investigation. Document will be updated as additional evidence is collected.*
