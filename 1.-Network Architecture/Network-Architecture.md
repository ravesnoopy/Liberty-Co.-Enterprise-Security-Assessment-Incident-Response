# Network Architecture — Before & After
## Liberty Co. Security Engagement | Sprint 1

---

## Before — Flat Network / No Segmentation

![Before](images/before.png)

| Device | Role | IP |
|---|---|---|
| Kali Linux | Security testing endpoint | — |
| Ubuntu | General purpose server | — |
| Windows Server 2019 | Domain Controller / File Server | — |
| Microsoft SQL Server | Production database | 10.160.201.57 |
| SIEM | Log collection | — |

**Critical weaknesses identified:**

- All devices shared the same logical network — no isolation between systems.
- Microsoft SQL Server was reachable from every endpoint without restriction.
- No firewall policy enforced between internal devices.
- Unrestricted lateral movement possible from any compromised host.
- No centralized visibility over security events.

---

## After — Segmented Network / DMZ / Default Deny Firewall

![After](images/after.png)

| VLAN | Zone | Device | IP |
|---|---|---|---|
| VLAN 1 | Internal | Kali Linux | 10.160.210.210 |
| VLAN 2 | Internal | Ubuntu | 10.160.104.228 |
| VLAN 3 | Internal | Windows Server 2019 | 10.160.69.139 |
| VLAN 4 | DMZ | Microsoft SQL Server | 10.160.201.57 |
| VLAN 5 | Monitoring | SIEM (Wazuh) | — |

**Controls implemented:**

- 5 VLANs separate devices by function and criticality.
- Microsoft SQL Server isolated in DMZ (VLAN 4) — only authorized connections permitted.
- Firewall configured with Default Deny — all inter-VLAN traffic explicitly authorized.
- SIEM (Wazuh) deployed in dedicated segment — receives logs from all VLANs.
- Credentials rotated across all systems following segmentation deployment.

---

## Comparative Analysis

| Area | Before | After |
|---|---|---|
| **Network topology** | Flat — unrestricted communication between all devices | 5 VLANs — traffic isolated by function and criticality |
| **SQL Server exposure** | Reachable from any host on the network | Isolated in DMZ — only authorized services can connect |
| **Firewall policy** | No Default Deny — traffic flowed freely between systems | Default Deny — only explicitly authorized traffic permitted |
| **Security visibility** | No centralized logging — events distributed and unmonitored | Wazuh SIEM centralizes all events — real-time correlation enabled |
| **Lateral movement risk** | High — any compromised host could reach all other systems | Low — VLAN segmentation contains blast radius per segment |
| **Credential security** | No rotation policy — credentials potentially exposed | Full rotation applied — hardened passwords across all accounts |

---

*Diagrams produced as part of Sprint 1 infrastructure assessment — June 18, 2026.*
