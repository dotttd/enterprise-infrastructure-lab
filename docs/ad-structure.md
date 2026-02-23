# AD Structure

## Purpose

Active Directory in this lab is designed to simulate a small-to-medium enterprise infrastructure with centralized identity management, department-based access control, and tiered administrative separation.

The objectives of the AD design are:

- Provide centralized authentication via Domain Controller
- Separate users logically by department
- Implement group-based access control (RBAC)
- Support automated drive mapping via Group Policy
- Enforce tiered administrative model (Tier 0 / 1 / 2)
- Maintain scalability for future infrastructure expansion

**Domain Name:** `corp.local`  
**Domain Controller:** DC01 (192.168.200.10)

---

## OU Design

The lab uses a tiered Organizational Unit (OU) structure that reflects both departmental organization and administrative privilege separation.

### Current OU Structure

```text
corp.local
├── Domain Controllers
│   └── DC01
│
├── Tier0
│   (Domain-level admin accounts – admin.t0)
│
├── Tier1
│   └── Servers
│       └── FS01
│
├── Tier2
│   └── Workstations
│       └── WIN10
│
├── Users
│   ├── HR
│   │   └── hr.staff
│   ├── Finance
│   │   └── finance.staff
│   └── IT
│       └── it.admin
│
├── Admin
│   ├── admin.t0
│   ├── admin.t1
│   └── admin.t2
│
└── Groups
    ├── HR-Staff
    ├── Finance-Staff
    ├── IT-Admin
    ├── Tier0-Domain-Admins
    ├── Tier1-Server-Admins
    └── Tier2-Helpdesk-Admins
```

### OU Descriptions

| OU                   | Purpose                                                                                          |
| -------------------- | ------------------------------------------------------------------------------------------------ |
| `Domain Controllers` | Default Windows OU for DC01                                                                      |
| `Tier0`              | Placeholder for Tier0 scope; `admin.t0` placed in `Admin` OU                                     |
| `Tier1\Servers`      | Contains member servers; GPO `Tier1-Logon-Restriction` and `Tier1-Security-Baseline` linked here |
| `Tier2\Workstations` | Contains domain-joined workstations (WIN10)                                                      |
| `Users`              | Contains business user accounts, organized by department sub-OUs                                 |
| `Admin`              | Contains all dedicated administrative accounts (admin.t0, t1, t2)                                |
| `Groups`             | Contains all security groups used for RBAC and administrative delegation                         |

> **Key principle:** OUs are used for organization and GPO scoping — not for privilege assignment. All privileges are assigned through Security Group membership.

---

## Naming Convention

### User Accounts

Format: `department.username`

| Account         | Department              | OU Placement    |
| --------------- | ----------------------- | --------------- |
| `hr.staff`      | HR                      | `Users\HR`      |
| `finance.staff` | Finance                 | `Users\Finance` |
| `it.admin`      | IT                      | `Users\IT`      |
| `admin.t0`      | Administration (Tier 0) | `Admin`         |
| `admin.t1`      | Administration (Tier 1) | `Admin`         |
| `admin.t2`      | Administration (Tier 2) | `Admin`         |

### Security Groups

Format: `Department-Role` (business) / `TierN-RoleName` (administrative)

| Group                   | Type     | Scope  | Purpose                                 |
| ----------------------- | -------- | ------ | --------------------------------------- |
| `HR-Staff`              | Security | Global | File access + drive mapping for HR      |
| `Finance-Staff`         | Security | Global | File access + drive mapping for Finance |
| `IT-Admin`              | Security | Global | Full file access + drive mapping        |
| `Tier0-Domain-Admins`   | Security | Global | Nested into `Domain Admins`             |
| `Tier1-Server-Admins`   | Security | Global | Local admin on FS01, logon/RDP rights   |
| `Tier2-Helpdesk-Admins` | Security | Global | Delegated password reset on `Users` OU  |

### Servers

Format: `Role + Numeric Identifier`

| Server  | Role                              | IP                         |
| ------- | --------------------------------- | -------------------------- |
| `DC01`  | Domain Controller, DNS, DHCP      | 192.168.200.10 (Static)    |
| `FS01`  | File Server (Tier1 Member Server) | 192.168.200.20 (Static)    |
| `WIN10` | Domain Workstation (Tier2)        | DHCP (192.168.200.100–200) |

This naming model supports horizontal scaling (DC02, FS02, etc.).

---

## Group Policy Objects

| GPO Name                      | Linked To    | Type                   | Purpose                                      |
| ----------------------------- | ------------ | ---------------------- | -------------------------------------------- |
| `Default Domain Policy`       | `corp.local` | Computer + User        | Default domain settings                      |
| `Drive Mapping – Departement` | `OU=Users`   | User Configuration     | Auto-map drives by security group            |
| `Tier1-Logon-Restriction`     | `OU=Tier1`   | Computer Configuration | Restrict local + RDP logon to Tier1 accounts |
| `Tier1-Security-Baseline`     | `OU=Tier1`   | Computer Configuration | Audit policy, firewall, SMB hardening        |

---

## Implementation Notes

- AD DS, DNS, and DHCP are all hosted on DC01
- All user access is controlled via Security Groups — no direct permission assignments
- NTFS permissions on FS01 are aligned with AD group membership
- Drive mapping is automated via GPO with Item-Level Targeting based on security group
- Token validation tested using `whoami /groups`
- Policy validation tested using `gpresult /r` and `gpresult /r /scope computer`
- Domain authentication validated across two physical hosts (PC1 and PC2)
- Infrastructure evolved from single-host NAT virtualization to distributed multi-host physical LAN

---

## Design Principles Applied

- Group-Based Access Control (no direct user permissions)
- Departmental Segmentation via sub-OUs
- Tiered Administrative Model (Tier 0 / 1 / 2)
- Role Separation (Domain Controller vs Member Server vs Workstation)
- Least Privilege Principle
- Scalable OU model
- Centralized identity management
