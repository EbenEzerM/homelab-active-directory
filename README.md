# Dokumentasi Virtual Home Lab Active Directory Domain Services (AD DS) Menggunakan VirtualBox

## 1. Pendahuluan
Dokumentasi ini menjelaskan langkah-langkah membangun lingkungan virtual home lab Active Directory Domain Services (AD DS) menggunakan Oracle VirtualBox.

Dokumentasi mencakup:
- Instalasi dan konfigurasi Active Directory Domain Services
- Pembuatan Organizational Unit (OU)
- Pembuatan User dan Group
- Konfigurasi Group Policy Object (GPO)
- Konfigurasi File Sharing
- Join Client ke Domain

---

## 2. Topologi Lab

### Server dan Client
| Hostname | OS | Fungsi | IP Address |
|---|---|---|---|
| DC01 | Windows Server 2019 | Domain Controller + DNS | 192.168.10.10 |
| CLIENT01 | Windows 10/11 | Client Domain | 192.168.10.20 |

### Domain
| Item | Value |
|---|---|
| Domain Name | homelab.local |
| NetBIOS Name | HOMELAB |

---

## 3. Persiapan Virtual Machine
### Install Oracle VirtualBox
Unduh dan install Oracle VirtualBox dari: https://www.virtualbox.org

### Install Windows Server 2019
Unduh dan install Windows 2019 dari: https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019

### Install Windows 10/11
Unduh dan install Windows 10/11 dari: https://www.microsoft.com/software-download/windows10 atau https://www.microsoft.com/software-download/windows11

### Langkah Membuat VM di VirtualBox

1. Buka **Oracle VirtualBox**, klik **New** untuk membuat VM baru.
2. Masukkan nama VM sesuai peran:
   - `DC01` untuk Domain Controller
   - `CLIENT01` untuk Client
3. Pilih tipe OS:
   - **Microsoft Windows**, versi sesuai ISO (Server 2019 atau Windows 10/11).
4. Atur **Memory Size**:
   - DC01: 1536 MB
   - CLIENT01: 1536 MB
5. Buat virtual hard disk baru:
   - Format: VDI
   - Storage: Dynamically allocated
   - Size: 20 GB
6. Setelah VM dibuat, buka **Settings**:
   - **System → Processor**: set sesuai spesifikasi (DC01: 2 core, CLIENT01: 1 core).
   - **Network → Adapter 1**: pilih **Internal Network / Host-Only Adapter**.
7. Masukkan ISO OS ke **Storage → Optical Drive**.
8. Klik **Start** untuk memulai instalasi OS.
9. Ikuti wizard instalasi Windows Server 2019 atau Windows 10/11 hingga selesai.

---

## 4. Instalasi Active Directory Domain Services
### Konfigurasi IP Address Server
| Setting | Value |
|---|---|
| IP Address | 192.168.10.10 |
| Subnet Mask | 255.255.255.0 |
| Gateway | 192.168.10.1 |
| Preferred DNS | 192.168.10.10 |

### Install Role AD DS
1. Buka **Server Manager** → **Add Roles and Features**.  
2. Pilih:  
   - Active Directory Domain Services  
   - DNS Server  
3. Klik **Install**.  

### Promote Server Menjadi Domain Controller
1. Klik notifikasi bendera di Server Manager.  
2. Pilih **Promote this server to a domain controller**.  
3. Pilih **Add a new forest**.  
4. Isi root domain: `homelab.local`.  
5. Tentukan password DSRM.  
6. Klik **Install** → server akan restart otomatis.  

---

## 5. Membuat Organizational Unit (OU)

### Membuka Active Directory Users and Computers
Jalankan:
```text
dsa.msc
```

### Struktur OU
```text
Company
├── Computers
│   ├── HR
│   ├── IT
│   └── Finance
├── Users
│   ├── HR
│   ├── IT
│   └── Finance
└── Servers
```

### Langkah Membuat OU
1. Klik kanan domain ``homelab.local``.
2. Pilih **New → Organizational Unit**.
3. Buat OU utama: ``Company``.
4. Di dalam OU Company, buat OU: ``Computers``, ``Users``, dan ``Servers``.
5. Tambahkan sub-OU sesuai divisi (HR, IT, Finance) di dalam OU ``Computers`` dan ``Servers``.

