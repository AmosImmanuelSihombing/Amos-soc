# Sistem Deteksi dan Pencegahan Serangan DDoS Berbasis Threshold pada Web

- **Penulis:** Amos Immanuel Sihombing (NIM: 2022071020)
- **Program Studi:** Informatika, Fakultas Teknologi dan Desain, Universitas Pembangunan Jaya (2026)
- **Dosen Pembimbing:** Prio Handoko, S.Kom., M.T.I.
- **Topik:** Keamanan Jaringan / Web Security / SOC Operations
- **Pemetaan MITRE ATT&CK:** `T1498` (Network Denial of Service), `T1499` (Endpoint Denial of Service)
- **Infrastruktur Uji:** Virtual Private Server (VPS) Ubuntu Server, Nginx Web Server, Domain: `ddosta.my.id`

---

## 1. Ringkasan Eksekutif
Serangan *Distributed Denial of Service* (DDoS) merupakan ancaman kritis terhadap ketersediaan layanan web (*availability*). Penelitian Tugas Akhir ini merancang dan mengimplementasikan sistem deteksi dan mitigasi otomatis berbasis metode *threshold*. Sistem memantau berkas *access log* web server Nginx secara real-time, mengidentifikasi lonjakan permintaan (*request surges*), dan secara otomatis memblokir alamat IP penyerang menggunakan firewall `iptables`, dilengkapi mekanisme pelepasan blokir (*auto-release*) dan *dashboard monitoring* real-time.

---

## 2. Arsitektur & Alur Kerja Sistem

```
[ Pengguna / Client ] ---> [ Web Server Nginx (Ubuntu VPS) ]
                                      |
                               (Mencatat Request)
                                      v
                             [ Nginx access.log ]
                                      |
                            (Dibaca Real-Time seek(0,2))
                                      v
                     [ Program Deteksi: monitor.py (Python) ]
                                      |
                  (Hitung Request dlm Interval 10 Detik)
                                      |
                       +--------------+--------------+
                       |                             |
             [ Request <= Threshold ]      [ Request > Threshold ]
                       |                             |
               (Update Traffic Log)          (Catat ke attack.log)
                       |                             |
                       |                             v
                       |               [ Eksekusi iptables -j DROP ]
                       |                             |
                       |                     (blocked.log dibuat)
                       |                             |
                       |                             v
                       |                 [ Timer Pelepasan: 300s ]
                       |                             |
                       |                     (iptables -D dieksekusi)
                       |                             |
                       +--------------+--------------+
                                      |
                                      v
                      [ Dashboard Monitoring Web (PHP) ]
                 (Status Sistem, Total Serangan, Log Real-Time)
```

---

## 3. Komponen Teknis Utama

### A. Pembacaan Log Real-Time (`monitor.py`)
Menggunakan mekanisme `seek(0, 2)` untuk langsung berpindah ke baris paling akhir file `access.log` Nginx, sehingga hanya request baru yang diproses secara efisien tanpa pembacaan ulang berkas yang membebani memori.

### B. Algoritma Deteksi Berbasis Threshold & Sliding Window
- **Interval Waktu (Window):** 10 detik.
- **Batas Ambang (Threshold):** 1000 request (dapat disesuaikan).
- Menggunakan struktur data `defaultdict(deque)` untuk melacak timestamp setiap request per alamat IP sumber.

### C. Mekanisme Pencegahan Otomatis (Firewall `iptables`)
Ketika suatu IP melampaui batas ambang, program mengeksekusi sub-proses firewall:
```bash
iptables -A INPUT -s <IP_PENYERANG> -j DROP
```
Aktivitas pemblokiran dicatat secara terstruktur ke `blocked.log`.

### D. Mekanisme Pelepasan Otomatis (*Auto-Release Timer*)
Mencegah pemblokiran permanen terhadap pengguna yang sah (*False Positive mitigation*). Menggunakan `threading.Timer(300, unblock_ip, args=[ip])` untuk menghapus aturan blokir setelah 300 detik:
```bash
iptables -D INPUT -s <IP_PENYERANG> -j DROP
```
Pelepasan dicatat ke dalam `unblocked.log`.

### E. Dashboard Monitoring Real-Time (PHP)
Menyajikan antarmuka visual terpusat yang me-refresh data secara otomatis setiap 1 detik (`meta http-equiv="refresh"`), menampilkan:
- Status Sistem (`NORMAL` vs `ATTACK DETECTED`)
- Total Serangan Terdeteksi & Total IP Terblokir
- Total IP yang Telah Dilepaskan
- Log 20 Peristiwa Trafik Terkini

---

## 4. Hasil Pengujian & Kesimpulan
1. **Validasi Deteksi:** Sistem terbukti 100% mampu mengidentifikasi lonjakan request saat batas threshold terlampaui.
2. **Ketersediaan Layanan:** Saat IP penyerang diblokir oleh `iptables`, pengguna normal lainnya tetap dapat mengakses website tanpa mengalami penurunan performa atau server down.
3. **Relevansi SOC:** Proyek ini membuktikan pemahaman menyeluruh atas siklus penanganan insiden (*Detection & Analysis -> Containment -> Recovery -> Monitoring*).
