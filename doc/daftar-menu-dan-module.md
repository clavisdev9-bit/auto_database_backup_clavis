# Daftar Menu & Modul — Auto Database Backup Clavis (Odoo 18)

**Nama Teknis:** `auto_database_backup_clavis`
**Versi:** 18.0.2.0.1
**Author / Maintainer:** Clavis Development
**Kategori:** Extra Tools
**Lisensi:** LGPL-3
**Dependensi Modul Odoo:** `base`, `mail`
**Tipe:** Modul teknis (`application: False`) — muncul di menu Settings, bukan sebagai app tersendiri
**Terakhir diverifikasi terhadap source code:** 12 September 2026

---

## 1. Struktur Menu (yang terlihat user)

Modul ini **tidak memiliki app icon sendiri**. Seluruh menunya berada di bawah menu Technical.

```
Settings
└── Technical                                  [butuh Developer Mode aktif]
    └── Automatic Database Backup              (menu root, sequence 10)
        └── Backup Configuration               → daftar konfigurasi backup
```

| # | Menu | ID Teknis | Parent | Action | Model | Tampilan |
|---|------|-----------|--------|--------|-------|----------|
| 1 | Automatic Database Backup | `db_backup_menu_root` | `base.menu_custom` (Settings → Technical) | — (menu container) | — | — |
| 2 | Backup Configuration | `db_backup_configure_menu` | Automatic Database Backup | `db_backup_configure_action` | `db.backup.configure` | List, Form |

> **Catatan untuk PM:** menu `Settings → Technical` hanya tampil bila user mengaktifkan **Developer Mode** (grup `base.group_no_one`). Untuk user operasional/non-teknis, menu ini tidak akan terlihat kecuali dibuatkan menu alternatif di luar Technical.

### Layar yang tersedia

| Layar | Jenis View | Isi Utama |
|---|---|---|
| Backup Configuration — List | `list` | Name, Database Name, Backup Destination, Backup Frequency, Active (baris non-aktif ditampilkan abu-abu) |
| Backup Configuration — Form | `form` | Konfigurasi database, format, destinasi + kredensial dinamis per destinasi, jadwal, retensi, notifikasi, tombol Test Connection & Setup Token |
| Backup Configuration — Search | `search` | Cari by Name / Database Name; filter All & Archived; Group By Backup Type |
| Authorization Code (Dropbox) | `form` (wizard popup) | Link ambil authorization code + input code + tombol Confirm |

---

## 2. Daftar Komponen / Modul Teknis

### 2.1 Model (Objek Data)

| # | Model | Tipe | Deskripsi | File |
|---|---|---|---|---|
| 1 | `db.backup.configure` | Model persisten | Master konfigurasi backup — menyimpan kredensial, jadwal, destinasi, retensi, dan menjalankan proses backup | `models/db_backup_configure.py` |
| 2 | `dropbox.auth.code` | Transient (wizard) | Wizard untuk memasukkan Dropbox authorization code dan menghasilkan refresh token | `wizard/dropbox_auth_code.py` |

### 2.2 Sub-modul Fungsional pada `db.backup.configure`

| # | Sub-modul | Fungsi | Method Terkait |
|---|---|---|---|
| 1 | Konfigurasi Database | Nama DB, master password, format backup (Zip / Dump), status aktif | `_check_db_credentials` |
| 2 | Penjadwalan Backup | Frekuensi Daily / Weekly / Monthly, dieksekusi oleh Scheduled Action | `_schedule_auto_backup(frequency)` |
| 3 | Proses Dump Database | Membuat file dump/zip dari database | `dump_data`, `_dump_db_manifest` |
| 4 | Retensi Backup | Hapus backup lama otomatis setelah N hari (`auto_remove`, `days_to_remove`) | bagian dari `_schedule_auto_backup` |
| 5 | Test Connection | Uji koneksi ke destinasi sebelum backup dijalankan | `action_sftp_connection` (FTP/SFTP), `action_nextcloud`, `action_s3cloud` |
| 6 | Otorisasi OAuth | Setup / reset token untuk layanan cloud | `action_get_dropbox_auth_code`, `action_get_gdrive_auth_code`, `action_get_onedrive_auth_code`, `generate_*_refresh_token`, `get_*_tokens` |
| 7 | Notifikasi Email | Kirim email sukses/gagal ke user yang ditunjuk (`notify_user`, `user_id`) | menggunakan mail template |

### 2.3 Destinasi Penyimpanan yang Didukung (8 destinasi)

| # | Destinasi | Nilai Teknis | Kredensial yang Diminta |
|---|---|---|---|
| 1 | Local Storage | `local` | Backup Path |
| 2 | Google Drive | `google_drive` | Client ID, Client Secret, Redirect URI, Drive Folder ID + OAuth token |
| 3 | FTP | `ftp` | Host, Port (default 21), User, Password, Path |
| 4 | SFTP | `sftp` | Host, Port (default 22), User, Password, Path |
| 5 | Dropbox | `dropbox` | App Key, App Secret, Dropbox Folder + refresh token via wizard |
| 6 | OneDrive | `onedrive` | Client ID, Client Secret, Redirect URI, Folder ID + OAuth token |
| 7 | Next Cloud | `next_cloud` | Domain Name, User Name, Password, Folder ID |
| 8 | Amazon S3 | `amazon_s3` | Access Key, Secret Key, Bucket Name, File/Folder Name |