---

## 6. Membuat User dan Group
### Membuat User
1. Masuk ke OU Users → OU divisi.
2. Klik kanan → **New** → **User**.
3. Isi data user dan password.
4. Hilangkan centang **User must change password at next logon**.
5. Klik **Finish**.

### Membuat Group
1. Masuk ke OU Users → OU divisi.
2. Klik kanan → **New** → **Group**.
3. Contoh group:
   - HR-Staff
   - IT-Staff
   - Finance-Staff

### Menambahkan User ke Group
1. Klik kanan user → **Add to a group**.
2. Masukkan nama group (misalnya ``IT-Staff``).
3. Klik **OK**.

---

## 7. Join Client ke Domain

### Konfigurasi IP Client
| Setting | Value |
|---|---|
| IP Address | 192.168.10.20 |
| Subnet Mask | 255.255.255.0 |
| Gateway | 192.168.10.1 |
| DNS Server | 192.168.10.10 |

### Join Domain
1. Buka **Settings** → **System** → **About**.
2. Klik **Rename this PC (Advanced)** → **Change**.
4. Ubah Computer Name: ``CLIENT01``.
5. Pilih Domain, isi: ``homelab.local``.
6. Masukkan credential: ``HOMELAB\Administrator``.
7. Restart komputer

### Login Menggunakan User Domain
1. Pada login screen, pilih **Other User**.
2. Masukkan username dan password yang sudah terdaftar di Active Directory
3. Contoh:
   - Username: budi
   - Password: ********

---

## 8. Membuat Group Policy Object (GPO)

### Membuka Group Policy Management
Tekan **Windwows** + **r**, lalu jalankan:
```text
gpmc.msc
```

### Membuat GPO Baru
Contoh: **Disable Control Panel**
1. Klik kanan OU Users → **Create a GPO in this domain, and Link it here**.
2. Nama GPO: ``Disable Control Panel``.

### Konfigurasi Policy
Masuk ke:
```text
User Configuration → Policies → Administrative Templates → Control Panel
```

Aktifkan policy:
- **Prohibit access to Control Panel and PC settings** → Enabled

### Update GPO pada Client
Jalankan CMD sebagai Administrator:
```text
gpupdate /force
```
---

## 9. Konfigurasi File Sharing
### Membuat Folder Sharing
Contoh folder:
```text
C:\SharedData
```

### Share Folder
1. Klik kanan folder → **Properties** → **Sharing** → **Advanced Sharing**.
2. Centang **Share this folder** → **Permissions**.

### Konfigurasi Permission
| Group/User | Permission |
|---|---|
| IT-Staff | Full Control |
| HR-Staff | Read |

### Akses Shared Folder dari Client
Pada File Explorer masuk ke:
```text
\DC01\SharedData
```

Atau menggunakan IP:
```text
\192.168.10.10\SharedData
```
---

## 10. Verifikasi dan Testing
### Testing DNS
Pada CMD di client, jalankan:
```text
ping dc01
nslookup homelab.local
```

### Testing GPO
Coba buka Control Panel. Jika akses diblok → GPO berhasil.

### Testing File Sharing
Akses ``\\DC01\SharedData`` → cek permission sesuai group.

---

## 11. Troubleshooting

### Client Gagal Join Domain
Periksa:
- DNS client harus mengarah ke Domain Controller
- Pastikan waktu server dan client sinkron
- Pastikan network adapter berada pada network yang sama

### GPO Tidak Berjalan
Periksa dengan:
```text
gpupdate /force
gpresult /r
```

### Shared Folder Tidak Bisa Diakses
Periksa:
- Sharing Permission
- NTFS Permission
- Firewall Windows

---

## 12. Kesimpulan
Dengan mengikuti dokumentasi ini, lingkungan virtual home lab Active Directory dapat digunakan untuk:
- Simulasi administrasi jaringan Windows
- Manajemen user dan group
- Implementasi Group Policy
- Simulasi file server
- Pembelajaran domain environment menggunakan VirtualBox

---

## 13. Teknologi yang Digunakan
- Windows Server 2019
- Windows 10/11
- Oracle VirtualBox
- Active Directory Domain Services (AD DS)
- DNS Server
- Group Policy Management
