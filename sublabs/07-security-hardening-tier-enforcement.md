# Sub-Lab 07 – Security Hardening & Tier Enforcement

## 📌 Objective

Mengimplementasikan logon restriction berbasis Tier Model agar pemisahan hak akses administratif benar-benar ter-enforce.

Fokus utama:

- Mencegah Tier2 dan user bisnis login ke server
- Membatasi akses Domain Controller hanya untuk Tier0
- Memastikan Tier1 hanya mengelola server
- Meng-enforce boundary menggunakan Group Policy
- Memvalidasi hasil melalui testing login nyata

---

## 🏗 Environment Context

Domain: corp.local  
Domain Controller: DC01 (192.168.200.10)  
File Server: FS01 (192.168.200.20)  
Workstation: WIN10 (192.168.200.100)

---

## 🧭 Konsep Arsitektur

Model yang digunakan adalah Tiered Administrative Model:

```text
Tier   | Fungsi                | Scope Akses
Tier0  | Domain Administration | Domain Controller
Tier1  | Server Administration | Member Servers
Tier2  | Helpdesk              | Workstations
Users  | Business Users        | Workstations Only
```

Prinsip utama:

- Lower tier tidak boleh mengakses higher tier

---

## 🛠 Implementasi Teknis

### 1) Pembuatan GPO Logon Restriction

GPO dibuat dan di-link ke OU `Tier1`.

Nama GPO:

- `Tier1-Logon-Restriction`

Karena logon restriction berbasis computer policy, GPO diterapkan pada OU tempat server berada.

---

### 2) Konfigurasi Allow Log On Locally

Path konfigurasi:

```text
Computer Configuration
-> Policies
-> Windows Settings
-> Security Settings
-> Local Policies
-> User Rights Assignment
-> Allow log on locally
```

Konfigurasi:

- Define these policy settings
- Tambahkan hanya:
- `Administrators`
- `Tier1-Server-Admins`

Tidak menyertakan:

- `Domain Users`
- `Authenticated Users`
- `Users`

Dampak:

- Hanya Tier1 dan administrator yang dapat login ke FS01

---

### 3) Penghapusan Deny Log On Locally

Untuk menghindari konflik override (Deny > Allow), digunakan model Allow-only.

Konfigurasi:

- `Deny log on locally` = Not Defined

---

### 4) Verifikasi Policy Application

Di `FS01` dijalankan:

```text
gpupdate /force
gpresult /r /scope computer
```

Memastikan `Tier1-Logon-Restriction` masuk Applied GPO pada Computer Scope.

---

### 5) Validasi Akses Login

Pengujian login langsung ke `FS01` menggunakan beberapa akun.

Hasil validasi:

```text
Account   | Login FS01
admin.t0  | YES
admin.t1  | YES
admin.t2  | NO
HR user   | NO
Finance   | NO
```

---

### 6) Privilege Adjustment (Shutdown Rights)

Ditemukan `admin.t1` bisa login tetapi tidak bisa restart server.

Analisis:

- Login privilege != System shutdown privilege

Hak restart diatur pada:

```text
User Rights Assignment -> Shut down the system
```

Solusi:

- Menambahkan `Tier1-Server-Admins` ke `Shut down the system`
- Atau memastikan group tersebut menjadi member local `Administrators` di `FS01`

---

## ✅ Hasil Akhir

Setelah implementasi:

- Tier separation enforced
- Business users tidak bisa login server
- Tier2 tidak bisa login server
- Tier1 hanya bisa akses server
- Privilege granular mulai terlihat
- GPO berhasil override default member server behavior

Sub-lab ini membuktikan bahwa desain Tier Model bukan hanya struktur OU, tetapi benar-benar diterapkan melalui security policy.

---

## 🧪 Catatan Teknis

- User Rights Assignment sering membutuhkan restart untuk fully apply
- `gpresult /r /scope computer` wajib digunakan untuk validasi Computer Configuration
- Default behavior Windows member server mengizinkan Domain Users login jika tidak di-override
- Model Allow-only lebih stabil dibandingkan Deny-based restriction untuk skenario ini

---

## 🧠 Insight Arsitektural

Sub-lab ini meningkatkan maturity lab dari:

- "AD Deployment Lab"

Menjadi:

- "Security Enforced Enterprise AD Simulation"

Konsep yang diterapkan di sini merupakan praktik umum pada:

- Enterprise Corporate Infrastructure
- Financial Institutions
- Mining dan Industrial Networks
- Zero Trust-aligned Active Directory Design
