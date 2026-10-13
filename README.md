# Portofolio SOC Analyst Tier 1 & Cyber Security

Template portofolio interaktif dan modern khusus kandidat **SOC Analyst Tier 1**, **Blue Team**, atau **IT Support / Network Monitoring**.

Dibangun dengan arsitektur **HTML5 + Tailwind CSS + Vanilla JS** tanpa dependensi build tools yang rumit, sehingga dapat langsung di-deploy secara instan dan gratis ke **GitHub Pages**, **Vercel**, atau **Netlify**.

---

## 🚀 Fitur Unggulan Portofolio

1. **Cyber & SOC Operational Aesthetic:**
   - Desain Dark Mode profesional standar ruang operasi SOC.
   - Terminal telemetri live simulation pada bagian hero.
   - Pembedaan badge taktik MITRE ATT&CK (T1110, T1003, T1059, dll).
2. **Interactive Alert Triage Simulator:**
   - Pengunjung / recruiter dapat menguji pemikiran analitis Anda secara langsung dalam mengevaluasi log mentah, checklist investigasi, dan menentukan keputusan *True Positive* vs *False Positive*.
3. **4 Studi Kasus & Lab Hands-on Lengkap:**
   - **Wazuh SIEM:** Homelab detection engineering & active response.
   - **Splunk BOTS:** Investigasi kompromi web server & C2 reverse shell.
   - **Phishing Triage:** Ekstraksi header email, SPF/DKIM check, ANY.RUN sandbox, dan YARA rule.
   - **Wireshark Forensics:** Deteksi DNS Tunneling exfiltration & HTTP jitter beaconing.
4. **Modal Dialog Interaktif:**
   - Pengunjung dapat membaca ringkasan teknis proyek langsung di halaman tanpa harus berpindah tab.
5. **Mobile & Desktop Responsive:**
   - Tampilan optimal di layar ponsel hingga monitor ultrawide.

---

## 📂 Struktur Direktori Proyek

```
soc-portfolio/
├── index.html                                 # Halaman utama portofolio (HTML/CSS)
├── README.md                                  # Panduan proyek & profil
└── projects/                                  # Dokumentasi teknis
    ├── ddos-threshold-defense-system.md       # Dokumentasi Tugas Akhir (DDoS Detection & iptables) [Selesai]
    ├── fortify-dast-enterprise-infrastructure.md # Infrastruktur Server Magang (Fortify DAST & SSC) [Selesai]
    └── active-directory-homelab.md            # Dokumentasi Lab Active Directory & GPO [On-Going]
```

---

## 👤 Informasi Profil Pemilik
- **Nama:** Amos Immanuel Sihombing
- **Pendidikan:** S1 Informatika - Universitas Pembangunan Jaya (2026)
- **WhatsApp:** [+62 896-3626-6115](https://wa.me/6289636266115)
- **Email:** `amosimmanuel3@gmail.com`
- **LinkedIn:** [Amos Immanuel Sihombing](https://www.linkedin.com/in/amos-immanuel-810316427)
- **JobStreet:** [JobStreet Profile](https://id.jobstreet.com/id/profiles/amos-immanuelsihombing-tfzft6bg4r)

---

## 🌐 Panduan Deploy Gratis ke GitHub Pages (Hanya 3 Menit)

Agar portofolio Anda memiliki tautan publik gratis seperti `https://username.github.io/soc-portfolio/`:

### Langkah 1: Buat Repository Baru di GitHub
1. Buka [GitHub](https://github.com) dan buat repository baru bernama `soc-portfolio` (atau `<username>.github.io` jika ingin menjadi domain utama akun Anda).
2. Setel visibilitas sebagai **Public**.

### Langkah 2: Unggah Berkas via Terminal / Git
Jalankan perintah berikut di folder proyek:

```bash
git init
git add .
git commit -m "feat: initial SOC Tier 1 portfolio website"
git branch -M main
git remote add origin https://github.com/<USERNAME-ANDA>/soc-portfolio.git
git push -u origin main
```

### Langkah 3: Aktifkan GitHub Pages
1. Pada halaman repository GitHub Anda, klik menu **Settings**.
2. Di sidebar kiri, pilih menu **Pages**.
3. Di bawah bagian **Build and deployment**:
   - **Source:** Pilih `Deploy from a branch`
   - **Branch:** Pilih `main` dan folder `/(root)`
4. Klik **Save**.
5. Tunggu sekitar 1-2 menit, situs portofolio Anda sudah online dan dapat dibagikan di CV atau LinkedIn!

---

## 💡 Tips Menambahkan Nilai Plus di Mata Recruiter & SOC Manager
- Tautkan repository GitHub asli tempat Anda menyimpan script parser Python, kustom rule Wazuh XML, atau rule YARA.
- Masukkan screenshot dashboard Wazuh atau Splunk Anda jika telah menjalankan lab homelab.
- Sertakan link profil TryHackMe atau sertifikasi Coursera / Cisco di kartu sertifikasi.
