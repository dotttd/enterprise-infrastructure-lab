# Sub-Lab 09 – Centralized Event Log Monitoring (Windows Event Forwarding)

## 📌 Objective

Implement Windows Event Forwarding (WEF) to centralize security event collection from FS01 into DC01's Forwarded Events log, enabling centralized monitoring without direct server access.

Key objectives:

- DC01 as **Collector** — pulls Security events from FS01
- FS01 as **Event Source** — exposes Security log via WinRM
- Validate Event IDs 4624, 4625, 4672 appear in DC01 Forwarded Events
- Scalable design applicable to additional Tier 1 servers

---

## 🏗 Environment Context

Domain: corp.local
Domain Controller / Collector: DC01 (192.168.200.10)
File Server / Source: FS01 (192.168.200.20)
Workstation: WIN10 (192.168.200.100)

---

## 🏛 WEF Architecture

```text
DC01 (Collector)                     FS01 (Event Source)
+-------------------------+          +----------------------+
| wecsvc (Event Collector)|  <pull-- | WinRM Service :5985  |
| Forwarded Events Log    |  admin.t1| Security Event Log   |
| Subscription:           |          | Event 4624/4625/4672 |
|   FS01-Security-Events  |          +----------------------+
+-------------------------+
```

Model used: **Collector-Initiated (Pull)**

- DC01 periodically polls FS01 via WinRM using `CORP\admin.t1` credentials
- FS01 does not need to reach DC01 (no push, no WEC endpoint required on DC01)
- Subscription mode: Custom Pull, 30-second delivery interval

---

## 🛠 Implementation

### DC01 — Collector Setup

Windows Event Collector service activated and configured:

```cmd
wecutil qc /q
```

![wecsvc Running on DC01](../docs/screenshots/09-centralized-event-log-monitoring/01-wecsvc-running-dc01.png)

Subscription `FS01-Security-Events` created via Event Viewer GUI:

- Type: Collector Initiated
- Source: `FS01.corp.local`
- Events: Security log, EventID 4624, 4625, 4672
- Credentials: `CORP\admin.t1`
- Configuration: Custom Pull, 30-second latency, `ReadExistingEvents: true`

Subscription XML deployed via:

```cmd
wecutil cs C:\wef-ci.xml
```

![Subscription Runtime Status Active](../docs/screenshots/09-centralized-event-log-monitoring/07-subscription-runtime-status-active-gui.png)

---

### GPO — WEF-Source-Configuration (Tier1 OU)

GPO linked to `corp.local/Tier1`, applied to FS01:

| Setting                               | Value                                                                         |
| ------------------------------------- | ----------------------------------------------------------------------------- |
| Configure target Subscription Manager | `Server=http://DC01.corp.local:5985/wsman/SubscriptionManager/WEC,Refresh=60` |
| Event Log Readers (Restricted Groups) | `NETWORK SERVICE` added as member                                             |

![GPO WEF-Source-Configuration Linked to Tier1](../docs/screenshots/09-centralized-event-log-monitoring/02-gpo-wef-source-configuration-linked-tier1.png)

![GPO Subscription Manager URL Setting](../docs/screenshots/09-centralized-event-log-monitoring/03-gpo-subscription-manager-url.png)

---

### FS01 — Source Configuration

| Configuration                          | Details                                   |
| -------------------------------------- | ----------------------------------------- |
| WinRM                                  | Active, port 5985, HTTP listener          |
| Firewall — Windows Remote Management   | Enabled — Domain profile, Inbound         |
| Firewall — Remote Event Log Management | Enabled — Inbound (RPC, NP-In, RPC-EPMAP) |
| Local group: Event Log Readers         | `CORP\DC01$`, `NETWORK SERVICE`           |
| Local group: Remote Management Users   | `CORP\DC01$`                              |
| Local group: Administrators            | `CORP\Tier1-Server-Admins`                |

![WinRM Listener FS01 Port 5985](../docs/screenshots/09-centralized-event-log-monitoring/04-winrm-listener-fs01.png)

![Remote Management Users DC01$ Member](../docs/screenshots/09-centralized-event-log-monitoring/05-remote-management-users-dc01-fs01.png)

