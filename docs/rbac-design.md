# RBAC Design

## Purpose

This document describes the Role-Based Access Control (RBAC) model implemented across the enterprise lab environment.

The RBAC design covers three distinct layers:

1. **File System Access** — department-based folder permissions on FS01 (Sub-Lab 04)
2. **Group Policy Access** — automated drive assignment by security group (Sub-Lab 05)
3. **Administrative Access** — tiered privilege separation for administrative accounts (Sub-Lab 06–08)

---

## Layer 1 – File System Access (FS01)

### Share Structure

Hidden administrative shares are used (`$` suffix) to prevent casual browsing.

| Share Name | UNC Path          | Department         |
| ---------- | ----------------- | ------------------ |
| `HR$`      | `\\FS01\HR$`      | HR Department      |
| `Finance$` | `\\FS01\Finance$` | Finance Department |
| `IT$`      | `\\FS01\IT$`      | IT Department      |

### Permission Matrix

NTFS permissions are assigned via Security Groups only — no direct user assignments.

| Security Group  | HR$          | Finance$     | IT$          |
| --------------- | ------------ | ------------ | ------------ |
| `HR-Staff`      | Read/Write   | ❌ No Access | ❌ No Access |
| `Finance-Staff` | ❌ No Access | Read/Write   | ❌ No Access |
| `IT-Admin`      | Read/Write   | Read/Write   | Read/Write   |

Share permissions: `Everyone: Read` (access controlled at NTFS layer).

### Validation

- `hr.staff` → access confirmed on `HR$`, denied on `Finance$` with error: _"Windows cannot access \\192.168.200.20\Finance$"_
- IT admin → simultaneously accessed `IT$`, `HR$`, and `Finance$`

---

## Layer 2 – Drive Mapping (GPO)

Network drives are automatically mapped based on security group membership using GPO Preferences with Item-Level Targeting.

| Security Group  | Drive Letter     | Label               | Target Share      |
| --------------- | ---------------- | ------------------- | ----------------- |
| `HR-Staff`      | `H:`             | HR DRIVE            | `\\FS01\HR$`      |
| `Finance-Staff` | `F:`             | Finance DRIVE       | `\\FS01\Finance$` |
| `IT-Admin`      | `H:`, `F:`, `I:` | HR/Finance/IT DRIVE | All shares        |

GPO Name: `Drive Mapping – Departement`  
GPO linked to: `OU=Users`  
GPO type: User Configuration

Item-Level Targeting uses Security Group membership for drive scoping.

---

## Layer 3 – Administrative Access (Tiered Model)

Administrative access is separated into three tiers to enforce least privilege.

### Tier Definitions

| Tier   | Account    | Group                                   | Access Scope                    |
| ------ | ---------- | --------------------------------------- | ------------------------------- |
| Tier 0 | `admin.t0` | `Tier0-Domain-Admins` → `Domain Admins` | Domain Controller only          |
| Tier 1 | `admin.t1` | `Tier1-Server-Admins`                   | Member Servers (FS01)           |
| Tier 2 | `admin.t2` | `Tier2-Helpdesk-Admins`                 | Password reset on Users OU only |

### Access Validation Matrix

| Account         | Login DC01 | Login FS01 | RDP FS01 | Reset Password | Domain Admin |
| --------------- | ---------- | ---------- | -------- | -------------- | ------------ |
| `admin.t0`      | ✅ YES     | ✅ YES     | ✅ YES   | ✅ YES         | ✅ YES       |
| `admin.t1`      | ❌ NO      | ✅ YES     | ✅ YES   | ❌ NO          | ❌ NO        |
| `admin.t2`      | ❌ NO      | ❌ NO      | ❌ NO    | ✅ YES         | ❌ NO        |
| `hr.staff`      | ❌ NO      | ❌ NO      | ❌ NO    | ❌ NO          | ❌ NO        |
| `finance.staff` | ❌ NO      | ❌ NO      | ❌ NO    | ❌ NO          | ❌ NO        |

### Enforcement Mechanisms

- **Logon restriction** — GPO `Tier1-Logon-Restriction` linked to `OU=Tier1`, User Rights Assignment: `Allow log on locally` and `Allow log on through Remote Desktop Services` scoped to Tier1 accounts only
- **Delegation** — ADUC Delegation Wizard scoped to `OU=Users`, rights limited to password reset
- **Nested group chain** — `admin.t0 → Tier0-Domain-Admins → Domain Admins`
- **Local admin** — `Tier1-Server-Admins` added to Local Administrators group on FS01

---

## Security Baseline (FS01)

Applied via GPO `Tier1-Security-Baseline` linked to `OU=Tier1`:

| Control                | Configuration                                                 | Status      |
| ---------------------- | ------------------------------------------------------------- | ----------- |
| Advanced Audit Logging | Logon Success/Failure (Event 4624/4625), Special Logon (4672) | ✅ Enabled  |
| RDP Access             | Restricted to `Administrators` + `Tier1-Server-Admins`        | ✅ Enforced |
| Windows Firewall       | Domain Profile enabled                                        | ✅ Active   |
| SMBv1                  | Feature not installed                                         | ✅ Disabled |
| Anonymous Enumeration  | `Do not allow anonymous enumeration of SAM accounts`          | ✅ Enabled  |

---

## Design Principles

- All permissions assigned through Security Groups — never directly to individual users
- Deny-based restrictions avoided in favor of Allow-only model to prevent override conflicts
- Tier separation enforced through both group structure and Group Policy
- Administrative accounts are separate from business user accounts
- Domain Admin accounts are never used for day-to-day operations
