# IP Design

## Purpose

This document describes the IP addressing and network design used in the distributed enterprise lab environment.

The lab was redesigned from a single-host NAT-based topology into a multi-host physical LAN architecture to improve performance and better simulate a real enterprise network.

The objectives of the IP design are:

- Provide stable static addressing for infrastructure servers
- Centralize IP management using an internal DHCP server
- Maintain isolation from the home network
- Ensure proper DNS and domain authentication functionality

---

## Network Overview

| Parameter       | Value                         |
| --------------- | ----------------------------- |
| Network Address | 192.168.200.0/24              |
| Subnet Mask     | 255.255.255.0                 |
| Gateway         | Not configured (isolated LAN) |
| DNS             | 192.168.200.10 (DC01)         |

This network is dedicated to the lab environment, connected via direct Ethernet cable between two physical machines. No external gateway is configured as the lab operates as an isolated internal network.

---

## Physical Topology

```text
PC2 (Infrastructure Host)
  └── DC01 — 192.168.200.10 (Domain Controller, DNS, DHCP)
        │
        │ [Direct Ethernet Cable — LAN 192.168.200.0/24]
        │
PC1 (Application Host)
  ├── FS01 — 192.168.200.20 (File Server)
  └── WIN10 — DHCP (192.168.200.100–200) (Domain Workstation)
```

The lab does not rely on a home router for internal communication.

---

## Server IP Allocation

| Host  | IP Address     | Role                              | Config |
| ----- | -------------- | --------------------------------- | ------ |
| DC01  | 192.168.200.10 | Domain Controller, DNS, DHCP      | Static |
| FS01  | 192.168.200.20 | File Server (Tier1 Member Server) | Static |
| WIN10 | DHCP assigned  | Domain Workstation (Tier2)        | DHCP   |

---

## DHCP Configuration

| Parameter    | Value                             |
| ------------ | --------------------------------- |
| DHCP Server  | DC01 (192.168.200.10)             |
| Scope Name   | `LAN-LABS`                        |
| Scope Range  | 192.168.200.100 – 192.168.200.200 |
| Subnet Mask  | 255.255.255.0                     |
| DNS Server   | 192.168.200.10                    |
| Gateway      | Not configured                    |
| Scope Status | **Active**                        |

---

## DNS Design

DNS is integrated with Active Directory and hosted on DC01.

| Parameter           | Value                       |
| ------------------- | --------------------------- |
| Primary DNS         | 192.168.200.10              |
| Forward Lookup Zone | corp.local                  |
| Integration         | Active Directory Integrated |

DNS is responsible for:

- Resolving `corp.local` domain
- Resolving server hostnames (`DC01`, `FS01`)
- Supporting Kerberos authentication between hosts

---

## Design Decisions

- Static IP is used for all infrastructure servers to prevent dependency on DHCP for critical services
- DHCP is reserved for client devices only (WIN10 and future workstations)
- Internal-only network eliminates external interference and home router dependency
- Dedicated LAN between PC1 and PC2 improves performance and cross-host stability
- Segmented infrastructure and client roles across two physical hosts mirrors enterprise topology

---

## Evolution of Design

### Initial Design (Deprecated)

- Single physical host (PC1)
- All VMs (DC01, FS01, WIN10) on the same machine
- VirtualBox internal NAT network (`10.10.x.x`)
- DC01 shared resources with FS01 and WIN10

**Problem:** Memory contention caused DC01 instability, authentication failures, and GPO propagation issues.

### Current Design

| Change       | Old             | New                        |
| ------------ | --------------- | -------------------------- |
| DC01 host    | PC1 (shared)    | PC2 (dedicated)            |
| Network type | VirtualBox NAT  | Physical LAN (Bridged NIC) |
| IP scheme    | 10.10.x.x       | 192.168.200.0/24           |
| DHCP source  | VirtualBox DHCP | Windows Server DHCP (DC01) |

This redesign reflects real-world infrastructure troubleshooting and architectural decision-making under hardware constraints.
