# Sub-Lab 03 – DHCP Deployment & Network Redesign

## 📌 Objective

Design and implement centralized DHCP service while redesigning the lab network topology to support distributed multi-host infrastructure.

This sub-lab represents the transition from a single-host virtual network to a physical LAN-based enterprise simulation.

---

## 🏗 Initial Network Design

Initial implementation used:

- VirtualBox NAT / Internal Network
- IP scheme: 10.10.x.x
- All VMs hosted on a single physical machine

Limitations identified:

- Resource contention
- Unstable VM performance
- Limited realism compared to enterprise environment

---

## 🔄 Infrastructure Redesign

To improve stability and realism, the architecture was redesigned:

### Final Physical Topology

PC2 (Infrastructure Host)
  └── DC01 (Domain Controller, DNS, DHCP)

LAN Direct Ethernet Connection

PC1 (Application Host)
  ├── FS01 (File Server)
  └── WIN10 (Client)

Network redesigned to:

192.168.200.0/24

This removed dependency on VirtualBox NAT and home router.

---

## 🌐 IP Scheme Migration

Old Network:
10.10.x.x (NAT-based)

New Network:
192.168.200.0/24 (Physical LAN)

Server Allocation:

DC01 → 192.168.200.10 (Static)  
FS01 → 192.168.200.20 (Static)  
WIN10 → DHCP Assigned  

This required:

- Reconfiguring static IPs
- Rebuilding DHCP scope
- Validating DNS settings
- Rejoining network services

---

## 📡 DHCP Deployment

DHCP role hosted on:

DC01 (192.168.200.10)

Scope Configuration:

Range:
192.168.200.100 – 192.168.200.200  

Subnet Mask:
255.255.255.0  

DNS Server:
192.168.200.10  

Gateway:
Not configured (isolated LAN)

Scope activated and validated.

---

## 🔍 Validation Testing

Performed the following validations:

- ipconfig /renew on WIN10
- Verified IP assignment from DHCP
- Verified DNS resolution
- Verified domain authentication after IP migration
- Verified cross-host ping
- Confirmed FS01 reachable from WIN10

Confirmed full functionality of:

- DHCP
- DNS
- AD authentication
- SMB connectivity

---

## ⚠ Challenges Encountered

- DHCP scope mismatch after IP redesign
- Client receiving incorrect network range
- Firewall blocking ICMP between hosts
- Bridged adapter misconfiguration
- Token revalidation required after network migration

Troubleshooting involved:

- Reviewing network adapter mode
- Flushing DHCP leases
- Restarting DHCP service
- Validating DNS settings
- Secure channel verification

---

## 🧠 Design Decisions

- Static IP assigned to infrastructure servers
- DHCP reserved for client endpoints
- Physical LAN chosen for stability and realism
- Internal-only network improves isolation
- Clear separation between infrastructure and client workloads

---

## 🎯 Outcome

Successfully implemented centralized DHCP service within a redesigned distributed network architecture.

The lab now operates on a stable, physical LAN-based enterprise topology with:

- Centralized IP management
- Integrated DNS & AD
- Cross-host authentication
- Improved infrastructure reliability

This sub-lab demonstrates real-world network redesign and infrastructure migration practices.