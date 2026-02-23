# Enterprise Infrastructure Lab

> A distributed Windows Server enterprise lab environment built across two physical machines, simulating real-world IT infrastructure design, security hardening, and administrative governance.

---

## 🖥 Physical Architecture

```text
PC2 (Infrastructure Host)
  └── DC01 — 192.168.200.10  [Domain Controller, DNS, DHCP]
        │
        │  Direct Ethernet — 192.168.200.0/24
        │
PC1 (Application & Client Host)
  ├── FS01 — 192.168.200.20  [File Server, Tier1 Member Server]
  └── WIN10 — DHCP           [Domain Workstation, Tier2]

PC3 (Security & Analytics Host)
  └── WA_SIEM — 192.168.200.30  [Wazuh SIEM Manager]
```

✅ No home router dependency — dedicated internal enterprise LAN  
✅ Cross-host distributed simulation (2 physical machines)  
✅ All VMs bridged to physical NIC for real network communication

---

## 📋 Sub-Lab Index

| #   | Sub-Lab                                                                                      | Topic                                                              | Status         |
| --- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ | -------------- |
| 01  | [Virtualization Setup](./sublabs/01-virtualization-setup.md)                                 | Multi-host topology design, distributed VM architecture            | ✅ Complete    |
| 02  | [Active Directory Deployment](./sublabs/02-active-directory-deployment.md)                   | AD DS, DNS, domain join, cross-host authentication                 | ✅ Complete    |
| 03  | [DHCP & Network Redesign](./sublabs/03-dhcp-network-redesign.md)                             | Physical LAN migration, DHCP scope, IP scheme redesign             | ✅ Complete    |
| 04  | [File Server & RBAC](./sublabs/04-file-server-rbac.md)                                       | NTFS permissions, SMB hidden shares, security group access         | ✅ Complete    |
| 05  | [GPO Drive Mapping](./sublabs/05-gpo-drive-mapping.md)                                       | Group Policy Preferences, Item-Level Targeting, auto drive mapping | ✅ Complete    |
| 06  | [Tiered Admin Model](./sublabs/06-tiered-admin-model.md)                                     | Tier 0/1/2 separation, delegated administration, nested groups     | ✅ Complete    |
| 07  | [Security Hardening – Tier Enforcement](./sublabs/07-security-hardening-tier-enforcement.md) | GPO logon restriction, User Rights Assignment, tier validation     | ✅ Complete    |
| 08  | [Server Hardening Baseline](./sublabs/08-server-hardening-baseline.md)                       | Audit policy, RDP restriction, firewall, SMB hardening             | ✅ Complete    |
| 09  | [Centralized Event Log Monitoring](./sublabs/09-centralized-event-log-monitoring.md)         | Windows Event Forwarding, WEF collector/source setup               | ✅ Complete    |
| 10  | [Sysmon Integration](./sublabs/10-sysmon-integration-advanced-monitoring.md)                 | Advanced monitoring, EDR fundamentals, Sysmon + WEF                | ✅ Complete    |
| 11  | [Wazuh SIEM Integration](./sublabs/11-wazuh-siem-integration.md)                             | SIEM, Log Analytics, Zero-Trust Networking (Tailscale)             | 🚧 In Progress |

---

## 🗺 Project Connection Map

```text
[01 Virtualization]
        │
        ▼
[02 Active Directory] ──────────────> docs/ad-structure.md
        │
        ▼
[03 DHCP & Network] ─────────────────> docs/ip-design.md
        │
        ▼
[04 File Server & RBAC] ─────────────> docs/rbac-design.md
        │
        ▼
[05 GPO Drive Mapping]
        │
        ▼
[06 Tiered Admin Model] ─────────────> docs/ad-structure.md (Tier OU)
        │                               docs/rbac-design.md (Tier Access)
        ▼
[07 Security Hardening – Tier Enforcement]
        │
        ▼
[08 Server Hardening Baseline]
        │
        ▼
[09 Centralized Event Log Monitoring]
        │
        ▼
[10 Sysmon Integration (EDR)]
        │
        ▼
[11 Wazuh SIEM Integration] ────────> docs/siem-design.md (Upcoming)
```

---

## 🔐 Security Architecture Summary

### Access Control Layers

| Layer             | Mechanism                                                 | Scope                      |
| ----------------- | --------------------------------------------------------- | -------------------------- |
| NTFS Permissions  | Security Groups (`HR-Staff`, `Finance-Staff`, `IT-Admin`) | File system access on FS01 |
| GPO Drive Mapping | Item-Level Targeting by group membership                  | Automated drive assignment |
| Tier 0 Admin      | `Tier0-Domain-Admins` → `Domain Admins` (nested)          | Domain Controller only     |
| Tier 1 Admin      | `Tier1-Server-Admins` → FS01 Local Administrators         | Member Servers only        |
| Tier 2 Admin      | `Tier2-Helpdesk-Admins` → Delegated on Users OU           | Password reset only        |
| Logon Restriction | GPO `Tier1-Logon-Restriction` linked to `OU=Tier1`        | Console + RDP access       |
| Security Baseline | GPO `Tier1-Security-Baseline` linked to `OU=Tier1`        | Audit, Firewall, SMB       |

