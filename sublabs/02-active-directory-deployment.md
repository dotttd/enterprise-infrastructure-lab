# Sub-Lab 02 – Active Directory Deployment

## 📌 Objective

Deploy Active Directory Domain Services (AD DS) to establish centralized identity, authentication, and directory services within the enterprise lab environment.

This sub-lab transforms the infrastructure from standalone Windows machines into a domain-based enterprise architecture.

---

## 🏗 Environment Context

Domain Controller: DC01  
IP Address: 192.168.200.10  
Domain Name: corp.local  

Deployment performed in distributed architecture:

- DC01 hosted on dedicated physical machine (PC2)
- FS01 and WIN10 hosted on separate physical machine (PC1)
- Connected via dedicated LAN (192.168.200.0/24)

---

## 🛠 Deployment Steps Performed

### 1️⃣ Install Active Directory Domain Services Role

- Added AD DS role on DC01
- Included DNS Server integration
- Verified prerequisite checks

---

### 2️⃣ Promote DC01 to Domain Controller

- Created new forest: corp.local
- Set Directory Services Restore Mode (DSRM) password
- Installed DNS integration automatically
- Completed domain controller promotion
- Rebooted server to finalize deployment

---

### 3️⃣ DNS Integration

DNS was automatically integrated with Active Directory.

Configured:

- DC01 using itself as DNS (192.168.200.10)
- Verified forward lookup zone for corp.local
- Tested resolution using:

nslookup corp.local

DNS successfully resolves internal hostnames.

---

### 4️⃣ Domain Join Validation

WIN10 (Client) joined to domain:

- Verified DNS settings pointed to 192.168.200.10
- Joined to corp.local
- Restarted system
- Logged in using domain credentials

FS01 (Member Server) joined to domain:

- Configured static IP
- Verified DNS resolution
- Joined corp.local successfully

---

## 🔐 Authentication Validation

Validated:

- Domain login from WIN10
- Cross-host authentication (PC1 ↔ PC2)
- Kerberos ticket issuance
- Group membership validation via:

whoami /groups

Tested secure channel trust between client and DC.

---

## ⚠ Challenges & Troubleshooting

Several issues were encountered and resolved:

- Incorrect DNS configuration preventing domain join
- Token propagation delay after group membership changes
- Kerberos ticket caching issues
- Firewall blocking ICMP during cross-host testing
- Network redesign requiring revalidation of domain services

Tools used for troubleshooting:

- ipconfig /all
- nslookup
- gpresult /r
- klist purge
- net use * /delete

---

## 🧠 Design Decisions

- DNS integrated with AD for centralized management
- Static IP for Domain Controller
- No dependency on external gateway
- Single-DC model appropriate for small enterprise simulation
- Group-based identity design prepared for RBAC implementation

---

## 🎯 Outcome

Successfully deployed a fully functional Active Directory environment with:

- Centralized authentication
- DNS integration
- Cross-host domain validation
- Distributed infrastructure stability

This sub-lab established the identity backbone for all subsequent implementations including DHCP, File Server, RBAC, and Group Policy automation.