![Firewall WinRM and Remote Event Log Enabled](../docs/screenshots/09-centralized-event-log-monitoring/06-firewall-winrm-remote-eventlog-fs01.png)

---

## 🔍 Validation Results

### Subscription Status (DC01)

```
Subscription: FS01-Security-Events
RunTimeStatus: Active
LastError: 0
EventSources:
    FS01.corp.local
        RunTimeStatus: Active
        LastError: 0
```

![wecutil gr Active LastError 0](../docs/screenshots/09-centralized-event-log-monitoring/08-wecutil-gr-active-lasterror-0.png)

![wecutil gs Subscription Config](../docs/screenshots/09-centralized-event-log-monitoring/15-wecutil-gs-subscription-config-pull-admin-t1.png)

### Forwarded Events (DC01)

| Event ID | Description                 | Source Computer |
| -------- | --------------------------- | --------------- |
| 4624     | Successful logon            | FS01.corp.local |
| 4625     | Failed logon                | FS01.corp.local |
| 4672     | Special privileges assigned | FS01.corp.local |

Total events forwarded: **1,740 events** confirmed in DC01 Forwarded Events log.

![Forwarded Events Overview 1740 Events from FS01](../docs/screenshots/09-centralized-event-log-monitoring/09-forwarded-events-overview-1740-events-fs01.png)

![Event 4624 Logon Detail from FS01](../docs/screenshots/09-centralized-event-log-monitoring/10-event-4624-logon-detail-fs01.png)

![Event 4625 Failed Logon hr.staff from FS01](../docs/screenshots/09-centralized-event-log-monitoring/11-event-4625-failed-logon-hr-staff-fs01.png)

![Event 4672 Special Privilege admin.t1 from FS01](../docs/screenshots/09-centralized-event-log-monitoring/14-event-4672-special-privilege-admin-t1-fs01.png)

---

## ⚠ Challenges & Troubleshooting

This sub-lab required extensive troubleshooting before WEF functioned correctly. Issues are documented in order of discovery:

---

**1. `wef\storage` folder missing — Ghost subscriptions (Error 0x2)**

Every subscription became a ghost (registered in registry but missing backing file) because `C:\Windows\System32\wef\storage\` was never created by `wecutil qc /q`. Required manual folder creation before subscriptions could be persisted:

```cmd
mkdir C:\Windows\System32\wef\storage
```

---

**2. GPO Subscription Manager URL pointed to wrong host**

The GPO `WEF-Source-Configuration` was configured with `192.168.200.20` (FS01's own IP) instead of `DC01.corp.local`. FS01 was forwarding events to itself. Fixed by correcting the GPO value to `http://DC01.corp.local:5985/wsman/SubscriptionManager/WEC,Refresh=60`.

---

**3. WEC WinRM endpoint (Source-Initiated) returned HTTP 404**

When using Source-Initiated model, FS01 received HTTP 404 on `http://DC01.corp.local:5985/wsman/SubscriptionManager/WEC`. Repeated `wecutil qc /q` and WinRM restarts could not register the WEC plugin endpoint. `winrm enumerate winrm/config/plugin | findstr Event` returned empty — plugin was never registered.

Resolution: **Switched to Collector-Initiated model** which bypasses the WEC endpoint requirement entirely. DC01 connects to FS01's WinRM, not the reverse.

---

**4. AllowedSourceDomainComputers SDDL used wrong SID alias**

Source-Initiated subscription was configured with SDDL `(A;;GA;;;DC)` — `DC` in SDDL refers to **Domain Controllers** group (S-1-5-21-...-516), not Domain Computers. FS01 is a member server, not a DC, so it was never authorized to push events. Fixed by replacing with Domain Computers SID:

```powershell
$sid = (Get-ADGroup "Domain Computers").SID.Value
wecutil ss FS01-Security-Events /adc:"O:NSG:NSD:(A;;GA;;;$sid)(A;;GA;;;NS)"
```

---

**5. DC01 machine account (DC01$) lacked WinRM access on FS01**

