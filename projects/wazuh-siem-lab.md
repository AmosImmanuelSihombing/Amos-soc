# Project Write-Up: Wazuh SIEM Homelab & Detection Engineering

- **Role Focus:** SOC Analyst Tier 1 / Blue Team / Detection Engineering
- **Target Techniques (MITRE ATT&CK):**
  - `T1110.001` - Password Guessing / Brute Force
  - `T1003.001` - OS Credential Dumping: LSASS Memory (Mimikatz)
  - `T1059.001` - PowerShell Execution
- **Environment:**
  - **SIEM Core:** Wazuh Server 4.7 (Manager, Indexer, Dashboard) on Ubuntu 22.04 LTS
  - **Endpoints:** Windows 11 Enterprise (Sysmon v14 + Wazuh Agent) & Debian 12
  - **Attacker VM:** Kali Linux 2024.1 (Hydra, Metasploit, Impacket)

---

## 1. Executive Summary
Proyek laboratorium ini bertujuan untuk membangun visibilitas telemetri keamanan menyeluruh pada jaringan enterprise lokal. Dengan mengintegrasikan **Wazuh SIEM** dan **Microsoft Sysmon**, seluruh aktivitas proses, koneksi jaringan, dan autentikasi dipantau secara real-time. Serangan brute-force SSH dan ekstraksi kredensial memori menggunakan Mimikatz disimulasikan untuk menguji efektifitas deteksi dan respon aktif otomatis (*Active Response*).

---

## 2. Lab Architecture & Data Flow

```
[ Attacker: Kali Linux (192.168.10.45) ]
                |
                v (Attacks: Hydra Brute-Force & Mimikatz)
[ Target: Windows 11 / Debian (192.168.10.20) ]
       | (Sysmon Event Logs + Syslog)
       v
[ Wazuh Agent (Telemetry Forwarder) ]
       | (Encrypted Port 1514)
       v
[ Wazuh Manager & Indexer (192.168.10.10) ]
       | (Alert Triggers & Active Response)
       v
[ Automated Containment: Firewall Drop Rule + Dashboard Alert ]
```

---

## 3. Threat Simulation & Telemetry Analysis

### Skenario A: SSH Brute-Force Attack
- **Simulasi:** Menggunakan `hydra -l root -P rockyou.txt ssh://192.168.10.20`
- **Analisis Telemetri:**
  - Menghasilkan puluhan event kegagalan autentikasi dalam waktu kurang dari 30 detik pada `/var/log/auth.log`.
  - Wazuh Rule `5710` (Attempt to login using a non-existent user) dan Rule `5712` (SSHD brute force trying to get access to the system) terpicu pada level keparahan 10.

### Skenario B: Mimikatz LSASS Credential Dumping
- **Simulasi:** Eksekusi `sekurlsa::logonpasswords` pada endpoint Windows 11.
- **Analisis Telemetri Sysmon:**
  - Sysmon Event ID `10` (ProcessAccess) menangkap akses handle mencurigakan ke proses `lsass.exe` dari proses yang tidak dikenal dengan hak akses `0x1010` atau `0x1fffff`.

---

## 4. Custom Detection Engineering (Wazuh XML Rule)

Dibuat aturan kustom pada `/var/ossec/etc/rules/local_rules.xml` untuk mendeteksi upaya dumping memori LSASS secara spesifik:

```xml
<group name="sysmon,credential_dumping,">
  <rule id="100550" level="13">
    <if_sid>61603</if_sid>
    <field name="win.eventdata.targetImage">C:\\Windows\\System32\\lsass.exe</field>
    <field name="win.eventdata.grantedAccess">0x1010|0x1038|0x1fffff</field>
    <description>Possible LSASS Memory Dump (Mimikatz Activity Detected)</description>
    <mitre>
      <id>T1003.001</id>
    </mitre>
  </rule>
</group>
```

---

## 5. Automated Active Response (Containment)

Konfigurasi automated active response pada `ossec.conf`:

```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5712, 100550</rules_id>
  <timeout>3600</timeout>
</active-response>
```

**Hasil:** Segera setelah 8 percobaan login gagal terdeteksi berturut-turut, script `firewall-drop` mengeksekusi perintah iptables untuk menjatuhkan seluruh paket dari IP attacker (192.168.10.45) selama 1 jam.

---

## 6. Key Learnings & SOC Analyst Takeaways
1. **Pentingnya Sysmon Tuning:** Log Windows default tidak cukup mendalam untuk menangkap memory injection; tuning konfigurasi Sysmon (filtering noise) sangat krusial untuk mencegah alert fatigue.
2. **Korelasi Multi-Event:** Mengandalkan single event rawan memicu false positive. Rule deteksi berbasis threshold frekuensi (misal: 10 kegagalan dalam 60 detik) jauh lebih handal.
3. **Automated vs Manual Action:** Tindakan isolasi otomatis sangat efektif untuk brute force, namun untuk akses LSASS pada server kritis, eskalasi ke Tier 2 dan investigasi manual tetap diperlukan sebelum mengisolasi layanan inti bisnis.
