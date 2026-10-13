# Project Write-Up: Network Packet Forensics (Wireshark & Zeek)

- **Role Focus:** SOC Analyst Tier 1 / Network Security / Threat Hunting
- **Target Techniques (MITRE ATT&CK):**
  - `T1071.004` - Application Layer Protocol: DNS (DNS Tunneling)
  - `T1041` - Exfiltration Over C2 Channel
  - `T1071.001` - Web Protocols: HTTP Jitter Beaconing
- **Tools Used:** Wireshark, Tshark, Zeek (Bro), NetworkMiner

---

## 1. Executive Summary
Investigasi forensik paket jaringan dilakukan terhadap berkas `capture_incident_04.pcap` (2.4 GB) menyusul alert anomali beban lalu lintas outbound pada endpoint akuntansi internal `10.0.4.15`. Melalui analisis statistik percakapan protokol, pola frekuensi permintaan DNS, dan filter payload Wireshark, teridentifikasi adanya eksfiltrasi berkas rahasia menggunakan teknik **DNS Tunneling** serta komunikasi berkala (beaconing) dengan server penyerang.

---

## 2. Metodologi Analisis Wireshark

### Langkah 1: Tinjauan Statistik Percakapan (Protocol Hierarchy)
- **Menu:** `Statistics -> Protocol Hierarchy`
- **Temuan Awal:** Protokol DNS mencakup lebih dari 65% dari seluruh paket UDP, sebuah volume yang sangat tidak wajar untuk workstation kantor biasa.

### Langkah 2: Deteksi DNS Tunneling & Subdomain Entropy Tinggi
Menggunakan display filter untuk memfilter permintaan query DNS yang memiliki panjang karakter di atas normal:

```wireshark
dns.flags.response == 0 and dns.qry.name.len > 50
```

Contoh query yang tertangkap:
```
Mzg2NDAxOTJhYTJjOGI2ZGE5Mj.c3ZjLWFjY291bnRpbmctZGF0YQ.cdn-sync-update.biz
OWIxMmY0N2E4Y2QxYTIzOWZhZW.c3ZjLWFjY291bnRpbmctZGF0YQ.cdn-sync-update.biz
```
- **Karakteristik DNS Tunneling:**
  - Subdomain terdiri dari string acak heksadesimal / Base64 terenkripsi.
  - TTL (Time-To-Live) bernilai 0 atau 1 detik untuk memaksa DNS resolver tidak melakukan caching, sehingga setiap query dipaksa diteruskan ke Authoritative Name Server milik penyerang.

### Langkah 3: Identifikasi Pola HTTP Beaconing
Menganalisis traffic HTTP outbound menuju IP eksternal `198.51.100.22`:

```wireshark
http.request and ip.dst == 198.51.100.22
```

- Teramati request HTTP GET ke endpoint `/api/v1/ping` yang terjadi setiap 60 detik (&plusmn;10% jitter acak).
- Nilai User-Agent: `Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)` — pola default malware C2 legacy.

---

## 3. Ekstraksi Data yang Dieksfiltrasi
Melalui skrip `tshark`, seluruh query subdomain diekstrak, digabungkan, dan didekode dari format Base64:

```bash
tshark -r capture_incident_04.pcap -Y "dns.flags.response == 0 and dns.qry.name contains 'cdn-sync-update.biz'" -T fields -e dns.qry.name \
  | awk -F'.' '{print $1$2}' \
  | base64 -d > exfiltrated_data.bin
```

Pemeriksaan header berkas dengan perintah `file exfiltrated_data.bin` mengonfirmasi bahwa data yang dicuri adalah arsip database password `KDBX (KeePass Password Database)`.

---

## 4. Zeek / Bro Logs Correlation
Pada file `dns.log` yang dihasilkan oleh Zeek:
- Domain `cdn-sync-update.biz` mencatatkan lebih dari 8.500 query unik dalam waktu 30 menit.
- Modul deteksi Zeek memicu notice: `DNS::High_Volume_Unique_Subdomains`.

---

## 5. Tindakan Penanganan Insiden (Containment & Hardening)
1. **Network Containment:**
   - Memutus akses jaringan host `10.0.4.15` di tingkat port switch managed.
   - Sinkhole domain `cdn-sync-update.biz` pada server DNS internal perusahaan.
2. **Defensive Hardening:**
   - Menerapkan batasan firewall: workstation internal dilarang melakukan query DNS langsung ke internet (port 53 UDP/TCP) dan diwajibkan hanya melewati internal DNS server terdaftar.
   - Mengaktifkan fitur *DNS Inspection & Guard* pada firewall perimeter untuk membatasi panjang karakter query name maksimal 60 karakter.
