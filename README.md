# Enterprise Infrastructure Lab (Distributed Architecture)

## 📌 Overview

This project documents the design and implementation of a distributed Windows Server enterprise lab environment built across two physical machines connected via dedicated LAN.

The lab simulates a small business infrastructure including:

- Active Directory Domain Services
- DNS & DHCP
- Role-Based Access Control (RBAC)
- File Server with NTFS & SMB segmentation
- Automated Drive Mapping via Group Policy
- Multi-host distributed topology

---

## 🖥 Physical Architecture

### Topology

PC2 (Infrastructure Host)
  └── DC01 (Domain Controller)

Dedicated LAN Connection (Ethernet Cable)

PC1 (Application & Client Host)
  ├── FS01 (File Server)
  └── WIN10 (Domain Client)

✔ No dependency on home router  
✔ Dedicated internal enterprise network  
✔ Cross-host infrastructure simulation  

---

## 🌐 Network Design

Network: 192.168.200.0/24

DC01  → 192.168.200.10 (Static)  
FS01  → 192.168.200.20 (Static)  
Win10 → 192.168.200.100 (DHCP)  

Services:

- DHCP served by DC01
- DNS integrated with Active Directory
- Kerberos authentication enabled
- Internal LAN-based infrastructure

---

## 🏢 Infrastructure Components

### 🔹 DC01 – Domain Controller

- New forest deployment: corp.local
- DNS integrated with AD
- DHCP scope configured (192.168.200.100–200)
- Group Policy Management
- Security group design
- Kerberos authentication validation

---

### 🔹 FS01 – File Server

Department folder structure:

D:\Shares\HR  
D:\Shares\Finance  
D:\Shares\IT  

Implemented:

- NTFS permissions via Security Groups
- SMB hidden shares (HR$, Finance$, IT$)
- Role-based access segmentation

---

### 🔹 WIN10 – Domain Client

- Domain joined
- DHCP validated
- DNS resolution tested
- GPO application tested

---

## 🔐 RBAC Implementation

Security Groups:

- HR-Staff
- Finance-Staff
- IT-Admins

Access Matrix:

| Role     | HR | Finance | IT |
|----------|----|----------|----|
| HR       | ✔  | ❌       | ❌ |
| Finance  | ❌ | ✔        | ❌ |
| IT       | ✔  | ✔        | ✔ |

---

## 📂 Group Policy Automation

Drive Mapping implemented via:

User Configuration  
→ Preferences  
→ Windows Settings  
→ Drive Maps  

Features:

- Action: Replace
- Item-level targeting
- Security Group-based drive assignment
- Automated drive provisioning per department

---

## 🛠 Engineering Challenges Solved

- Resolved VM memory contention by migrating DC to dedicated host
- Redesigned DHCP scope after network topology change
- Migrated from internal NAT network to physical LAN
- Debugged Kerberos token propagation
- Corrected Item-level targeting (AND vs OR logic)
- Diagnosed firewall ICMP blocking
- Rebuilt distributed infrastructure across hosts

---

## 🎯 Skills Demonstrated

- Active Directory Deployment
- DNS & DHCP Configuration
- Network Topology Redesign
- Role-Based Access Control
- NTFS & SMB Permission Design
- Group Policy Automation
- Distributed Infrastructure Engineering
- Troubleshooting & Root Cause Analysis

---

## 🚀 Current Status

Sub-Lab Completed:

- Virtualization Setup
- Active Directory Deployment
- DHCP & Network Redesign
- File Server & RBAC
- GPO Drive Mapping

Next Steps:

- OU & Delegation Design
- Security Hardening
- Secondary Domain Controller
- Backup & Recovery Simulation