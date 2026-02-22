# Sub-Lab 06 – Tiered Administrative Model

## 📌 Objective

Implement Tiered Administrative Model (Tier 0/1/2) untuk memisahkan hak akses administrator berdasarkan tingkat sensitivitas sistem.

Fokus utama:

- Mencegah penggunaan Domain Admin untuk pekerjaan sehari-hari
- Menerapkan principle of least privilege
- Mengimplementasikan delegated administration untuk helpdesk
- Membuat boundary antara Domain Controller, Member Server, dan Workstation

---

## 🏗 Environment Context

Domain: corp.local  
Domain Controller: DC01 (192.168.200.10)  
File Server: FS01 (192.168.200.20)  
Workstation: WIN10 (192.168.200.100)

---

## 🧱 OU Structure Redesign (Tier Model)

Struktur OU diubah menjadi:

```text
corp.local
|-- Domain Controllers
|   |-- DC01
|-- Tier0
|-- Tier1
|   |-- Servers
|   |   |-- FS01
|-- Tier2
|   |-- Workstations
|   |   |-- WIN10
|-- Users
|   |-- HR
|   |-- Finance
|   |-- IT
|-- Admin
|   |-- admin.t0
|   |-- admin.t1
|   |-- admin.t2
|-- Groups
|   |-- Tier0-Domain-Admins
|   |-- Tier1-Server-Admins
|   |-- Tier2-Helpdesk-Admins
```

Perpindahan computer objects:

- FS01 ke `Tier1\Servers`
- WIN10 ke `Tier2\Workstations`

---

## 👤 Dedicated Administrative Accounts

Akun admin terpisah dari user bisnis:

- `admin.t0`: Domain-level administration
- `admin.t1`: Member server administration
- `admin.t2`: Helpdesk / password reset

Semua akun admin ditempatkan di OU `Admin`.

---

## 🔐 Security Groups (Role-Based Access)

Security group dengan Global Scope:

- `Tier0-Domain-Admins`: Domain-level privilege
- `Tier1-Server-Admins`: Member server local admin
- `Tier2-Helpdesk-Admins`: Delegated helpdesk privilege

Group disimpan di OU `Groups`.

---

## 🧩 Nested Group for Domain Admin (Tier0)

Best practice: tidak menambahkan user langsung ke `Domain Admins`.

Nested group:

```text
admin.t0
  -> Tier0-Domain-Admins
     -> Domain Admins
```

Hasil:

- `admin.t0` mendapatkan privilege Domain Admin
- Tidak ada user bisnis menjadi Domain Admin

---

## 🛠 Delegated Administration (Tier2)

Delegation pada OU `Users` diberikan ke:

- `Tier2-Helpdesk-Admins`

Hak yang diberikan:

- Reset user password
- Force password change at next logon
- Unlock account

Validasi:

- `admin.t2` dapat reset password HR/Finance
- `admin.t2` tidak dapat modify `Domain Admins`
- `admin.t2` tidak dapat membuat OU baru
- `admin.t2` tidak dapat mengedit GPO
- Helpdesk tidak bisa login ke Domain Controller

---

## 🖥 Server-Level Administration (Tier1)

Pada `FS01`, group `Tier1-Server-Admins` ditambahkan ke:

- Local Administrators (FS01)

Dampak:

- `admin.t1` dapat login ke `FS01`
- `admin.t1` memiliki local admin rights di `FS01`
- `admin.t1` tidak memiliki Domain Admin privilege
- `admin.t1` tidak dapat login ke `DC01`

---

## 🔍 Access Validation Matrix

```text
Account     | Login DC | Login FS01 | Reset User | Domain Admin
admin.t0    | YES      | YES        | YES        | YES
admin.t1    | NO       | YES        | NO         | NO
admin.t2    | NO       | NO         | YES        | NO
HR User     | NO       | NO         | NO         | NO
```

---

## ✅ Security Model Achieved

Lab ini berhasil mengimplementasikan:

- Tiered Administrative Model
- Least Privilege Principle
- Role-Based Access Control (RBAC)
- Nested Group Strategy
- Delegated Administration
- Separation of Duties

---

## 🧠 Key Learning Outcomes

- OU bukan untuk privilege, hanya untuk organization dan GPO scope
- Permission harus diberikan via group, bukan langsung ke user
- Domain Admin tidak boleh digunakan untuk daily operation
- Helpdesk harus delegated, bukan privileged
- Member Server admin tidak boleh login Domain Controller

---

## 🏢 Enterprise Relevance

Model ini menyerupai Microsoft ESAE / Red Forest principle secara simplified.

Desain ini mencegah:

- Privilege escalation
- Credential theft lateral movement
- Overprivileged account misuse
