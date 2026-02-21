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
- Automatically map all departmental drives for IT-Admins
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

IT-Admins granted access to:

- HR
- Finance
- IT

Separate drive mappings created targeting:

Security Group → IT-Admins  

This avoids logical conflict in targeting conditions.

---

## 🔍 Targeting Logic Issue & Resolution

Issue encountered:

Adding multiple security groups inside one targeting entry resulted in AND logic, meaning the user had to belong to both groups.

Example:
HR-Staff AND IT-Admins

Resolution:

Created separate drive mapping entries for IT-Admins instead of combining groups.

This ensured OR-like behavior while maintaining clean policy structure.

---

## 🧪 Validation & Testing

Performed validation using:

gpupdate /force  
gpresult /r  
whoami /groups  

Tested scenarios:

- HR user login → Only H: visible
- Finance user login → Only F: visible
- IT-Admin login → H:, F:, I: visible

Confirmed:

- Proper policy application
- No unauthorized drive exposure
- Cross-host GPO propagation working correctly

---

## ⚠ Troubleshooting Performed

- Token refresh delay after group membership changes
- Required full client restart for group membership update
- Kerberos ticket purge using:
  klist purge
- Removed cached SMB sessions:
  net use * /delete
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

## 🎯 Outcome

Successfully automated drive provisioning aligned with RBAC model.

The infrastructure now supports:

- Centralized identity management
- Automated resource assignment
- Departmental segmentation
- Distributed policy enforcement across physical hosts

This sub-lab demonstrates enterprise-style Group Policy implementation and troubleshooting.