# AD Structure

## Tujuan

Active Directory in this lab is designed to simulate a small business infrastructure with centralized identity management and department-based access control.

The objectives of the AD design are:

- Provide centralized authentication via Domain Controller
- Separate users logically by department
- Implement group-based access control (RBAC)
- Support automated drive mapping via Group Policy
- Maintain scalability for future infrastructure expansion

Domain Name: corp.local  
Domain Controller: DC01 (192.168.200.10)

---

## OU Design

The lab uses a simplified but scalable Organizational Unit (OU) structure.

Current structure:

corp.local  
├── OU=Users  
│   ├── OU=HR  
│   ├── OU=Finance  
│   └── OU=IT  
│  
├── OU=Groups  
│  
└── OU=Servers  

### Description

- OU=Users  
  Contains all domain user accounts organized by department.

- OU=Groups  
  Contains security groups used for RBAC.

- OU=Servers  
  Contains member servers such as FS01.

This structure allows:

- Future GPO assignment per department
- Easier delegation of administrative control
- Logical separation between users, groups, and servers

---

## Naming Convention

To maintain clarity and scalability, the following naming standards are applied:

### User Accounts

Format:
department.username

Examples:
- hr.staff
- finance.staff
- it.admin

---

### Security Groups

Format:
Department-Role

Examples:
- HR-Staff
- Finance-Staff
- IT-Admins

Group Type: Security  
Group Scope: Global  

Security groups are used for all permission assignments instead of individual users.

---

### Servers

Format:
Role + Numeric Identifier

Examples:
- DC01 (Domain Controller)
- FS01 (File Server)

This naming model supports scaling (e.g., DC02, FS02).

---

## Catatan Implementasi

- Active Directory Domain Services deployed on DC01
- DNS integrated with Active Directory
- DHCP hosted on DC01 with custom scope 192.168.200.0/24
- All user access controlled via Security Groups
- NTFS permissions aligned with AD group membership
- Drive mapping automated using GPO with Item-level targeting
- Token validation tested using `whoami /groups`
- Policy validation tested using `gpresult /r`
- Domain authentication validated across two physical hosts

Infrastructure evolved from single-host virtualization to distributed multi-host architecture to eliminate memory contention and better simulate enterprise design.

---

## Design Principles Applied

- Group-Based Access Control
- Departmental Segmentation
- Role Separation (Domain Controller vs File Server)
- Static IP allocation for infrastructure servers
- Scalable OU model
- Centralized identity management