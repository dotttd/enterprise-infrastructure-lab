# Sub-Lab 08 – Server Hardening Baseline

## 📌 Objective

Menerapkan baseline hardening pada server Tier1 (`FS01`) untuk mensimulasikan praktik keamanan enterprise modern.

Cakupan utama:

- Advanced audit logging
- RDP access restriction
- Firewall enforcement
- SMB protocol hardening
- Anonymous access restriction
- Validasi kontrol keamanan melalui pengujian langsung

Sub-lab ini mengubah environment dari sekadar "functional infrastructure" menjadi secured and controlled infrastructure.

---

## 🏗 Environment Context

Domain: corp.local  
Domain Controller: DC01 (192.168.200.10)  
File Server: FS01 (192.168.200.20)  
Workstation: WIN10 (192.168.200.100)

---

## 🧭 Arsitektur yang Digunakan

Tier Model:

```text
Tier0 -> Domain Controller
Tier1 -> Member Server (FS01)
Tier2 -> Workstation (WIN10)
```

GPO diterapkan melalui OU-based scoping.

GPO utama yang dibuat:

- `Tier1-Security-Baseline`

Diterapkan ke:

- `OU=Tier1`

---

## 🛠 Implementasi yang Dilakukan

### 1) Advanced Audit Policy Configuration

Menggunakan modern granular audit configuration.

Path:

```text
Computer Configuration
-> Policies
-> Windows Settings
-> Security Settings
-> Advanced Audit Policy Configuration
-> Audit Policies
```

Konfigurasi aktif:

Logon/Logoff:

- Audit Logon -> Success + Failure
- Audit Special Logon -> Success

Account Logon:

- Audit Credential Validation -> Success + Failure

Tambahan enforcement:

- `Audit: Force audit policy subcategory settings` = Enabled

Tujuan: memastikan Advanced Audit override legacy audit policy.

---

### 2) RDP Restriction (Tier Enforcement)

Pembatasan akses RDP sesuai Tier Model.

Path:

```text
User Rights Assignment
-> Allow log on through Remote Desktop Services
```

Isi:

- `Administrators`
- `Tier1-Server-Admins`

`Deny` dikosongkan untuk menghindari override conflict.

Hasil validasi:

```text
User      | RDP ke FS01
admin.t0  | YES
admin.t1  | YES
admin.t2  | NO
HR/Finance| NO
```

Verifikasi melalui `mstsc` dan Event ID `4624/4625`.

---

### 3) Firewall Enforcement

Mengaktifkan Windows Defender Firewall pada Domain Profile.

Validasi:

```text
Get-NetFirewallProfile
```

Hasil:

- Domain Profile -> Enabled: True

RDP diuji:

```text
Test-NetConnection 192.168.200.20 -Port 3389
```

---

### 4) SMB Hardening

SMBv1 diperiksa dan dinonaktifkan (jika tersedia):

```text
Get-WindowsFeature FS-SMB1
Uninstall-WindowsFeature FS-SMB1
```

Tujuan:

- Menghilangkan protokol legacy rentan (WannaCry class attack)

---

### 5) Restrict Anonymous Enumeration

Mengaktifkan:

- `Network access: Do not allow anonymous enumeration of SAM accounts`

Validasi:

```text
net view \\192.168.200.20
```

Hasil:

- Anonymous access ditolak
- Credential required

---

## 🔍 Validasi Keamanan

Event Viewer:

```text
Windows Logs -> Security
```

Log berhasil tercatat:

```text
Event ID | Arti
4624     | Successful logon
4625     | Failed logon
4672     | Special privilege logon
```

---

## 🧠 Prinsip Keamanan yang Diterapkan

- Least Privilege
- Tiered Administrative Model
- Attack Surface Reduction
- Credential Isolation
- Centralized Policy Enforcement
- Auditable Infrastructure

---

## ✅ Hasil Akhir

Setelah sub-lab ini:

- Server hanya dapat diakses oleh admin Tier1
- Akses login tercatat dan dapat diaudit
- Protokol legacy dinonaktifkan
- Anonymous enumeration diblokir
- Firewall aktif pada Domain profile

Infrastructure tidak hanya berjalan, tetapi:

- Secure by design and validated through testing

---

## 🏢 Dampak terhadap Enterprise Simulation

Sub-lab ini memposisikan lab sebagai:

- Simulasi enterprise-grade environment
- Implementasi baseline hardening nyata
- Model segmentasi akses berdasarkan tier
