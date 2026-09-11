# Dokumentasi Modul: Auto Database Backup (Odoo 18)

**Nama Teknis:** `auto_database_backup`
**Versi Terbaru:** 19.0.2.0.2 (Rilis: 20 Juli 2026)
**Vendor / Author:** Clavisdev
**Lisensi:** LGPL-3
**Kompatibilitas:** Odoo Community & Enterprise
**Ketersediaan:** On Premise (tidak tersedia untuk Odoo.sh maupun Odoo Online)
**Dependensi Modul:** Discuss (mail)
**Jumlah Baris Kode:** ± 2.639


---

## 1. Ringkasan

Modul ini menyediakan fitur backup database Odoo secara otomatis maupun manual, dengan dukungan penjadwalan (scheduled action), riwayat backup, notifikasi email, serta dukungan multi-database dan multi-destinasi penyimpanan — mulai dari server lokal hingga berbagai layanan cloud storage.

---

## 2. Fitur Utama

| No | Fitur | Deskripsi |
|----|-------|-----------|
| 1 | **12 Destinasi Penyimpanan** | Local, FTP, SFTP, Google Drive, Dropbox, OneDrive, Nextcloud, Amazon S3, Azure, Google Cloud, WebDAV, dan penyedia S3-compatible lainnya |
| 2 | **Backup Terjadwal** | Backup otomatis harian, mingguan, atau bulanan melalui Odoo Scheduled Actions |
| 3 | **Backup Now** | Menjalankan backup secara instan dari form konfigurasi |
| 4 | **Download Backup** | Membuat backup baru dan langsung mengunduhnya ke browser |
| 5 | **Riwayat & Status Backup** | Setiap proses backup dicatat lengkap dengan status, durasi, dan error (jika ada) |
| 6 | **Pembersihan Backup Lama** | Menghapus backup otomatis setelah melewati jumlah hari tertentu (retention) |
| 7 | **Notifikasi Email** | Mengirim email saat backup berhasil, atau khusus saat backup gagal |
| 8 | **Multi-Database Backup** | Satu konfigurasi dapat mem-backup satu database, seluruh database, atau daftar database tertentu |
| 9 | **Keamanan** | Master password tidak pernah disimpan; hanya grup Backup Manager yang dapat mengakses konfigurasi & kredensial |

---

## 3. Destinasi Penyimpanan yang Didukung

| Destinasi | Keterangan |
|---|---|
| Local Storage | Disimpan di direktori server lokal |
| FTP | Upload ke server FTP |
| SFTP | Upload aman melalui SSH |
| Google Drive | Disimpan di folder Google Drive |
| Dropbox | Disimpan di folder Dropbox |
| OneDrive | Microsoft OneDrive (OAuth) |
| Nextcloud | Server Nextcloud self-hosted (via WebDAV) |
| Amazon S3 | S3 & layanan yang kompatibel dengan S3 |
| Azure Blob Storage | Microsoft Azure |
| Google Cloud Storage | Google Cloud |
| WebDAV | Server WebDAV apa pun |
| Backblaze B2 | Cloud storage murah (S3 Compatible) |
| Wasabi | Hot cloud storage (S3 Compatible) |
| Cloudflare R2 | Zero-egress storage (S3 Compatible) |
| MinIO | Server S3 self-hosted (S3 Compatible) |
| DigitalOcean Spaces | Object storage DigitalOcean (S3 Compatible) |
| iDrive e2 | Cloud object storage (S3 Compatible) |

---

## 4. Persiapan Sebelum Instalasi

Modul ini membutuhkan beberapa package Python eksternal. Install package sesuai destinasi yang akan digunakan **sebelum** menginstal modul.

**Package yang dibutuhkan:**
`dropbox`, `pyncclient`, `boto3`, `nextcloud-api-wrapper`, `paramiko`, `azure-storage-blob`, `google-cloud-storage`, `webdavclient3`

```bash
pip3 install dropbox
pip3 install pyncclient
pip3 install boto3
pip3 install nextcloud-api-wrapper
pip3 install paramiko
pip3 install azure-storage-blob
pip3 install google-cloud-storage
pip3 install webdavclient3
```

