# Sub-Lab 06 – Tiered Administrative Model

## 📌 Objective

Implement a Tiered Administrative Model (Tier 0/1/2) to separate administrator privileges based on system sensitivity level.

Key objectives:

- Prevent the use of Domain Admin accounts for day-to-day operations
- Enforce the principle of least privilege
- Implement delegated administration for helpdesk staff
- Establish clear boundaries between Domain Controllers, Member Servers, and Workstations

---

## 🏗 Environment Context

Domain: corp.local  
Domain Controller: DC01 (192.168.200.10)  
File Server: FS01 (192.168.200.20)  
Workstation: WIN10 (192.168.200.100)

---

## 🧱 OU Structure Redesign (Tier Model)

The OU structure was redesigned into the following Tier-based hierarchy:

```text
corp.local
|-- Domain Controllers
|   |-- DC01
|-- Tier0
|-- Tier1
|   |-- Servers
|   |   |-- FS01
|-- Tier2
|   |-- Workstations
|   |   |-- WIN10
|-- Users
|   |-- HR
|   |-- Finance
|   |-- IT
|-- Admin
|   |-- admin.t0
|   |-- admin.t1
|   |-- admin.t2
|-- Groups
|   |-- Tier0-Domain-Admins
|   |-- Tier1-Server-Admins
|   |-- Tier2-Helpdesk-Admins
```

Computer objects relocated:

- `FS01` moved to `Tier1\Servers`
- `WIN10` moved to `Tier2\Workstations`

---

## 👤 Dedicated Administrative Accounts

Administrative accounts are kept separate from business user accounts:

- `admin.t0`: Domain-level administration
- `admin.t1`: Member server administration
- `admin.t2`: Helpdesk / password reset

All admin accounts are placed under the `Admin` OU.

---

## 🔐 Security Groups (Role-Based Access)

Security groups created with Global scope:

- `Tier0-Domain-Admins`: Domain-level privilege
- `Tier1-Server-Admins`: Member server local admin
- `Tier2-Helpdesk-Admins`: Delegated helpdesk privilege

All groups are stored in the `Groups` OU.

---

## 🧩 Nested Group for Domain Admin (Tier0)

Best practice: users are never added directly to `Domain Admins`.

Nested group chain:

```text
admin.t0
  -> Tier0-Domain-Admins
     -> Domain Admins
```

Result:

- `admin.t0` receives Domain Admin privilege through group nesting
- No business user account has Domain Admin membership

---

## 🛠 Delegated Administration (Tier2)

Delegation on the `Users` OU was granted to:

- `Tier2-Helpdesk-Admins`

Rights delegated:

- Reset user password
- Force password change at next logon
- Unlock account

Validation results:

- `admin.t2` can reset passwords for HR/Finance users
- `admin.t2` cannot modify `Domain Admins` group
- `admin.t2` cannot create new OUs
- `admin.t2` cannot edit GPOs
- Helpdesk accounts cannot log into the Domain Controller

---

## 🖥 Server-Level Administration (Tier1)

On `FS01`, the group `Tier1-Server-Admins` was added to:

- Local Administrators group (FS01)

Effect:

- `admin.t1` can log into `FS01`
- `admin.t1` has local admin rights on `FS01`
- `admin.t1` does not have Domain Admin privilege
- `admin.t1` cannot log into `DC01`

---

## 🔍 Access Validation Matrix

```text
Account     | Login DC | Login FS01 | Reset User | Domain Admin
admin.t0    | YES      | YES        | YES        | YES
admin.t1    | NO       | YES        | NO         | NO
admin.t2    | NO       | NO         | YES        | NO
HR User     | NO       | NO         | NO         | NO
```

---

## ✅ Security Model Achieved

This sub-lab successfully implemented:

- Tiered Administrative Model
- Least Privilege Principle
- Role-Based Access Control (RBAC)
- Nested Group Strategy
- Delegated Administration
- Separation of Duties

---

## 🧠 Key Learning Outcomes

- OUs are not for privilege enforcement — they are for organization and GPO scoping
- Permissions must be assigned via groups, never directly to individual users
- Domain Admin accounts must not be used for daily operations
- Helpdesk access should be delegated, not privileged
- Member Server administrators must not have access to Domain Controllers

---

## ⚠ Challenges & Troubleshooting

Several issues were encountered and resolved during implementation:

- **Group membership not immediately effective** — After adding `admin.t2` to `Tier2-Helpdesk-Admins`, login attempts still failed until the Kerberos token was refreshed. Required full logoff/logon cycle or `klist purge` to force token update.
- **admin.t2 login blocked correctly on DC01** — Validated that the "sign-in method you're trying to use isn't allowed" message appeared when `admin.t2` attempted to log into DC01 directly. This confirmed logon restriction was effective even without an explicit GPO on Domain Controllers OU.
- **admin.t2 Access Denied on AD group modification** — When `admin.t2` attempted to modify `Tier2-Helpdesk-Admins` group membership via ADUC, received: _"You do not have permission to modify the group corp.local/Groups/Tier2-Helpdesk-Admins"_. Delegation scope was correct — helpdesk is limited to password reset only.
- **Nested group privilege not immediately visible** — After nesting `admin.t0` → `Tier0-Domain-Admins` → `Domain Admins`, privilege propagation required replication. Validated with `whoami /groups` after relogon.
- **FS01 local admin verification** — Used `lusrmgr.msc` on FS01 to confirm `Tier1-Server-Admins` was correctly added to local Administrators group. Validated with `whoami /groups` showing `CORP\Tier1-Server-Admins` in session token.

Tools used for troubleshooting:

- `whoami /groups` — verify active group membership in session token
- `klist purge` — force Kerberos ticket refresh
- `gpresult /r` — confirm GPO application
- `lusrmgr.msc` — verify local group membership on FS01

---

## 🛠 Skills Demonstrated

- Organizational Unit (OU) design and computer object relocation
- Tiered Administrative Model implementation (Tier 0 / 1 / 2)
- Dedicated administrative account creation and separation
- Nested security group strategy (user → role group → privileged group)
- Delegated Administration configuration via ADUC Delegation Wizard
- Principle of Least Privilege enforcement through group-based access
- Kerberos token behavior and group membership propagation
- Local Administrators group management on member servers
- Access validation using `whoami /groups` and live login testing

---

## 🏢 Enterprise Relevance

This model mirrors the Microsoft ESAE (Enhanced Security Admin Environment) / Red Forest principle in a simplified lab context.

This design prevents:

- Privilege escalation via overprivileged accounts
- Credential theft lateral movement between tiers
- Overprivileged account misuse in daily operations
