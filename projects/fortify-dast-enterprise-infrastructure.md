# Project Write-Up: Enterprise DAST Server Infrastructure & Deployment (Micro Focus Fortify)

- **Peran:** Cyber Security / Infrastructure Intern
- **Nama Pelaksana:** Amos Immanuel Sihombing
- **Teknologi Utama:**
  - **Hypervisor Enterprise:** VMware ESXi 7.0 U2
  - **Platform Keamanan:** Micro Focus / OpenText Fortify ScanCentral DAST & Software Security Center (SSC)
  - **Sistem Operasi:** Windows Server 2019 Standard (Domain Joined: `test.lab`)
  - **Application Server & Database:** Apache Tomcat 9.x, Microsoft IIS, Microsoft SQL Server 2019/2022 (SSMS)
  - **Runtime & Komponen:** OpenJDK / Java JDK 11, .NET Framework 4.8, ASP.NET 4.8

---

## 1. Ringkasan Eksekutif
Dalam program magang ini, dirancang dan dibangun infrastruktur server enterprise untuk mendukung pengujian keamanan aplikasi web secara dinamis (*Dynamic Application Security Testing* / DAST) menggunakan **Fortify ScanCentral DAST** dan **Fortify Software Security Center (SSC)**. 

Proyek ini mencakup penyiapan mesin virtual enterprise pada **VMware ESXi**, instalasi dan konfigurasi server aplikasi (Apache Tomcat & IIS), konfigurasi basis data relasional (Microsoft SQL Server), instalasi modul lisensi terpusat (*License and Infrastructure Manager* / LIM), serta konfigurasi inisialisasi setup wizard SSC hingga sistem siap mengelola dan menganalisis kerentanan aplikasi.

---

## 2. Arsitektur Sistem Fortify ScanCentral DAST

```
+-------------------------------------------------------------------------+
|                  CORE SERVER (VMware ESXi Host)                         |
|                                                                         |
|  +------------------------------+     +-------------------------------+ |
|  | Software Security Center     |     | Software Security Center DB   | |
|  | (SSC - Apache Tomcat / Java) |<--->| (MSSQL: db_ssc)               | |
|  +------------------------------+     +-------------------------------+ |
|                 ^                                     ^                 |
|                 | REST API                            |                 |
|  +--------------v---------------+     +---------------v---------------+ |
|  | ScanCentral DAST Core        |     | ScanCentral DAST DB           | |
|  | (Global, Utility, API)       |<--->| (Konfigurasi & Hasil Scan)    | |
|  +------------------------------+     +-------------------------------+ |
|           ^           ^                                                 |
|           | Lisensi   | Endpoint API                                    |
|  +--------v----+      |                                                 |
|  | LIM Server  |      |                                                 |
|  | (IIS / .NET)|      |                                                 |
|  +-------------+      |                                                 |
+-----------------------|-------------------------------------------------+
                        |
                        v
         +------------------------------+
         | WebInspect Sensor Engine     |
         | (Automated DAST Scanner VM)  |
         +------------------------------+
```

---

## 3. Rincian Implementasi Infrastruktur & Server

### Server 1: Fortify Software Security Center (SSC)
- **Hostname:** `Tes-FortifySSC`
- **IP Address:** `192.168.1.121:8080`
- **Lingkungan:** Windows Server 2019 (2 vCPU, 4GB RAM) pada VMware ESXi 7.0
- **Tahapan Instalasi & Konfigurasi:**
  1. **Prasyarat Sistem:** Instalasi Microsoft JDK 11 dan web container Apache Tomcat 9.x.
  2. **Konfigurasi Database:** Membuat basis data `db_ssc` pada Microsoft SQL Server menggunakan SQL Server Management Studio (SSMS).
  3. **Deploy Web Application:** Menempatkan artefak `ssc.war` pada direktori `webapps/` Apache Tomcat dan memverifikasi *startup service*.
  4. **Setup Wizard SSC:**
     - Mengunggah berkas lisensi resmi Fortify.
     - Mengonfigurasi direktori sistem `C:\Users\administrator.LAB\.fortify\ssc`.
     - Validasi HTTP Host Header & konfigurasi parameter pencarian global.
     - Menghubungkan koneksi JDBC ke Microsoft SQL Server (`db_ssc`).
     - Melakukan *data seeding*: mengimpor templat proses (`Fortify_Process_Seed_Bundle`) dan laporan kepatuhan (`Fortify_Report_Seed_Bundle`).
  5. **Verifikasi:** Login user admin dan memastikan Dashboard Vulnerability (*Issue Stats*, *Application Versions*, *Remediation Tracker*) berfungsi normal.

---

### Server 2: License and Infrastructure Manager (LIM 22.2)
- **Hostname:** `Tes-LIM` (Full Name: `Tes-LIM.test.lab`)
- **IP Address:** `192.168.1.123`
- **Domain:** Terhubung ke Active Directory Domain `test.lab`
- **Fungsi:** Mengelola lisensi terpusat bagi sensor scanner WebInspect dan API DAST.
- **Tahapan Konfigurasi:**
  1. Penyiapan VM Windows Server 2019 pada hypervisor VMware ESXi 7.0.
  2. Instalasi peran Server Web (IIS) beserta komponen pendukung: ASP.NET 4.8, .NET Extensibility, ISAPI Extensions/Filters, dan .NET Framework 4.8.
  3. Pemasangan installer `LocalLicenseServer64.msi` (Fortify Security LIM 22.2).
  4. Inisialisasi dan aktivasi lisensi server lokal.

---

## 4. Nilai Tambah & Keahlian untuk Karier Cyber Security & IT Operations

1. **Enterprise Virtualization (VMware ESXi):** Berpengalaman langsung mengoperasikan hypervisor bare-metal standar industri korporat untuk provisioning server.
2. **Manajemen Kerentanan (Vulnerability Management / DAST):** Memahami arsitektur pengujian keamanan aplikasi dinamis (DAST), integrasi sensor WebInspect, dan pelaporan kerentanan terpusat.
3. **Administrasi Server & Middleware:** Menguasai deployment aplikasi berbasis Java/WAR pada Apache Tomcat, pengelolaan web server Microsoft IIS, integrasi domain Active Directory (`test.lab`), dan administrasi basis data Microsoft SQL Server.