> Catatan: instal hanya package yang relevan dengan destinasi backup yang akan dipakai (misalnya jika hanya menggunakan Local Storage/FTP/SFTP, package cloud storage lain bisa dilewati — namun dokumentasi resmi menyarankan instalasi seluruh package di atas untuk mendukung semua fitur).

---

## 5. Alur Penggunaan

1. **Buat Konfigurasi Backup** — tentukan format backup dan destinasi penyimpanan, lalu uji koneksi (test connection).
2. **Atur Frekuensi & Opsi** — pilih jadwal (harian/mingguan/bulanan) dan aktifkan opsi penghapusan backup lama secara otomatis jika diperlukan.
3. **Pilih Database** — satu database, seluruh database di server, atau daftar database tertentu.
4. **Jalankan Backup** — via jadwal otomatis (cron/Scheduled Actions) atau tombol **Backup Now** untuk eksekusi instan.
5. **Pantau Riwayat** — cek log Backup History (status, durasi, error) dan status terakhir pada tiap konfigurasi.
6. **Notifikasi** — aktifkan email notifikasi agar mendapat pemberitahuan setiap backup berhasil/gagal.

---

## 6. Frequently Asked Questions (FAQ)

**Destinasi apa saja yang didukung?**
Dua belas destinasi: Local, FTP, SFTP, Google Drive, Dropbox, OneDrive, Nextcloud, Amazon S3 (termasuk penyedia S3-compatible seperti Backblaze B2, Wasabi, Cloudflare R2, MinIO), Azure Blob Storage, Google Cloud Storage, dan WebDAV.

**Seberapa sering backup dijalankan?**
Sesuai jadwal yang dipilih pengguna — harian, mingguan, atau bulanan — menggunakan Odoo Scheduled Actions. Backup juga bisa dijalankan kapan saja melalui tombol **Backup Now**.

**Apakah backup lama dihapus otomatis?**
Ya. Aktifkan opsi "Remove Old Backups" dan tentukan jumlah hari retensi; backup yang lebih tua dari itu akan otomatis dihapus dari destinasi penyimpanan.

**Apakah master password database disimpan?**
Tidak. Master password hanya digunakan untuk otorisasi saat konfigurasi disimpan dan tidak pernah disimpan pada record. Backup terjadwal tidak memerlukan input password tersebut.

**Bisakah backup lebih dari satu database sekaligus?**
Ya. Satu konfigurasi dapat mem-backup satu database, seluruh database di server, atau daftar database tertentu — setiap database mendapat entri riwayat (history) sendiri.

**Bagaimana cara mengetahui apakah backup berhasil?**
Melalui log Backup History (satu baris per proses, dengan status, durasi, dan error jika ada) serta status terakhir pada setiap konfigurasi. Notifikasi email juga bisa diaktifkan.

---

## 7. Riwayat Rilis

| Versi | Tanggal | Perubahan |
|---|---|---|
| **19.0.2.0.2** | 20 Juli 2026 | Refactor untuk Odoo 19. Penambahan destinasi Azure Blob, Google Cloud Storage & WebDAV, endpoint S3-compatible, fitur Backup Now, riwayat backup, status last-run, notifikasi saat gagal, download backup, dan multi-database backup |
| **19.0.1.0.0** | Rilis Awal | Backup database otomatis ke destinasi lokal, remote, dan cloud |

---

## 8. Dukungan & Kontak

- **Email:** project@clavis.co.id
- **WhatsApp / Telepon:** +91 9074270811
- **Instant Support:** https://support.clavis.co.id/instant-support
- **Video Tutorial:** https://youtu.be/Q2yMZyYjuTI

---

## 9. Catatan Tambahan

- Modul ini **tidak tersedia** untuk Odoo Online maupun Odoo.sh — hanya untuk instalasi **On Premise**.
- Akses konfigurasi dan kredensial backup dibatasi hanya untuk grup **Backup Manager** demi keamanan.
- Modul memiliki versi terpisah untuk versi Odoo 12.0 s/d 19.0.

---

*Dokumen ini disusun berdasarkan informasi yang dipublikasikan di halaman Odoo Apps Store untuk modul `auto_database_backup` versi 19.0, per tanggal akses September 2026.*
