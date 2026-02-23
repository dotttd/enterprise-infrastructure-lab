# Sub-Lab 01 – Virtualization & Infrastructure Foundation

## 📌 Objective

Establish the virtualization environment required to simulate an enterprise Windows infrastructure lab.

The goal was to deploy core infrastructure components using VirtualBox and prepare a stable foundation for Active Directory, DNS, DHCP, and file services.

---

## 🖥 Initial Environment Design (Single-Host Model)

The lab was initially designed using a single physical machine running multiple virtual machines.

Initial VM deployment:

- DC01 (Windows Server – Domain Controller)
- FS01 (Windows Server – File Server)
- WIN10 (Domain Client)

All VMs were running simultaneously on a host machine with 8GB RAM.

---

## ⚠ Challenges Encountered

During implementation, several performance issues occurred:

- Frequent VM hangs
- Domain Controller instability
- Resource contention
- High memory usage due to multiple Windows Server instances

Root Cause Identified:

- RAM overcommit (3 VMs consuming most host memory)
- AD services and DNS requiring consistent performance
- Windows Server GUI overhead

This required architectural redesign.

---

## 🔄 Infrastructure Redesign

To resolve performance bottlenecks, the lab architecture was redesigned into a distributed multi-host topology.

### Final Physical Design

PC2 (Infrastructure Host)
└── DC01 (Domain Controller)

LAN Direct Connection (Ethernet Cable)

PC1 (Application Host)
├── FS01 (File Server)
└── WIN10 (Domain Client)

This approach:

- Reduced memory contention
- Isolated Domain Controller workload
- Improved network reliability
- Better simulated real-world infrastructure

---

## 🌐 Virtualization Configuration

VirtualBox was used as the hypervisor.

### DC01 Configuration

- Installed on dedicated physical host (PC2)
- RAM allocated separately from PC1 workload to prevent contention
- Bridged network adapter (Intel PRO/1000 MT) to physical LAN interface
- Configured static IP: 192.168.200.10

### FS01 & WIN10 Configuration

- Hosted on PC1
- Bridged network adapter to dedicated LAN connection
- Integrated into distributed AD environment

---

## 🔌 Network Mode Evolution

The lab evolved through multiple network modes:

1. NAT-based internal network (10.10.x.x)
2. VirtualBox internal network
3. Final architecture: Bridged adapter to physical LAN (192.168.200.x)

The final implementation uses:

- Dedicated Ethernet cable
- Isolated internal 192.168.200.0/24 network
- Internal DHCP and DNS via DC01

---

## 🛠 Technical Skills Demonstrated

- VirtualBox deployment & configuration
- VM resource optimization
- Performance troubleshooting
- Network mode configuration (NAT vs Bridged)
- Cross-host infrastructure migration
- Memory bottleneck analysis
- Infrastructure redesign based on constraints

---

## 🏢 Enterprise Relevance

In production enterprise environments, Domain Controllers are always isolated on dedicated hardware or dedicated VMs with reserved resources — exactly what this redesign simulates. Running a DC on shared, resource-contended hardware is a known reliability risk that leads to Kerberos authentication failures and AD replication issues in real deployments.

The migration from a single-host NAT topology to a dedicated physical LAN also mirrors real-world infrastructure evolution: many small-to-medium companies begin with flat virtualized environments and progressively redesign as the environment grows.

This sub-lab demonstrates the engineering judgment to recognize architectural limitations and redesign proactively — a critical skill for IT Infrastructure and Systems Engineering roles.

---

## 🎯 Outcome

Successfully established a stable distributed virtualization foundation capable of supporting:

- Active Directory Domain Services
- DNS & DHCP
- File Server with RBAC
- Group Policy automation

Validated cross-host connectivity: DC01 (192.168.200.10) successfully pinged FS01 (192.168.200.20) and WIN10 (192.168.200.100) with 0% packet loss.

This redesign reflects real-world infrastructure engineering practices, including root cause analysis and architectural adjustment under hardware limitations.

→ Continued in [Sub-Lab 02 – Active Directory Deployment](./02-active-directory-deployment.md)
