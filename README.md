# SIEM Home Lab: Linux Threat Detection & Monitoring with Splunk

## 📌 Deskripsi Proyek
Proyek ini adalah simulasi lingkungan *Security Information and Event Management* (SIEM) yang dibangun untuk memantau keamanan *host* Linux (Ubuntu). Matlamat utama dari lab ini adalah melakukan *ingestion* data log autentikasi, menyimulasikan serangan siber secara langsung, dan membangun kapabilitas deteksi (*Threat Hunting*) serta peringatan otomatis.

Selain dari perspektif operasional *Blue Team*, proyek ini juga didokumentasikan untuk memenuhi standar audit dan tata kelola keamanan (GRC), dengan fokus pada evaluasi kepatuhan kontrol akses pengguna.

## 🛠️ Teknologi & Lingkungan
* **Sistem Operasi:** Ubuntu Linux
* **SIEM:** Splunk Enterprise (Single Node)
* **Log Shipper:** Splunk Universal Forwarder
* **Sumber Data:** `/var/log/auth.log`, `/var/log/syslog`

## ⚔️ Skenario Serangan (*The Attack Narrative*)
Untuk menguji kapabilitas deteksi, simulasi serangan dilakukan dengan tahapan berikut:
1. **Initial Access:** Melakukan *SSH Brute Force* secara berulang untuk mencoba masuk ke sistem.
2. **Privilege Escalation:** Menyalahgunakan akses `sudo` untuk membuat *backdoor account* (`sysupdate`) guna mempertahankan akses (*Persistence*).
3. **Execution:** Menggunakan akun *backdoor* tersebut untuk mengunduh skrip dari internet (`curl`) dan mengeksekusinya di `/tmp`.

## 🔍 Threat Hunting & SPL Queries
Berikut adalah beberapa *query* Search Processing Language (SPL) utama yang digunakan untuk mengekstrak dan menganalisis log mentah.

**1. Mendeteksi SSH Brute Force (Ekstraksi dengan Regex)**
```splunk
index="linux_host" "Failed password" sshd
| rex field=_raw "Failed password for (invalid user )?(?<user>\S+) from (?<src_ip>\S+)"
| stats count by user, src_ip
| sort - count
```

**2. Mendeteksi Pembuatan User Baru (Audit Kepatuhan)**
```splunk
index="linux_host" ("useradd" OR "usermod" OR "new group" OR "new user") "sysupdate"
| table _time, _raw
```

**3. Mendeteksi Eksekusi Sudo yang Mencurigakan**
```splunk
index="linux_host" sudo "COMMAND=" ("curl" OR "chmod" OR "update.sh")
| rex field=_raw "USER=(?<run_as_user>\S+).*COMMAND=(?<sudo_command>.*)"
| table _time, user, run_as_user, sudo_command
```

## 📊 Visualisasi & Dashboard SOC

## 📄 Laporan Investigasi (Triage)
Hasil analisis log diubah menjadi laporan insiden yang berfokus pada mitigasi teknis dan penegakan kebijakan kontrol akses. Laporan lengkap dapat dilihat di sini:
Laporan Triage INC-2026-001
