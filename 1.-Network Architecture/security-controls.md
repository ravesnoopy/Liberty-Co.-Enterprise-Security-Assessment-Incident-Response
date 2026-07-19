# Security Controls — Implementation Report
## Liberty Co. Security Engagement | Sprint 1

---

## Overview

Following the identification of critical weaknesses in Liberty Co.'s original network architecture, five security controls were designed and implemented to reduce the attack surface, limit lateral movement, and improve security visibility across the environment.

---

## 1. VLAN Segmentation

**Problem identified:** All systems shared a flat network with unrestricted communication. A single compromised endpoint had direct access to every other device, including the production database.

**Control implemented:** The infrastructure was segmented into five VLANs separated by function and criticality level.

| VLAN | Zone | Device | IP Address |
|---|---|---|---|
| VLAN 1 | Internal — Security | Kali Linux | 10.160.210.210 |
| VLAN 2 | Internal — Operations | Ubuntu | 10.160.104.228 |
| VLAN 3 | Internal — Infrastructure | Windows Server 2019 | 10.160.69.139 |
| VLAN 4 | DMZ | Microsoft SQL Server | 10.160.201.57 |
| VLAN 5 | Monitoring | SIEM (Wazuh) | — |

**Impact:** Inter-VLAN traffic is now controlled exclusively through the firewall. A compromised host in VLAN 1 cannot directly reach VLAN 3 or the DMZ without explicit authorization. Lateral movement blast radius is contained per segment.

---

## 2. DMZ — Demilitarized Zone

**Problem identified:** Microsoft SQL Server shared the same network zone as all other systems, exposing the organization's most critical asset — client financial records, trading history, and account data — to unrestricted access from any internal endpoint.

**Control implemented:** SQL Server was relocated to a dedicated DMZ segment (VLAN 4), logically isolated from all other VLANs. Communication to and from the DMZ is restricted exclusively to authorized services and source addresses, enforced at the firewall level.

**Impact:** Direct access to the production database from general-purpose endpoints is no longer possible. Any connection attempt from an unauthorized source is blocked and logged by the firewall.

---

## 3. Firewall — Default Deny Policy

**Problem identified:** No enforced firewall policy existed between internal systems. Traffic flowed freely across the network without inspection or restriction, enabling unrestricted communication between all devices.

**Control implemented:** The perimeter and inter-VLAN firewall was configured with a **Default Deny** policy. All traffic is blocked by default. Only explicitly authorized communication paths are permitted.

**Authorized rules defined:**

- Inter-VLAN traffic permitted only where operationally required and documented.
- Access to DMZ (VLAN 4 — SQL Server) restricted to authorized source VLANs and ports only.
- All connection attempts — permitted and blocked — are logged and forwarded to the SIEM.
- Outbound internet access restricted to authorized services.

**Impact:** Attack surface significantly reduced. Unauthorized lateral movement attempts are now blocked at the network level and generate visible alerts in the SIEM.

---

## 4. SIEM — Centralized Security Monitoring

**Problem identified:** Security events were distributed across individual systems with no centralized collection or correlation capability. Suspicious activity could occur across the environment without generating any observable alert.

**Control implemented:** Wazuh SIEM was deployed in a dedicated monitoring segment (VLAN 5). All VLANs forward security events to the SIEM for centralized collection, correlation, and alerting.

**Log sources integrated:**

- Firewall connection and block events
- Windows Server 2019 — Windows Event Logs
- Microsoft SQL Server — authentication and query events
- Ubuntu and Kali Linux — system and authentication logs
- Network devices — inter-VLAN traffic logs

**Impact:** Full visibility over security events across all segments. Suspicious activity — including failed authentication attempts, inter-VLAN access violations, and anomalous query volumes — now generates correlated alerts. This capability was the foundation for all five incident investigations in Sprint 2.

---

## 5. Credential Hardening

**Problem identified:** No credential rotation policy existed prior to this engagement. Credentials across all systems had been in use for an undetermined period, with no validation of strength or exposure history. Credentials used in the pre-segmentation environment may have been intercepted or compromised.

**Control implemented:** A full credential rotation was performed across all user accounts and service accounts following the infrastructure redesign. New credentials were required to meet hardened complexity requirements.

**Actions taken:**

- All user account passwords reset and hardened across Windows Server 2019, Ubuntu, and Kali Linux.
- Service account credentials rotated — including database service accounts on SQL Server.
- Password policy enforced: minimum length, complexity, and expiration requirements applied.
- Default and unused accounts reviewed — unnecessary accounts disabled.

**Impact:** Risk of unauthorized access using previously exposed credentials eliminated. Service accounts operating under the new segmented architecture are scoped to minimum required privileges.

---

## Control Coverage Summary

| Control | Threat Mitigated | Status |
|---|---|---|
| VLAN Segmentation | Lateral movement, unrestricted internal access | Implemented |
| DMZ | Unauthorized database access, data exposure | Implemented |
| Default Deny Firewall | Unrestricted inter-system communication | Implemented |
| SIEM (Wazuh) | Lack of visibility, undetected suspicious activity | Implemented |
| Credential Hardening | Credential reuse, unauthorized access via exposed passwords | Implemented |

---

*Controls deployed as part of Sprint 1 infrastructure hardening — June 18, 2026.*
