# Sub-Lab 09 – Centralized Event Log Monitoring (Windows Event Forwarding)

## 📌 Objective

Mengimplementasikan Windows Event Forwarding (WEF) agar DC01 dapat mengumpulkan event log
dari FS01 secara terpusat.

Fokus utama:

- DC01 sebagai **Collector** (mengumpulkan log)
- FS01 sebagai **Source** (mengirimkan log)
- Memanfaatkan WinRM dan Event Subscriptions
- Validasi melalui Event Viewer di DC01

---

## 🏗 Environment Context

Domain: corp.local
Domain Controller / Collector: DC01 (192.168.200.10)
File Server / Source: FS01 (192.168.200.20)
Workstation: WIN10 (192.168.200.100)

---

## 🧭 Arsitektur WEF

```text
FS01 (Source)                    DC01 (Collector)
+-----------------+              +-----------------+
| WinRM Service   | ---push----> | Event Collector |
| (port 5985)     |              | Service         |
| Forwards logs   |              | Subscription    |
+-----------------+              +-----------------+
```

Model yang digunakan: **Source-Initiated (Push)**

- FS01 secara aktif mendorong event ke DC01
- DC01 mendaftarkan komputer yang diizinkan mengirim via GPO
- Tidak memerlukan koneksi inbound dari DC01 ke FS01

---

## 🛠 Setup Langkah per Langkah

### [ DC01 ] — Konfigurasi Collector

#### 1) Aktifkan Windows Event Collector Service

Jalankan di DC01 (cmd/PowerShell sebagai Administrator):

```cmd
wecutil qc /q
```

Output yang diharapkan:

```
The service startup mode will be changed to Delay-Start.
Would you like to proceed (Y/N)?
```

Ketik `Y`. Service `Windows Event Collector` akan diaktifkan.

Verifikasi:

```cmd
sc query wecsvc
```

Pastikan `STATE` = `RUNNING`.

---

#### 2) Tambahkan DC01 ke Group "Event Log Readers" (opsional tapi direkomendasikan)

Di DC01, masuk ADUC → FS01 computer account tidak perlu diubah, tapi pastikan
`NETWORK SERVICE` dari DC01 bisa baca event FS01.

---

#### 3) Buat Event Subscription di DC01

Buka **Event Viewer → Subscriptions → klik kanan → Create Subscription**

Konfigurasi:

| Setting               | Value                                                    |
| --------------------- | -------------------------------------------------------- |
| Subscription Name     | `FS01-Security-Events`                                   |
| Subscription Type     | `Source computer initiated`                              |
| Computer Groups       | `Tier1-Server-Admins` atau buat group baru `WEF-Sources` |
| Events to Collect     | Security logs, Event ID 4624, 4625, 4672                 |
| Delivery Optimization | `Normal`                                                 |

Atau via PowerShell / wecutil (lihat bagian troubleshooting).

---

### [ FS01 ] — Konfigurasi Source

#### 4) Aktifkan WinRM di FS01

Jalankan di FS01 (cmd/PowerShell sebagai Administrator):

```cmd
winrm quickconfig -q
```

Output yang diharapkan:

```
WinRM service is already running on this machine.
WinRM is already set up for remote management on this computer.
```

Verifikasi:

```cmd
winrm enumerate winrm/config/listener
```

Pastikan ada listener di port **5985** (HTTP) atau **5986** (HTTPS).

---

#### 5) Tambahkan DC01 ke Network Service di FS01

Di FS01, tambahkan `NETWORK SERVICE` dari DC01 agar bisa baca Security log:

```cmd
wevtutil gl Security
```

Lihat `channelAccess` — pastikan `NETWORK SERVICE` memiliki read access.

Jika belum:

```cmd
wevtutil sl Security /ca:"O:BAG:SYD:(A;;0xf0005;;;SY)(A;;0x5;;;BA)(A;;0x1;;;S-1-5-32-573)(A;;0x1;;;NS)"
```

---

### [ GPO ] — Konfigurasi via Group Policy (Cara Terbaik)

Ini adalah cara yang paling scalable dan enterprise-grade.

#### 6) Buat GPO untuk mengaktifkan WinRM di Source (FS01)

Di DC01 → Group Policy Management → buat GPO baru:

Nama: `WEF-Source-Configuration`
Link ke: `OU=Tier1` (tempat FS01 berada)

Konfigurasi dalam GPO:

**a) Aktifkan WinRM Service:**

```
Computer Configuration
→ Preferences
→ Control Panel Settings
→ Services
→ New Service
  - Service Name: WinRM
  - Startup Type: Automatic
  - Service Action: Start service
```

**b) Tambahkan DC01's MACHINE ACCOUNT ke channel access FS01:**

```
Computer Configuration
→ Policies
→ Administrative Templates
→ Windows Components
→ Event Forwarding
→ Configure target Subscription Manager
  - Enabled
  - Value: Server=http://DC01.corp.local:5985/wsman/SubscriptionManager/WEC,Refresh=60
```

> ⚠️ Ganti `DC01.corp.local` dengan FQDN DC01 kamu. Bisa cek dengan `nslookup DC01`.

