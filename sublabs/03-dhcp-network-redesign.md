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

DHCP Server: DC01 (192.168.200.10)

Scope Configuration:

| Parameter   | Value                             |
| ----------- | --------------------------------- |
| Scope Name  | `LAN-LABS`                        |
| Range       | 192.168.200.100 – 192.168.200.200 |
| Subnet Mask | 255.255.255.0                     |
| DNS Server  | 192.168.200.10                    |
| Gateway     | Not configured (isolated LAN)     |

Scope status: **Active** — confirmed via DHCP Manager on DC01.

---

## 🔍 Validation Testing

Performed the following validations:

- `ipconfig /renew` on WIN10 — confirmed DHCP lease from 192.168.200.100–200 range
- Verified IP assignment from DHCP scope `LAN-LABS`
- Verified DNS resolution via `nslookup corp.local` returning 192.168.200.10
- Verified domain authentication after IP migration
- Verified cross-host ping (DC01 → FS01, DC01 → WIN10)
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

- Reviewing VirtualBox network adapter mode (NAT → Bridged)
- Flushing DHCP leases on DC01
- Restarting DHCP service via `net stop dhcpserver` / `net start dhcpserver`
- Revalidating DNS settings on all VMs
- Secure channel verification after token reissue

---

## 🧠 Design Decisions

- Static IP assigned to infrastructure servers
- DHCP reserved for client endpoints
- Physical LAN chosen for stability and realism
- Internal-only network improves isolation
- Clear separation between infrastructure and client workloads

---

## 🏢 Enterprise Relevance

In enterprise environments, DHCP is never served from a home router or unmanaged switch — it is always centralized on a Windows Server (or Linux equivalent) and integrated with DNS for dynamic name registration. This sub-lab replicates that model exactly.

The IP scheme migration from a flat NAT `10.10.x.x` to a structured `192.168.200.0/24` network mirrors real-world infrastructure migrations that IT teams perform when redesigning legacy networks. The process of rebuilding DHCP scopes, revalidating DNS, and re-joining domain services is identical to what happens in production during IP scheme changes.

The ability to operate an isolated internal network without a home router gateway also demonstrates understanding of network segmentation — a key security principle.

---

## 🎯 Outcome

Successfully implemented centralized DHCP service (`LAN-LABS` scope, Active) within a redesigned distributed network architecture.

The lab now operates on a stable, physical LAN-based enterprise topology with:

- Centralized IP management (DHCP scope active on dc01.corp.local)
- Integrated DNS & AD
- Cross-host authentication
- Improved infrastructure reliability

This sub-lab demonstrates real-world network redesign and infrastructure migration practices.

→ Continued in [Sub-Lab 04 – File Server & RBAC](./04-file-server-rbac.md)