### 2.4 Scheduled Action (Cron)

| # | Nama Cron | ID Teknis | Interval | Kode |
|---|---|---|---|---|
| 1 | Backup : Daily Database Backup | `ir_cron_auto_db_backup_daily` | 1 hari | `model._schedule_auto_backup('daily')` |
| 2 | Backup : Weekly Database Backup | `ir_cron_auto_db_backup_weekly` | 1 minggu | `model._schedule_auto_backup('weekly')` |
| 3 | Backup : Monthly Database Backup | `ir_cron_auto_db_backup_monthly` | 1 bulan | `model._schedule_auto_backup('monthly')` |

### 2.5 Mail Template

| # | Template | ID Teknis | Subject |
|---|---|---|---|
| 1 | Database Backup: Notification Successful | `mail_template_data_db_backup_successful` | Database Backup Successful: {db_name} |
| 2 | Database Backup: Notification Failed | `mail_template_data_db_backup_failed` | Database Backup Failed: {db_name} |

### 2.6 Controller / Endpoint Web

| # | Route | Auth | Fungsi |
|---|---|---|---|
| 1 | `/onedrive/authentication` | public | Callback OAuth OneDrive — menyimpan token lalu mengaktifkan konfigurasi |
| 2 | `/google_drive/authentication` | public | Callback OAuth Google Drive — menyimpan token lalu mengaktifkan konfigurasi |

### 2.7 Hak Akses (Security)

| # | Model | Grup | Read | Write | Create | Delete |
|---|---|---|---|---|---|---|
| 1 | `db.backup.configure` | `base.group_user` (Internal User) | ✔ | ✔ | ✔ | ✔ |
| 2 | `dropbox.auth.code` | `base.group_user` (Internal User) | ✔ | ✔ | ✔ | ✔ |

> **Catatan risiko untuk PM:** saat ini **semua Internal User** dapat membaca dan mengubah konfigurasi backup, termasuk kredensial cloud (API key, secret, password FTP/SFTP). Tidak ada grup khusus "Backup Manager". Menu memang tersembunyi di Technical, tetapi pembatasan itu hanya di level tampilan menu — bukan di level data. Disarankan menambah security group tersendiri bila modul dipakai di lingkungan produksi multi-user.

### 2.8 Dependensi Library Python (harus terpasang di server)

| # | Library | Dipakai untuk |
|---|---|---|
| 1 | `dropbox` | Upload ke Dropbox |
| 2 | `paramiko` | Koneksi SFTP |
| 3 | `boto3` | Upload ke Amazon S3 |
| 4 | `pyncclient` | Koneksi Nextcloud |
| 5 | `nextcloud-api-wrapper` | Koneksi Nextcloud |

---

## 3. Ringkasan Jumlah Komponen

| Komponen | Jumlah |
|---|---|
| Menu item | 2 (1 root + 1 menu aksi) |
| Window action | 1 |
| Model persisten | 1 |
| Wizard (transient model) | 1 |
| View | 4 (list, form, search, form wizard) |
| Scheduled action | 3 |
| Mail template | 2 |
| Controller route | 2 |
| Baris ACL | 2 |
| Destinasi backup | 8 |

---

## 4. Selisih dengan `doc/dokumentasi-auto-database-backup.md`

Dokumen `dokumentasi-auto-database-backup.md` menggambarkan versi produk yang **lebih besar dari kode yang ada di repository ini**. Berikut selisih yang perlu diluruskan sebelum dokumen dipakai ke klien:

| Klaim di dokumentasi lama | Kondisi aktual di kode |
|---|---|
| Versi 19.0.2.0.2 | Manifest: **18.0.2.0.1** |
| Nama teknis `auto_database_backup` | Folder & XML ID: **`auto_database_backup_clavis`** |
| 12+ destinasi (Azure, Google Cloud, WebDAV, Backblaze, Wasabi, R2, MinIO, DigitalOcean, iDrive) | Hanya **8 destinasi** (lihat tabel 2.3) |
| Fitur **Backup Now** (backup instan dari form) | **Belum ada** — tidak ada tombol/method-nya |
| Fitur **Download Backup** | **Belum ada** |
| **Riwayat & status backup** (log per proses) | **Belum ada model riwayat**; hanya field `generated_exception` dan `backup_filename` pada konfigurasi |
| **Multi-database backup** (all / list DB) | **Belum ada** — satu konfigurasi = satu `db_name` |
| Hanya grup **Backup Manager** yang bisa akses | Akses penuh diberikan ke **semua Internal User** |
| ± 2.639 baris kode | ± 1.100 baris pada `models/db_backup_configure.py` |

**Rekomendasi:** tentukan apakah dokumentasi lama akan dijadikan *backlog roadmap* (fitur yang belum dibangun) atau direvisi agar sesuai kondisi saat ini. Keduanya butuh keputusan PM, karena dokumen tersebut saat ini dapat terbaca sebagai janji fitur ke klien.
