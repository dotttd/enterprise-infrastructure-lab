# Sub-Lab 04 – File Server Deployment & RBAC Implementation

## 📌 Objective

Deploy a dedicated member file server and implement Role-Based Access Control (RBAC) using Active Directory Security Groups.

This sub-lab establishes department-based file segmentation aligned with enterprise best practices.

---

## 🏗 Environment Context

File Server: FS01  
IP Address: 192.168.200.20  
Domain: corp.local  

Architecture:

- DC01 hosted on PC2
- FS01 hosted on PC1
- WIN10 client hosted on PC1
- All systems connected via dedicated LAN (192.168.200.0/24)

---

## 🛠 File Server Deployment

### 1️⃣ Join FS01 to Domain

- Configured static IP
- DNS set to 192.168.200.10
- Joined FS01 to corp.local
- Verified domain trust
- Restarted server

---

### 2️⃣ Install File Server Role

Installed:

File and Storage Services  
→ File Server  

Confirmed SMB service operational.

---

## 📂 Folder Structure Design

Created structured directory:

D:\Shares\
  ├── HR
  ├── Finance
  └── IT

Each folder represents a department.

Folders shared as hidden shares:

- HR$
- Finance$
- IT$

Hidden shares prevent casual browsing and align with enterprise standards.

---

## 🔐 RBAC Implementation

### Security Groups Used

- HR-Staff
- Finance-Staff
- IT-Admins

Permissions assigned via Security Groups only.

No direct user-to-folder permissions applied.

---

## 📜 NTFS Permission Configuration

### HR Folder

- HR-Staff → Modify
- IT-Admins → Full Control

### Finance Folder

- Finance-Staff → Modify
- IT-Admins → Full Control

### IT Folder

- IT-Admins → Full Control

No cross-department permissions between HR and Finance.

---

## 🌐 Share-Level Permissions

Share permissions aligned with NTFS permissions to prevent conflicts.

Example:

HR$ share:
- HR-Staff → Change
- IT-Admins → Full Control

Finance$ share:
- Finance-Staff → Change
- IT-Admins → Full Control

IT$ share:
- IT-Admins → Full Control

---

## 🔍 Validation Testing

Access tested from WIN10 domain client:

- HR user → Access HR only
- Finance user → Access Finance only
- IT user → Access HR, Finance, IT

Validation performed using:

- \\FS01\HR$
- \\192.168.200.20\Finance$
- Manual access testing
- Access Denied verification
- whoami /groups to confirm token membership

Confirmed proper RBAC enforcement across distributed hosts.

---

## ⚠ Challenges Encountered

- Incorrect group membership propagation
- Token refresh required after group updates
- Incorrect NTFS inheritance during initial setup
- Share permission alignment issues
- Cross-host connectivity troubleshooting

Resolved using:

- gpupdate /force
- klist purge
- net use * /delete
- Permission review via Advanced Security Settings

---

## 🧠 Design Principles Applied

- Principle of Least Privilege
- Group-Based Access Control
- Separation of Identity and Permission
- Hidden share implementation
- Cross-host validation in distributed architecture
- Static IP allocation for infrastructure reliability

---

## 🎯 Outcome

Successfully deployed a domain-integrated file server with:

- Department-based segmentation
- Security group-based access control
- Hidden administrative shares
- Cross-host validation
- Scalable permission model

This sub-lab demonstrates enterprise-style RBAC implementation within a distributed Windows infrastructure.