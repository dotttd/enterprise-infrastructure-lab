# Sub-Lab 11 – Centralized Security Analytics (Wazuh SIEM)

## 📌 Objective

Implement a centralized **Security Information and Event Management (SIEM)** platform using Wazuh to visualize security telemetry, detect threats in real-time, and manage vulnerabilities across the enterprise infrastructure.

Key objectives:

- Deploy a dedicated **Wazuh Manager** on a Linux VM (scaled to PC3).
- Mass-deploy **Wazuh Agents** via Group Policy to all domain assets.
- Integrate **Sysmon** logs into the SIEM dashboard for deep forensic visibility.
- Monitor infrastructure-wide vulnerabilities (CVEs) and compliance (MITRE ATT&CK).

---

## 🏗 Environment Context

- **PC3 (New):** Dedicated host for Security Monitoring.
- **SIEM Manager:** Ubuntu 22.04 LTS (192.168.200.30).
- **Subnet:** 192.168.200.0/24 (via Network Switch).
- **End-to-End Pipeline:** `Sysmon -> WEF -> Wazuh Agent -> Wazuh Manager`.

---

## 🏛 3-PC Physical Topology

```text
[ PC1: FS/WIN10 ]    [ PC2: DC01 ]    [ PC3: WA_SIEM ]
       │                   │                │
       └─────┬─────────────┴────────────────┘
             ▼
      [ Physical Switch ]
       (192.168.200.x)
```

---

## 🛠 Implementation Steps (Summary)

### 1. Security Manager Setup (PC3)

- Install Ubuntu Server on PC3 with a Bridged Network Adapter.
- Deploy Wazuh using the one-liner installation script.

### 2. Agent Deployment (GPO)

- Create a the deployment package (MSI) on the `Deployment$` share.
- Use GPO to install the Wazuh Agent on **DC01**, **FS01**, and **WIN10**.

### 3. Log Integration

- Update the Wazuh Agent `ossec.conf` (via GPO or manual script) to ingest the **Sysmon** log channel.

---

## 🎯 Value for Portfolio

- **Security Operations:** Demonstrates ability to manage a full-scale SIEM pipeline.
- **Scaling Infrastructure:** Shows technical maturity in scaling a lab environment horizontally (3-PC setup).
- **Threat Detection:** Proving you can turn "Raw Logs" into "Actionable Intelligence" using industry-standard tools (Wazuh/Kibana).
