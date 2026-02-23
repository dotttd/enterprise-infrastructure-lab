# Sub-Lab 08 – Server Hardening Baseline

## 📌 Objective

Apply a security hardening baseline to the Tier1 server (`FS01`) to simulate modern enterprise security practices.

Scope of hardening:

- Advanced audit logging
- RDP access restriction
- Firewall enforcement
- SMB protocol hardening
- Anonymous access restriction
- Security control validation through direct testing

This sub-lab transforms the environment from a "functional infrastructure" into a secured and controlled infrastructure.

---

## 🏗 Environment Context

Domain: corp.local  
Domain Controller: DC01 (192.168.200.10)  
File Server: FS01 (192.168.200.20)  
Workstation: WIN10 (192.168.200.100)

---

## 🧭 Architecture Overview

Tier Model applied:

```text
Tier0 -> Domain Controller (DC01)
Tier1 -> Member Server (FS01)
Tier2 -> Workstation (WIN10)
```

All policies are applied via OU-based GPO scoping.

Primary GPO created:

- `Tier1-Security-Baseline`

Linked to:

- `OU=Tier1`

---

## 🛠 Implementation Steps

### 1) Advanced Audit Policy Configuration

Configured granular audit logging using the modern Advanced Audit Policy.

Path:

```text
Computer Configuration
-> Policies
-> Windows Settings
-> Security Settings
-> Advanced Audit Policy Configuration
-> Audit Policies
```

Audit settings enabled:

Logon/Logoff:

- Audit Logon → Success + Failure
- Audit Special Logon → Success

Account Logon:

- Audit Credential Validation → Success + Failure

Additional enforcement:

- `Audit: Force audit policy subcategory settings` = Enabled

Purpose: ensures Advanced Audit Policy overrides legacy audit settings.

---

### 2) RDP Restriction (Tier Enforcement)

Restricted RDP access to `FS01` in accordance with the Tier Model.

Path:

```text
User Rights Assignment
-> Allow log on through Remote Desktop Services
```

Accounts permitted:

- `Administrators`
- `Tier1-Server-Admins`

`Deny log on through Remote Desktop Services` was left as Not Defined to avoid override conflicts.

Validation results:

```text
User      | RDP to FS01
admin.t0  | YES
admin.t1  | YES
admin.t2  | NO
HR/Finance| NO
```

Verified using `mstsc` and Event ID `4624`/`4625` in Event Viewer.

---

### 3) Firewall Enforcement

Enabled Windows Defender Firewall on the Domain Profile.

Validation:

```powershell
Get-NetFirewallProfile
```

Result:

- Domain Profile → Enabled: True

RDP port reachability tested from WIN10:

```powershell
Test-NetConnection 192.168.200.20 -Port 3389
```

---

### 4) SMB Hardening

SMBv1 was checked and disabled (if present):

```powershell
Get-WindowsFeature FS-SMB1
Uninstall-WindowsFeature FS-SMB1
```

Purpose:

- Eliminate the legacy SMBv1 protocol, which is vulnerable to WannaCry-class attacks

---

### 5) Restrict Anonymous Enumeration

Enabled the following policy:

- `Network access: Do not allow anonymous enumeration of SAM accounts`

Validation:

```cmd
net view \\192.168.200.20
```

Result:

- Anonymous access denied
- Valid credentials required to enumerate shares

---

## 🔍 Security Validation

Event Viewer confirmed the following Security log entries:

```text
Windows Logs -> Security
```

Events recorded:

| Event ID | Meaning                 |
| -------- | ----------------------- |
| 4624     | Successful logon        |
| 4625     | Failed logon attempt    |
| 4672     | Special privilege logon |

---

## 🧠 Security Principles Applied

- Least Privilege
- Tiered Administrative Model
- Attack Surface Reduction
- Credential Isolation
- Centralized Policy Enforcement
- Auditable Infrastructure

---

## ✅ Final Result

After completing this sub-lab:

- Server is accessible only by Tier1 administrators
- All login activity is recorded and auditable via Event Viewer
- Legacy SMBv1 protocol is disabled
- Anonymous share enumeration is blocked
- Windows Defender Firewall is active on the Domain Profile

The infrastructure is not only functional, but:

- Secure by design and validated through live testing

---

## ⚠ Challenges & Troubleshooting

Several issues were encountered and resolved during implementation:

- **RDP restriction required separate User Rights Assignment** — Setting `Allow log on through Remote Desktop Services` in GPO was independent from the `Allow log on locally` policy configured in Sub-Lab 07. Both had to be explicitly defined to ensure RDP access was also tier-restricted. Validated via `mstsc` from WIN10: `CORP\hr.staff` received _"The connection was denied because the user account is not authorized for remote login"_, while `admin.t1` connected successfully.
- **Firewall verification needed separate testing** — After enabling Windows Defender Firewall on Domain Profile, `Get-NetFirewallProfile` confirmed `Enabled: True`, but actual port connectivity was tested separately using `Test-NetConnection 192.168.200.20 -Port 3389` from WIN10 to verify RDP port was reachable for authorized accounts.
- **Audit Policy vs Legacy Audit Policy conflict** — Initially, legacy audit settings (under `Audit Policy`) overlapped with `Advanced Audit Policy Configuration`. Resolved by enabling _"Audit: Force audit policy subcategory settings"_ to ensure Advanced Audit overrides legacy settings. Confirmed by checking Event Viewer Security log for Event ID 4624, 4625, 4672.
- **SMBv1 check returned "not installed"** — Running `Get-WindowsFeature FS-SMB1` on FS01 confirmed SMBv1 feature was not installed (Windows Server 2019 ships without it by default). `Uninstall-WindowsFeature FS-SMB1` was run as a validation step regardless.
- **Anonymous enumeration test** — After enabling `Do not allow anonymous enumeration of SAM accounts`, ran `net view \\192.168.200.20` from WIN10 without credentials to confirm access was denied. Validated credential-required access behavior.

Tools used for validation:

- `Get-NetFirewallProfile` — verify firewall Domain Profile state
- `Test-NetConnection 192.168.200.20 -Port 3389` — validate RDP port reachability
- `mstsc` — test RDP login per account tier
- `Get-WindowsFeature FS-SMB1` — check SMBv1 installation status
- `net view \\192.168.200.20` — test anonymous enumeration
- Event Viewer → Security log — verify audit events 4624/4625/4672

---

## 🛠 Skills Demonstrated

- Advanced Audit Policy configuration (Logon/Logoff, Credential Validation)
- RDP access restriction via User Rights Assignment in GPO
- Windows Defender Firewall enforcement on Domain Profile
- SMBv1 protocol hardening (detect and disable legacy protocol)
- Anonymous SAM enumeration restriction
- Security control validation using Event Viewer (Event ID 4624, 4625, 4672)
- PowerShell-based network security testing (`Test-NetConnection`, `Get-NetFirewallProfile`)
- GPO-based security baseline deployment on member servers
- Attack surface reduction methodology

---

## 🏢 Enterprise Simulation Impact

This sub-lab positions the lab as:

- An enterprise-grade environment simulation
- A real baseline hardening implementation
- A tier-based access segmentation model