When switching to Collector-Initiated, wecsvc on DC01 runs as `NETWORK SERVICE` (DC01$ machine account). DC01$ was not in FS01's `Remote Management Users` group, causing "WinRM cannot complete the operation" (error 0x803381E6) even though user-credential `Invoke-Command` to FS01 succeeded.

Fixed by adding DC01$ to both groups on FS01:

```cmd
net localgroup "Event Log Readers" "CORP\DC01$" /add
net localgroup "Remote Management Users" "CORP\DC01$" /add
```

---

**6. MinLatency mode silently switched DeliveryMode to Push**

Changing the subscription's `ConfigurationMode` to `MinLatency` automatically changed `DeliveryMode: Pull → Push`. In Push mode, FS01 must connect back to DC01's WEC endpoint — the same 404 issue as problem #3. Error: `0x80338095: The connectivity test from push subscription source to client failed`.

Resolution: Used `ConfigurationMode: Custom` with explicit `Mode="Pull"` and `MaxLatencyTime: 30000` in the subscription XML, keeping DC01 as the initiator.

---

**7. Tier1-Server-Admins not added to FS01 Local Administrators (Sub-Lab 06 gap)**

During sub-lab 09 testing, `admin.t1` could not elevate on FS01 (prompted for higher credentials). `net localgroup Administrators` showed only `Administrator` and `CORP\Domain Admins` — `Tier1-Server-Admins` was missing despite being documented in sub-lab 06.

Fixed by manually adding the group:

```powershell
Add-LocalGroupMember -Group "Administrators" -Member "CORP\Tier1-Server-Admins"
```

Enterprise-grade fix via GPO (Preferences → Local Users and Groups) was noted as the correct long-term approach.

---

**8. Final solution: Explicit credentials + Custom Pull mode**

Root cause of persistent failures was that wecsvc machine account (DC01$) lacked sufficient rights on FS01, and no Pull mode with short latency was achievable via `wecutil ss`. Final working subscription:

```xml
<SubscriptionType>CollectorInitiated</SubscriptionType>
<ConfigurationMode>Custom</ConfigurationMode>
<Delivery Mode="Pull">
    <Batching>
        <MaxItems>1</MaxItems>
        <MaxLatencyTime>30000</MaxLatencyTime>
    </Batching>
</Delivery>
<ReadExistingEvents>true</ReadExistingEvents>
<CommonUserName>CORP\admin.t1</CommonUserName>
```

---

## ✅ Security Model Achieved

- Centralized log collection without per-server access
- Events collected for authentication monitoring (4624, 4625, 4672)
- Collector authenticated with least-privilege Tier 1 account
- Source firewall explicitly allows only required ports (5985, RPC)

---

## 🧠 Key Learning Outcomes

- WEF storage folder must exist before any subscription is created
- `MinLatency` changes delivery mode to Push — avoid with Collector-Initiated
- Machine account (COMPUTER$) permissions on remote hosts differ from user credentials
- Collector-Initiated is more reliable when Source-Initiated WEC endpoint cannot be registered
- `wecutil qc /q` may report success without fully registering the WEC WinRM plugin
- SDDL alias `DC` = Domain Controllers, NOT Domain Computers

---

## 🛠 Skills Demonstrated

- Windows Event Forwarding (WEF) end-to-end configuration
- WinRM and WS-Management protocol troubleshooting
- Kerberos machine account permission management
- WinRM plugin diagnosis via `winrm enumerate` and `Invoke-WebRequest`
- Subscription management with `wecutil` (gs, ss, gr, cs, ds)
- Firewall rule management for WinRM and Remote Event Log Management
- SDDL interpretation and AllowedSourceDomainComputers configuration
- XML-based subscription import for fine-grained WEF control

---

## 🏢 Enterprise Relevance

WEF is a foundational component of any SIEM pipeline in Microsoft environments. This lab mirrors:

- Microsoft recommended WEF architecture for centralized security monitoring
- NSA/CISA guidance on Windows event log collection
- Log aggregation as prerequisite for threat detection and incident response
- Collector-Initiated model used by SIEM agents (Splunk UF, ArcSight, Sentinel)
