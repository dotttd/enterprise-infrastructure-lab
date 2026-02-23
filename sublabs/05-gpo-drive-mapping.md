# Sub-Lab 05 – Group Policy Drive Mapping Automation

## 📌 Objective

Automate department-based network drive assignment using Group Policy Preferences to align with the implemented RBAC model.

This sub-lab eliminates manual drive mapping and enforces centralized policy-driven access control.

---

## 🏗 Environment Context

Domain: corp.local  
Domain Controller: DC01 (192.168.200.10)  
File Server: FS01 (192.168.200.20)  
Client: WIN10

Distributed multi-host architecture over dedicated LAN (192.168.200.0/24)

---

## 🎯 Automation Goals

- Automatically map HR drive for HR users
- Automatically map Finance drive for Finance users
- Automatically map all departmental drives for IT-Admin
- Prevent unauthorized drive visibility
- Maintain alignment with RBAC permissions

---

## 🛠 GPO Configuration

Created new Group Policy Object:

Name: Drive Mapping – Department Access

Location:

User Configuration  
→ Preferences  
→ Windows Settings  
→ Drive Maps

---

## 🔐 Drive Mapping Configuration

### HR Mapping

Action: Replace  
Location: \\192.168.200.20\HR$  
Drive Letter: H:

Item-Level Targeting:
Security Group → HR-Staff

---

### Finance Mapping

Action: Replace  
Location: \\192.168.200.20\Finance$  
Drive Letter: F:

Item-Level Targeting:
Security Group → Finance-Staff

---

### IT Mapping

IT-Admin granted access to:

- HR
- Finance
- IT

Separate drive mappings created targeting:

Security Group → IT-Admin

This avoids logical conflict in targeting conditions.

---

## 🔍 Targeting Logic Issue & Resolution

Issue encountered:

Adding multiple security groups inside one targeting entry resulted in AND logic, meaning the user had to belong to both groups.

Example:
HR-Staff AND IT-Admin

Resolution:

Created separate drive mapping entries for IT-Admin instead of combining groups.

This ensured OR-like behavior while maintaining clean policy structure.

---

## 🧪 Validation & Testing

Performed validation using:

```cmd
gpupdate /force
gpresult /r
whoami /groups
```

Tested scenarios:

- HR user login → Only H: visible
- Finance user login → Only F: visible
- IT-Admin login → H:, F:, I: visible

Confirmed:

- HR user login → Only `H: HR DRIVE` visible (confirmed in This PC view)
- Finance user login → Only `F:` visible; `gpresult /r` shows `Drive Mapping – Departement` GPO applied from `DC01.corp.local`
- IT-Admin login → `H:`, `F:`, `I:` all visible
- Cross-host GPO propagation working correctly (GPO from DC01 on PC2, applied on WIN10 on PC1)

---

## ⚠ Troubleshooting Performed

- Token refresh delay after group membership changes
- Required full client restart for group membership update
- Kerberos ticket purge using:
  klist purge
- Removed cached SMB sessions:
  net use \* /delete
- Verified GPO linkage at correct OU level
- Ensured security filtering was not blocking policy

---

## 🧠 Design Principles Applied

- Policy-driven automation
- Security group-based targeting
- Avoidance of manual user configuration
- Separation of permission logic and presentation layer
- Scalable drive provisioning model

---

## 🏢 Enterprise Relevance

Manual drive mapping is one of the most common helpdesk complaints in organizations without GPO automation. Every time a user logs into a new machine or has a profile issue, IT has to re-map the drives manually. GPO Preferences with Item-Level Targeting completely eliminates this — which is why it is standard practice in every well-managed Windows domain.

The `gpresult /r` output confirmed the GPO `Drive Mapping – Departement` was applied correctly from `DC01.corp.local` to `finance.staff` as a User Configuration policy (`CN=finance,OU=FINANCE,DC=corp,DC=local`). This demonstrates understanding of how User Configuration GPOs are scoped and applied, which is essential knowledge for any Sysadmin or Infrastructure Engineer role.

The Item-Level Targeting AND vs OR logic issue encountered and resolved also reflects a real troubleshooting scenario that appears in enterprise environments when GPOs have complex targeting conditions.

---

## 🎯 Outcome

Successfully automated drive provisioning aligned with RBAC model.

The infrastructure now supports:

- Centralized identity management
- Automated resource assignment (H: for HR, F: for Finance, H:/F:/I: for IT-Admin)
- Departmental segmentation without manual IT intervention
- Distributed policy enforcement: GPO from DC01 (PC2) applied on WIN10 (PC1)

This sub-lab demonstrates enterprise-style Group Policy implementation and troubleshooting.

→ Continued in [Sub-Lab 06 – Tiered Admin Model](./06-tiered-admin-model.md)
