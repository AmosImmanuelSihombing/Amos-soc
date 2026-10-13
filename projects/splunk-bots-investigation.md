# Project Write-Up: Splunk BOTS Incident Investigation

- **Role Focus:** SOC Analyst Tier 1 / Incident Investigation
- **Target Techniques (MITRE ATT&CK):**
  - `T1190` - Exploit Public-Facing Application
  - `T1059.003` - Windows Command Shell
  - `T1071.001` - Web Protocols (C2 Communication)
- **Dataset:** Splunk Boss of the SOC (BOTS) v1
- **Tools Used:** Splunk Enterprise, SPL (Search Processing Language), CyberChef

---

## 1. Executive Summary
Kasus ini menginvestigasi insiden peretasan sistem web milik perusahaan fiktif `Wayne Enterprises` (website: `imreallynotbatman.com`). Sebagai SOC Analyst Tier 1, tugas utama adalah merekonstruksi *Kill Chain* penyerang: dari fase pengintaian (reconnaissance), eksploitasi web aplikasi, injeksi webshell/backdoor, hingga koneksi keluar menuju server *Command & Control* (C2).

---

## 2. Investigasi Berbasis SPL (Search Processing Language)

### Langkah 1: Mengidentifikasi IP Penyerang & Web Scanning
Mencari IP luar yang melakukan request HTTP dalam volume tinggi dengan kode status anomali:

```spl
index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70"
| stats count by src_ip, uri_path, status
| sort -count
```
**Temuan:** IP `40.80.148.42` melakukan lebih dari 12.000 request dalam durasi 15 menit menggunakan scanner `Acunetix Web Vulnerability Scanner`.

### Langkah 2: Menemukan Vektor Eksploitasi & Webshell Upload
Menyelidiki request HTTP bertipe POST yang berhasil (status 200) selama periode serangan:

```spl
index=botsv1 sourcetype=stream:http src_ip="40.80.148.42" http_method=POST
| table _time, uri, uri_path, form_data
```
**Temuan:** Penyerang mengeksploitasi kerentanan pada ekstensi Joomla CMS untuk mengunggah file berbahaya dengan nama samaran pada path direktori `/joomla/images/stories/`.

### Langkah 3: Melacak Eksekusi Backdoor di Endpoint Windows
Memeriksa log Sysmon Event ID 1 (Process Creation) pada server web `192.168.250.70`:

```spl
index=botsv1 sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational EventCode=1
| search CommandLine="*3791.exe*" OR Image="*cmd.exe*"
| table _time, host, ParentCommandLine, CommandLine, User
```
**Temuan:** Proses web server `w3wp.exe` melahirkan child process `cmd.exe`, yang kemudian mengeksekusi biner tersembunyi `3791.exe` dari folder temporary.

### Langkah 4: Identifikasi Saluran C2 Keluar (Command & Control)
Mencari lalu lintas jaringan outbound dari server korban yang terinfeksi:

```spl
index=botsv1 sourcetype=stream:tcp src_ip="192.168.250.70"
| stats count by dest_ip, dest_port
| sort -count
```
**Temuan:** Server korban terhubung keluar secara konsisten ke IP `204.145.94.79` melalui port 4444 (khas reverse shell Metasploit).

---

## 3. Incident Timeline Table

| Waktu (UTC) | Entitas / IP | Aktivitas / Event |
|---|---|---|
| `01:12:00` | `40.80.148.42` | Pemindaian kerentanan web masif via Acunetix |
| `01:21:35` | `40.80.148.42` | POST request berhasil mengunggah file PHP webshell |
| `01:23:50` | `192.168.250.70` | Eksekusi biner backdoor `3791.exe` via `cmd.exe` |
| `01:24:15` | `204.145.94.79:4444` | Pembentukan sesi reverse shell C2 aktif |

---

## 4. Remediasi & Tindakan Isolasi
1. **Network Containment:** Segera memblokir IP `40.80.148.42` dan `204.145.94.79` pada firewall gateway perimeter.
2. **Endpoint Eradication:** Menghentikan proses `3791.exe`, menghapus webshell dari server web, dan mereset kredensial database & admin CMS.
3. **Patching:** Melakukan update versi Joomla dan menonaktifkan upload file tanpa validasi tipe MIME yang ketat.