**c) Tambahkan NETWORK SERVICE ke Event Log Readers:**

```
Computer Configuration
→ Policies
→ Windows Settings
→ Security Settings
→ Restricted Groups
→ Add Group: "Event Log Readers"
  → Members: NETWORK SERVICE
```

---

## 🔥 Troubleshooting — Masalah Umum WEF

### ❌ Problem 1: Subscription menampilkan "Error" atau "Retrying"

**Cek di FS01:**

```cmd
winrm id
```

Kalau error → WinRM belum running.

```cmd
sc query winrm
```

Start jika belum:

```cmd
net start winrm
```

---

### ❌ Problem 2: "Access Denied" saat Subscription aktif

Penyebab: DC01's machine account tidak punya izin baca Security log di FS01.

**Solusi di FS01 (sebagai administrator):**

```cmd
wevtutil sl Security /ca:"O:BAG:SYD:(A;;0xf0005;;;SY)(A;;0x5;;;BA)(A;;0x1;;;S-1-5-32-573)(A;;0x1;;;NS)"
```

Atau tambahkan `Network Service` ke local group `Event Log Readers` di FS01:

```cmd
net localgroup "Event Log Readers" "NETWORK SERVICE" /add
```

---

### ❌ Problem 3: Event Viewer tidak menampilkan event

**Cek status subscription di DC01:**

```cmd
wecutil gr FS01-Security-Events
```

Lihat bagian `LastError` dan `RunTimeStatus`.

Jika ada error, lihat detail:

```cmd
wecutil gr FS01-Security-Events /f:xml
```

---

### ❌ Problem 4: Firewall memblokir WinRM

Di FS01, buka firewall untuk WinRM:

```cmd
netsh advfirewall firewall add rule name="WinRM HTTP" dir=in action=allow protocol=TCP localport=5985
```

Atau via PowerShell:

```powershell
Enable-NetFirewallRule -DisplayGroup "Windows Remote Management"
```

---

### ❌ Problem 5: GPO Subscription Manager belum apply

Di FS01, force apply GPO:

```cmd
gpupdate /force
gpresult /r /scope computer
```

Cari `WEF-Source-Configuration` di Applied GPOs.

Cek apakah registry key sudah ada di FS01:

```
HKLM\SOFTWARE\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager
```

---

### ❌ Problem 6: Kerberos Authentication gagal

WEF menggunakan Kerberos by default. Jika ada masalah auth:

Di DC01:

```cmd
klist
```

Test koneksi WinRM dari DC01 ke FS01:

```powershell
Test-WSMan -ComputerName FS01 -Authentication Kerberos
```

Output sukses:

```
wsmid           : http://schemas.dmtf.org/wbem/wsman/1/wsmanidentity
ProtocolVersion : http://schemas.dmtf.org/wbem/wsman/1/wsman.xsd
ProductVendor   : Microsoft Corporation
ProductVersion  : OS: ...
```

---

## 🔍 Validasi End-to-End

Setelah semua konfigurasi selesai:

### Di FS01: Trigger event

Login/logout dari WIN10 untuk generate event, atau:

```cmd
runas /user:corp\hr.staff cmd
```

(Masukkan password salah → generate Event ID 4625)

### Di DC01: Cek Forwarded Events

```
Event Viewer
→ Windows Logs
→ Forwarded Events
```

Event 4624, 4625, 4672 dari `FS01` seharusnya muncul di sini.

### Verifikasi via PowerShell di DC01:

```powershell
Get-WinEvent -LogName "ForwardedEvents" -MaxEvents 20 | Format-List TimeCreated, Id, MachineName, Message
```

---

## 🧪 Access Validation Matrix (setelah WEF berjalan)

```text
Lokasi Log     | Dapat dilihat dari
FS01 local     | Admin login langsung ke FS01
DC01 Forwarded | Admin di DC01 tanpa perlu login ke FS01
```

---

## 🧠 Prinsip yang Diterapkan

- Centralized Log Management
- Source-Initiated Event Forwarding
- Group Policy-driven automation
- Principle of Least Privilege (NETWORK SERVICE read-only)
- Kerberos-based authentication for log transport

---

## ✅ Hasil yang Diharapkan

Setelah sub-lab ini berhasil:

- DC01 menampilkan Security events dari FS01 di **Forwarded Events**
- Tidak perlu login langsung ke FS01 untuk melihat log
- Event 4624 (logon), 4625 (failed logon), 4672 (special privilege) tercatat terpusat
- Scalable: mudah ditambah source baru (server lain) tanpa konfigurasi manual

---

## 📝 Status

- [ ] WinRM aktif di FS01
- [ ] Windows Event Collector aktif di DC01
- [ ] GPO `WEF-Source-Configuration` dibuat dan di-link ke Tier1
- [ ] Subscription `FS01-Security-Events` dibuat di DC01
- [ ] NETWORK SERVICE ditambahkan ke Event Log Readers di FS01
- [ ] Firewall WinRM dibuka di FS01
- [ ] Forwarded Events muncul di DC01
