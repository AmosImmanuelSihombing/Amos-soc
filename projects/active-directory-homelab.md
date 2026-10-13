# Project Write-Up: Enterprise Active Directory Homelab & Security Auditing

- **Fokus Peran:** IT Support / Helpdesk Specialist & SOC Analyst Tier 1
- **Topik:** Identity and Access Management (IAM), Group Policy Hardening, Windows Event Logging
- **Biaya:** Rp 0 (100% Menggunakan Evaluasi Resmi Microsoft & Oracle VirtualBox)
- **Lingkungan Lab:**
  - **Hypervisor:** Oracle VirtualBox 7.x
  - **Domain Controller (DC):** Windows Server 2022 Datacenter Evaluation (Hostname: `DC01`, IP: `192.168.100.10`)
  - **Domain Name:** `corp.local`
  - **Client Endpoint:** Windows 10/11 Enterprise Evaluation (Hostname: `WS01-FIN`, IP: `192.168.100.20`)
- **Taktik MITRE ATT&CK yang Dipantau:**
  - `T1110.001` - Brute Force: Password Guessing
  - `T1078.002` - Valid Accounts: Domain Accounts
  - `T1003` - OS Credential Dumping

---

## 1. Ringkasan Eksekutif
Proyek ini mengimplementasikan infrastruktur jaringan enterprise terkelola menggunakan **Active Directory Domain Services (AD DS)**. Proyek mencakup perancangan struktur *Organizational Unit* (OU), pengelolaan *Role-Based Access Control* (RBAC), penerapan kebijakan keamanan ketat melalui *Group Policy Objects* (GPO), serta pengaktifan *Advanced Security Auditing* untuk mendeteksi anomali autentikasi dan penanganan insiden *Account Lockout* di level Helpdesk & SOC Tier 1.

---

## 2. Topologi Jaringan & Arsitektur Lab

```
                   [ VirtualBox Host (Physical Laptop) ]
                                     |
              +----------------------+----------------------+
              | (VirtualBox Network: Internal / NAT Network)|
              v                                             v
    [ Windows Server 2022 ]                       [ Windows 10/11 Client ]
      Hostname: DC01                                Hostname: WS01-FIN
      IP: 192.168.100.10 /24                        IP: 192.168.100.20 /24
      Roles: AD DS, DNS, DHCP                       DNS: 192.168.100.10 (Points to DC)
      Domain: corp.local                            Domain Joined: corp.local
```

---

## 3. Perancangan Struktur Active Directory (OU & Groups)

Untuk mencegah pemberian hak akses berlebih (*Principle of Least Privilege*), struktur direktori dirancang hierarkis:

```
corp.local/
├── CORP_ADMINISTRATION/
│   ├── OU_Admins/ (Akun Domain Admins terpisah dari akun harian)
│   └── OU_Helpdesk/ (Tier 1 Support dengan hak delegasi reset password)
├── CORP_DEPARTMENTS/
│   ├── OU_Finance/ (User: siti.aminah, budi.santoso)
│   └── OU_IT_Operations/ (User: amos.sihombing)
└── CORP_ASSETS/
    ├── OU_Servers/ (Server DC01, File Server)
    └── OU_Workstations/ (Komputer klien joined: WS01-FIN)
```

### Delegasi Hak Akses (Delegation of Control):
Kelompok `SEC_Helpdesk_L1` hanya diberikan hak khusus:
- *Reset user passwords and force password change at next logon*
- *Read/Write lockoutTime (Membuka kunci akun yang terblokir)*
- *TIDAK memiliki hak Administrator Domain*, menjaga server tetap aman sesuai prinsip *Zero Trust*.

---

## 4. Konfigurasi Group Policy Object (GPO) Hardening

Kebijakan keamanan diterapkan secara terpusat melalui Group Policy Management Console (`gpmc.msc`):

| Nama Kebijakan GPO | Pengaturan / Parameter | Tujuan Keamanan |
|---|---|---|
| **Account Lockout Policy** | Lockout Threshold: `5 invalid logon attempts`<br>Lockout Duration: `15 minutes`<br>Reset counter after: `15 minutes` | Mencegah serangan *online brute force* & *password spraying*. |
| **Password Policy** | Min. Length: `12 characters`<br>Complexity: `Enabled`<br>Max Age: `60 days`<br>History: `10 passwords remembered` | Memastikan kekuatan kredensial terhadap serangan *dictionary attack*. |
| **Advanced Audit Policy** | - Audit Logon: `Success & Failure`<br>- Audit Account Lockout: `Success`<br>- Process Creation: `Include commandLine` | Menghasilkan telemetri forensik ke Windows Event Log untuk analisis SOC. |
| **Insecure Protocol Disablement** | - Disable SMBv1 Client & Server<br>- Enforce NTLMv2 only (Refuse LM & NTLM) | Mitigasi eksploitasi legacy (seperti *EternalBlue/WannaCry*). |

---

## 5. Simulasi Insiden Operasional & Investigasi Event Log

### Skenario Insiden: Account Lockout & Brute Force Triage
1. **Pemicu:** Pengguna pada workstation `WS01-FIN` (atau penyerang) mencoba memasukkan password salah 5 kali berturut-turut pada akun `CORP\budi.santoso`.
2. **Respons Sistem:** Pada percobaan ke-5, Windows secara otomatis mengunci akun pengguna dan menolak seluruh upaya autentikasi berikutnya.
3. **Analisis SOC / Helpdesk via Event Viewer (`eventvwr.msc`):**

```
Log Name:      Security
Source:        Microsoft-Windows-Security-Auditing
Event ID:      4625 (An account failed to log on)
Account Name:  budi.santoso
Failure Reason: Unknown user name or bad password. (0xC000006A)
Workstation:   WS01-FIN
Caller IP:     192.168.100.20

--- TERUS MENERUS HINGGA MEMICU ---

Event ID:      4740 (A user account was locked out)
Target Account Name: budi.santoso
Subject Domain:      CORP
Caller Computer Name: WS01-FIN
Time:                2026-10-03 01:25:12
```

4. **Prosedur Resolusi Helpdesk Tier 1:**
   - Memverifikasi identitas pengguna via saluran terkonfirmasi (telepon/pesan internal).
   - Membuka *Active Directory Users and Computers* (`dsa.msc`).
   - Masuk ke properti user `budi.santoso` &rarr; tab *Account* &rarr; centang `Unlock account`.
   - Mengatur `User must change password at next logon`.
   - Mendokumentasikan nomor tiket pada sistem service desk dengan catatan resolusi.

---

## 6. Pembelajaran & Nilai Tambah untuk Karier

1. **Pemahaman Telemetri Windows:** Menguasai pembacaan Event ID kritis (`4624` Login Sukses, `4625` Login Gagal, `4688` Pembuatan Proses, `4740` Lockout, `4720` Pembuatan Akun Baru).
2. **Keahlian Troubleshooting Helpdesk:** Terbiasa dengan masalah umum autentikasi jaringan, sinkronisasi DNS Domain, dan pemecahan masalah hak akses user.
3. **Pondasi Blue Team:** Memahami bahwa deteksi insiden di tingkat endpoint enterprise selalu berakar pada arsitektur direktori terpusat (Active Directory).
