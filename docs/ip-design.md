# IP Design

## Tujuan

This document describes the IP addressing and network design used in the distributed enterprise lab environment.

The lab was redesigned from a single-host NAT-based topology into a multi-host physical LAN architecture to improve performance and better simulate a real enterprise network.

The objectives of the IP design are:

- Provide stable static addressing for infrastructure servers
- Centralize IP management using internal DHCP
- Maintain isolation from home network
- Ensure proper DNS and domain authentication functionality

---

## Network Overview

Network Address: 192.168.200.0/24  
Subnet Mask: 255.255.255.0  

This network is dedicated to the lab environment and connected via direct Ethernet cable between two physical machines.

No external gateway is configured as the lab operates in an isolated internal network.

---

## Physical Topology

PC2 (Infrastructure Host)
  └── DC01 (Domain Controller)

LAN Direct Connection (Ethernet Cable)

PC1 (Application Host)
  ├── FS01 (File Server)
  └── WIN10 (Domain Client)

The lab does not rely on a home router for internal communication.

---

## Server IP Allocation

DC01  
IP Address: 192.168.200.10  
Role: Domain Controller, DNS, DHCP  
Configuration: Static  

FS01  
IP Address: 192.168.200.20  
Role: File Server  
Configuration: Static  

WIN10  
IP Address: Assigned dynamically  
Configuration: DHCP  

---

## DHCP Configuration

DHCP Server: DC01  

Scope Range:  
192.168.200.100 – 192.168.200.200  

Subnet Mask:  
255.255.255.0  

DNS Server:  
192.168.200.10  

Gateway:  
Not configured (isolated LAN environment)

---

## DNS Design

DNS is integrated with Active Directory and hosted on DC01.

All clients use:

Preferred DNS: 192.168.200.10  

DNS is responsible for:

- Resolving corp.local domain
- Resolving server hostnames (DC01, FS01)
- Supporting Kerberos authentication

---

## Design Decisions

- Static IP used for all infrastructure servers to prevent dependency on DHCP for critical services
- DHCP reserved for client devices only
- Internal-only network eliminates external interference
- Dedicated LAN improves performance and stability
- Segmented infrastructure and client roles across physical hosts

---

## Evolution of Design

Initial lab design used:

10.10.x.x network with NAT-based VirtualBox internal networking.

After encountering performance and resource limitations, the lab was redesigned to:

- Use dedicated LAN between physical machines
- Migrate IP scheme to 192.168.200.x
- Rebuild DHCP scope
- Improve cross-host reliability

This redesign reflects real-world infrastructure troubleshooting and architectural decision-making.