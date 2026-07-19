# Incident Report — ALT-2024-004
## Anomalous Database Activity / DNS Tunneling / Data Exfiltration

| Field | Detail |
|---|---|
| **Alert ID** | ALT-2024-004 |
| **Date** | 2024-06-22 |
| **Analyst** | Leonardo Romero Garcia |
| **Severity** | High |
| **Status** | Contained — Under Investigation |
| **Alert Source** | Palo Alto NGFW |
| **NIST Phase** | Detection → Containment → Investigation |

---

## 1. Executive Summary

Palo Alto NGFW generated an alert following anomalous outbound DNS query volume originating from Liberty Co.'s production database server (`10.0.0.45`), associated with the service account `svc_database`. Analysis indicates a query rate of **14,200 DNS requests within 45 minutes** against a recently registered external domain — a pattern consistent with **DNS tunneling** used as a covert data exfiltration channel. The destination IP (`104.21.45.88`) resolves to a domain registered 8 days prior to the alert, with subdomains encoded in Base64 format observed within the query strings.

Evidence suggests this activity may be part of the broader attack chain initiated in ALT-2024-003, leveraging Domain Admin access to instrument the database server as an exfiltration point.

---

## 2. Alert Details

| Field | Value |
|---|---|
| **Source Host** | Production DB Server — `10.0.0.45` |
| **Source Account** | `svc_database` (non-interactive service account) |
| **Destination IP** | `104.21.45.88` |
| **Domain Age** | Registered 8 days prior to alert |
| **Query Volume** | 14,200 queries in 45 minutes |
| **Baseline** | 2–5 queries per hour |
| **Anomaly Factor** | ~3,000x above baseline |
| **Encoding Observed** | Base64 subdomains in DNS query strings |
| **Detection Source** | Palo Alto NGFW |

---

## 3. Timeline

| Time (approx.) | Event |
|---|---|
| T-8 days | Attacker registers destination domain (assessed as attacker-controlled infrastructure) |
| T-0 | `svc_database` begins anomalous DNS query activity from `10.0.0.45` |
| T+45 min | 14,200 DNS queries recorded — Palo Alto NGFW alert triggered |
| T+unknown | Base64-encoded subdomains identified in query strings |
| T+unknown | Destination IP `104.21.45.88` flagged — domain registration correlated |
| T+unknown | Alert escalated — outbound DNS to destination blocked |

> **Note:** Precise start time of tunneling activity and total data volume exfiltrated pending full NGFW and SIEM log review.

---

## 4. Affected Assets

| Asset | Impact |
|---|---|
| Production DB Server (`10.0.0.45`) | Primary source of anomalous DNS activity |
| `svc_database` account | Instrumentalized as exfiltration account — compromise assessed |
| Database contents | Scope of exfiltrated data not yet confirmed — under investigation |
| DNS infrastructure | Abused as covert exfiltration channel |
| Credential store | Valid service account credentials used — source of compromise under investigation |

---

## 5. CIA Impact Assessment

| Pillar | Assessment |
|---|---|
| **Confidentiality** | **High** — DNS tunneling pattern consistent with active data exfiltration; database contents assessed as at risk. Base64-encoded subdomains suggest structured data extraction |
| **Integrity** | **Medium** — Unauthorized use of `svc_database` account; database integrity cannot be confirmed until forensic review is complete |
| **Availability** | **Medium** — Domain may be rendered unavailable by sustained tunneling traffic volume; service account credentials at risk of being rotated by attacker |

---

## 6. False Positive Analysis

Legitimate enterprise application traffic or a misconfigured DNS resolver generating high query volume was considered.

This hypothesis is **not supported** by available evidence:

- A query rate of 14,200 per 45 minutes represents approximately 3,000x the documented baseline for this host — no authorized application generates traffic at this volume.
- The destination domain was registered 8 days prior to the alert — inconsistent with any established business relationship or authorized service.
- Base64-encoded subdomains within DNS query strings are not characteristic of legitimate DNS resolution — this encoding pattern is a well-documented indicator of DNS tunneling tooling (e.g., `iodine`, `dnscat2`).
- `svc_database` is a non-interactive service account — anomalous outbound network behavior from this account warrants investigation regardless of volume.

**Assessment: False positive ruled out.**

---

## 7. Correlation

- **ALT-2024-003 → ALT-2024-004:** Domain Admin access obtained via `svc_backup_adm` (ALT-2024-003) is assessed as the probable mechanism used to instrument `svc_database` on the production server.
- **ALT-2024-001 → ALT-2024-004:** LockBit C2 activity (ALT-2024-001) may share infrastructure or operational overlap with the DNS tunneling channel observed here — pending IOC cross-reference.
- **ALT-2024-004 → ALT-2024-005:** The internal network scanning activity observed in ALT-2024-005 may have been used to identify `10.0.0.45` as a viable exfiltration point prior to DNS tunneling deployment.

> **Status:** Correlation under active investigation. Attack chain sequencing requires full timestamp alignment across all five alerts.

---

## 8. Containment Actions Taken

- Outbound DNS traffic to `104.21.45.88` and associated domain blocked at Palo Alto NGFW.
- `svc_database` account suspended pending forensic review.
- Production DB server (`10.0.0.45`) isolated from external network segments.
- DNS query logs exported from NGFW for IOC extraction and threat intelligence correlation.

---

## 9. MITRE ATT&CK Mapping

| Technique ID | Technique Name | Observed Behavior |
|---|---|---|
| T1071.004 | Application Layer Protocol: DNS | DNS used as covert communication and exfiltration channel |
| T1048.001 | Exfiltration Over Alternative Protocol: DNS | Data exfiltrated via Base64-encoded DNS query subdomains |
| T1078.003 | Valid Accounts: Local Accounts | `svc_database` service account used for unauthorized activity |
| T1583.001 | Acquire Infrastructure: Domains | Attacker-registered domain used as DNS tunneling endpoint |
| T1041 | Exfiltration Over C2 Channel | DNS tunnel assessed as active C2 and exfiltration channel |

---

## 10. Recommendations

1. **Immediate:** Block all outbound DNS queries not destined for authorized internal resolvers — enforce DNS-over-resolver architecture at the perimeter.
2. **Immediate:** Rotate credentials for `svc_database` and audit all activity performed under this account during the compromise window.
3. **Short-term:** Deploy DNS security monitoring capable of detecting query frequency anomalies and Base64/entropy-based subdomain patterns (e.g., Palo Alto DNS Security, Cisco Umbrella).
4. **Short-term:** Restrict outbound internet access from database servers — production DB servers should have no direct external DNS resolution capability.
5. **Long-term:** Implement network segmentation controls that prevent service accounts from initiating outbound connections outside their defined operational scope.
6. **Long-term:** Integrate threat intelligence feeds to flag newly registered domains (NRDs) in DNS query logs — domains under 30 days old should trigger automatic review.

---

## 11. Open Investigative Leads

- Total volume and content of data exfiltrated via DNS tunnel not yet quantified.
- DNS tunneling tool used on the server-side not yet identified — artifact recovery from `10.0.0.45` pending.
- Whether additional hosts within the `10.0.0.0/24` segment were used as secondary exfiltration points — pending NGFW log review.
- Infrastructure overlap between `104.21.45.88` and C2 domain observed in ALT-2024-001 (`lockbit3-decryptor[.]onion`) — pending threat intelligence correlation.

---

*Report status: Active investigation. Document will be updated as additional evidence is collected.*