---

## 🛠 Engineering Challenges Solved

| Challenge                                     | Resolution                                                     |
| --------------------------------------------- | -------------------------------------------------------------- |
| DC01 memory contention on shared host         | Migrated DC01 to dedicated physical host (PC2)                 |
| VirtualBox NAT — no cross-host routing        | Redesigned to physical LAN with bridged NICs                   |
| IP scheme conflict after NAT removal          | Full migration from `10.10.x.x` to `192.168.200.0/24`          |
| Kerberos token not updated after group change | `klist purge` + full logoff/logon cycle                        |
| GPO drive not applying to correct users       | Debugged Item-Level Targeting AND vs OR logic                  |
| HR/Finance still able to log into FS01        | Explicitly defined Allow-only logon via User Rights Assignment |
| admin.t1 could login but not restart FS01     | Added Tier1-Server-Admins to `Shut down the system` URA        |
| Audit Policy conflicting with legacy settings | Enabled `Force audit policy subcategory settings` override     |
| RAM constraints for SIEM components           | Distributed lab to 3rd physical PC via **Tailscale SDN**       |

---

## 🏗 Reference Design Documents

| Document                                  | Description                                                            |
| ----------------------------------------- | ---------------------------------------------------------------------- |
| [ip-design.md](./docs/ip-design.md)       | Network topology, IP allocation, DHCP scope, DNS design                |
| [ad-structure.md](./docs/ad-structure.md) | OU hierarchy, Tier Model structure, naming conventions, GPO list       |
| [rbac-design.md](./docs/rbac-design.md)   | Full RBAC model — file access, drive mapping, tiered admin permissions |

---

## 🎯 Skills Demonstrated

**Infrastructure & Networking**

- Distributed multi-host virtualization architecture
- Physical LAN design and bridged networking
- DHCP & DNS deployment and troubleshooting

**Identity & Access Management**

- Active Directory Domain Services deployment
- Organizational Unit design (Tier-based model)
- Security group design and nested group strategy
- Role-Based Access Control (NTFS + SMB + AD)
- Delegated Administration (ADUC Delegation Wizard)

**Group Policy**

- GPO creation, linking, and scoping
- User Configuration vs Computer Configuration policy types
- Item-Level Targeting for dynamic drive assignment
- User Rights Assignment configuration

**Security Hardening**

- Tiered Administrative Model (Microsoft ESAE principle)
- Logon restriction enforcement via GPO
- Advanced Audit Policy configuration
- Windows Defender Firewall enforcement
- SMBv1 protocol hardening
- Anonymous enumeration restriction

**Troubleshooting & Validation**

- `gpresult /r /scope computer`, `whoami /groups`, `klist purge`
- Event Viewer — Security log (Event ID 4624, 4625, 4672)
- Cross-host connectivity testing (`ping`, `Test-NetConnection`)
- Live login testing per account tier

---

## 🚀 Current Status

**Completed (Sub-Lab 01–10):** Distributed lab environment with full AD, DHCP, RBAC, GPO, Tiered Admin Model, Security Hardening, and Sysmon EDR telemetry.

**In Progress (Sub-Lab 11):** Wazuh SIEM Integration — Implementing centralized analytics and Zero-Trust networking via Tailscale.

---

## 📁 Repository Structure

```text
enterprise-infrastructure-lab/
├── README.md
├── sublabs/
│   ├── 01-virtualization-setup.md
│   ├── 02-active-directory-deployment.md
│   ├── 03-dhcp-network-redesign.md
│   ├── 04-file-server-rbac.md
│   ├── 05-gpo-drive-mapping.md
│   ├── 06-tiered-admin-model.md
│   ├── 07-security-hardening-tier-enforcement.md
│   ├── 08-server-hardening-baseline.md
│   ├── 09-centralized-event-log-monitoring.md
│   ├── 10-sysmon-integration-advanced-monitoring.md
│   └── 11-wazuh-siem-integration.md
├── docs/
│   ├── ad-structure.md
│   ├── ip-design.md
│   ├── rbac-design.md
│   └── screenshots/
│       ├── 01-virtualization/
│       ├── 02-active-directory/
│       ├── 03-dhcp-network/
│       ├── 04-file-server-rbac/
│       ├── 05-gpo-drive-mapping/
│       ├── 06-tiered-admin-model/
│       ├── 07-security-hardening-tier-enforcement/
│       ├── 08-server-hardening-baseline/
│       ├── 09-centralized-event-log-monitoring/
│       └── 10-sysmon-integration-advanced-monitoring/
```
