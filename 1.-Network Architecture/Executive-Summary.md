# Executive Summary
## Liberty Co. Security Engagement | Sprint 1

---

## Engagement Scope

This report summarizes the findings, decisions, and outcomes of the Sprint 1 infrastructure assessment conducted for Liberty Co. The objective was to evaluate the existing network architecture, identify critical security weaknesses, and implement foundational controls to reduce the organization's attack surface prior to SOC monitoring activation.

---

## Critical Asset Identified

During the assessment, **Microsoft SQL Server (10.160.201.57)** was identified as the highest-priority asset in the environment.

This system stores client financial records, trading history, account data, and regulatory documentation — all classified as Restricted information under Liberty Co.'s data classification policy. In the original flat network configuration, this server was directly reachable from every other endpoint without restriction.

A threat actor with access to any device on the network could query, modify, or destroy the contents of this database without encountering any network-level control. The confidentiality, integrity, and availability of Liberty Co.'s most sensitive data were fully dependent on host-level controls alone — with no network segmentation or access restriction in place.

---

## Weaknesses Identified

| Weakness | Risk | Severity |
|---|---|---|
| Flat network — no segmentation | Any compromised host has unrestricted access to all systems including SQL Server | Critical |
| No DMZ | Production database exposed on the same network as general-purpose endpoints | Critical |
| No Default Deny firewall policy | Traffic flowed freely between all systems — no inter-device access control | High |
| No centralized logging | Suspicious activity across the environment generated no observable alert | High |
| No credential rotation policy | Credentials of unknown age and exposure history in use across all systems | High |

---

## Controls Implemented

| Control | Objective |
|---|---|
| 5 VLANs | Segment devices by function — contain lateral movement blast radius per zone |
| DMZ (VLAN 4) | Isolate SQL Server — restrict access to authorized services only |
| Default Deny Firewall | Block all unauthorized inter-VLAN communication — log all connection attempts |
| Wazuh SIEM | Centralize security events across all segments — enable real-time correlation and alerting |
| Credential rotation | Eliminate risk of unauthorized access via previously exposed credentials |

---

## Outcome

Following the implementation of these controls, Liberty Co.'s network transitioned from a flat, unrestricted architecture to a segmented environment with defined trust boundaries, enforced access control, and centralized security visibility.

The direct impact of these controls was demonstrated during Sprint 2: when LockBit 3.0 ransomware was deployed against the accounting segment, VLAN segmentation prevented the payload from spreading to additional network zones. Without this control, lateral movement to the SQL Server and other critical segments would have been significantly easier for the threat actor.

---

## Future Recommendations

| Recommendation | Priority |
|---|---|
| Enforce phishing-resistant MFA across all accounts | High |
| Deploy Microsoft Defender for Identity for AD monitoring | High |
| Implement application allowlisting on all endpoints | High |
| Conduct periodic firewall rule audits — apply minimum privilege | Medium |
| Establish a formal credential rotation schedule (90-day cycle) | Medium |
| Perform regular SIEM use case reviews — add detection rules as environment evolves | Medium |
| Conduct annual tabletop exercises simulating realistic attack scenarios | Medium |

---

## Conclusion

The Sprint 1 assessment confirmed that Liberty Co.'s original infrastructure presented multiple critical weaknesses that would have enabled a threat actor to achieve full environment compromise following a single successful initial access. The controls implemented during this sprint established the security foundation required to detect, contain, and investigate the incidents documented in Sprint 2.

Security is an ongoing process. The recommendations above represent the next layer of controls required to maintain and improve Liberty Co.'s security posture beyond this engagement.

---

*Assessment conducted and controls implemented: June 18, 2026.*
