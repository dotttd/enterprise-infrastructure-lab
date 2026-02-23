# Sub-Lab 07 – Security Hardening & Tier Enforcement

## 📌 Objective

Implement logon restrictions based on the Tier Model to ensure administrative access boundaries are actively enforced — not just structurally designed.

Key objectives:

- Prevent Tier2 and business users from logging into servers
- Restrict Domain Controller access to Tier0 accounts only
- Ensure Tier1 accounts can only manage member servers
- Enforce access boundaries using Group Policy
- Validate all restrictions through real login testing

---

## 🏗 Environment Context

Domain: corp.local  
Domain Controller: DC01 (192.168.200.10)  
File Server: FS01 (192.168.200.20)  
Workstation: WIN10 (192.168.200.100)

---

## 🧭 Architecture Concept

The model applied is the Tiered Administrative Model:

```text
Tier   | Role                  | Access Scope
Tier0  | Domain Administration | Domain Controller only
Tier1  | Server Administration | Member Servers only
Tier2  | Helpdesk              | Workstations only
Users  | Business Users        | Workstations only
```

Core principle:

- Lower tier accounts must not be able to access higher tier systems

---

## 🛠 Technical Implementation

### 1) Create GPO for Logon Restriction

A new Group Policy Object was created and linked to the `Tier1` OU.

GPO Name:

- `Tier1-Logon-Restriction`

Because logon restriction is a Computer Configuration policy, the GPO must be linked to the OU containing the target computer objects (not the user OU).

---

### 2) Configure Allow Log On Locally

Configuration path:

```text
Computer Configuration
-> Policies
-> Windows Settings
-> Security Settings
-> Local Policies
-> User Rights Assignment
-> Allow log on locally
```

Configuration:

- Enable "Define these policy settings"
- Add only:
  - `Administrators`
  - `Tier1-Server-Admins`

Excluded (not added):

- `Domain Users`
- `Authenticated Users`
- `Users`

Result:

- Only Tier1 accounts and local Administrators can log into `FS01`

---

### 3) Clear Deny Log On Locally

To avoid policy override conflicts (Deny overrides Allow), an Allow-only model is used.

Configuration:

- `Deny log on locally` = Not Defined

---

### 4) Verify Policy Application

Run the following on `FS01`:

```cmd
gpupdate /force
gpresult /r /scope computer
```

Confirm that `Tier1-Logon-Restriction` appears under Applied Group Policy Objects in the Computer Configuration scope.

---

### 5) Validate Login Access

Direct login testing on `FS01` console using multiple accounts.

Validation results:

```text
Account   | Login FS01
admin.t0  | YES
admin.t1  | YES
admin.t2  | NO
HR user   | NO
Finance   | NO
```

---

### 6) Privilege Adjustment (Shutdown Rights)

Discovered that `admin.t1` could log in but could not restart the server.

Analysis:

- Login privilege (`Allow log on locally`) does not automatically grant system shutdown rights

Shutdown rights are configured at:

```text
User Rights Assignment -> Shut down the system
```

Resolution:

- Add `Tier1-Server-Admins` to `Shut down the system` User Rights Assignment
- Or ensure the group is a member of local `Administrators` on `FS01`

---

## ✅ Final Result

After implementation:

- Tier separation is actively enforced
- Business users cannot log into the server
- Tier2 (helpdesk) accounts cannot log into the server
- Tier1 accounts can only access member servers
- Privilege granularity is visible and testable
- GPO successfully overrides default Windows member server behavior

This sub-lab proves that the Tier Model is not merely an OU organization structure — it is a security policy that actively enforces access boundaries.

---

## 🧪 Technical Notes

- User Rights Assignment changes under Security Settings often require a server restart to fully apply
- `gpresult /r /scope computer` is mandatory for validating Computer Configuration GPOs
- Default Windows member server behavior allows Domain Users to log on locally — this must be explicitly overridden
- An Allow-only logon model is more stable than a Deny-based model for this scenario, since Deny always overrides Allow

---

## ⚠ Challenges & Troubleshooting

Several issues were encountered and resolved during implementation:

- **GPO not applied after creation** — After creating `Tier1-Logon-Restriction` GPO, policy was not immediately active on FS01. Required running `gpupdate /force` on FS01, then confirmed via `gpresult /r /scope computer` which showed the GPO listed under Applied Group Policy Objects.
- **HR and Finance users still able to attempt login** — Default Windows Server behavior allows Domain Users to log on locally. Until `Allow log on locally` was explicitly redefined with only `Administrators` and `Tier1-Server-Admins`, non-tier users were not restricted. Verified fix by relogging — login screen showed _"The sign-in method you're trying to use isn't allowed"_ for HR/Finance accounts.
- **admin.t1 could log in but could not restart server** — Discovered that login privilege (`Allow log on locally`) does not automatically include system shutdown rights. Resolved by adding `Tier1-Server-Admins` to `Shut down the system` User Rights Assignment in the same GPO.
- **User Rights Assignment delay** — Changes to User Rights Assignment under Security Settings required a restart on FS01 to fully apply, unlike Preference-based settings which apply on next `gpupdate`.
- **Validating GPO scope** — `gpresult /r /scope computer` was critical for confirming Computer Configuration policies. Using `/r` alone (without `/scope computer`) would only show User Configuration, missing the logon restriction.

Tools used for validation:

- `gpupdate /force` — force immediate policy refresh
- `gpresult /r /scope computer` — verify Computer Configuration GPO application
- `whoami /groups` — confirm session token group membership after login
- Direct login testing on FS01 console per account tier

---

## 🛠 Skills Demonstrated

- Group Policy Object (GPO) creation and OU-level linking
- User Rights Assignment configuration via Security Settings
- Computer Configuration policy scoping (vs User Configuration)
- Allow-only logon restriction model (avoiding Deny override conflicts)
- GPO result validation using `gpresult /r /scope computer`
- Tier-based access enforcement through policy rather than manual configuration
- Privilege granularity analysis (login rights vs shutdown rights)
- Real-world login testing per account tier as validation method

---

## 🧠 Architectural Insight

This sub-lab elevates the lab maturity from:

- "AD Deployment Lab"

To:

- "Security-Enforced Enterprise AD Simulation"

The concepts applied here are standard practice in:

- Enterprise Corporate Infrastructure
- Financial Institutions
- Industrial and Mining Networks
- Zero Trust-aligned Active Directory Design
