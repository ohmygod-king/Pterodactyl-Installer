# Vogue Official — Pterodactyl Installer

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnubash&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04%20%7C%2022.04%20%7C%2024.04-E95420?style=flat&logo=ubuntu&logoColor=white)
![Debian](https://img.shields.io/badge/Debian-11%20%7C%2012-A81D33?style=flat&logo=debian&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-8.3-777BB4?style=flat&logo=php&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-Web%20Server-009639?style=flat&logo=nginx&logoColor=white)
![MariaDB](https://img.shields.io/badge/MariaDB-Database-003545?style=flat&logo=mariadb&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-Cache-DC382D?style=flat&logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Wings-2496ED?style=flat&logo=docker&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue.svg)

Script instalasi otomatis untuk **Pterodactyl Panel** dan **Wings (Daemon Node)**, lengkap dengan konfigurasi PHP 8.3, Nginx (anti 404 & 502), MariaDB, Redis, SSL otomatis via Let's Encrypt, serta opsi uninstall yang aman dengan backup otomatis.

---

## Daftar Isi

- [Fitur Utama](#fitur-utama)
- [Persyaratan Sistem](#persyaratan-sistem)
- [Cara Penggunaan](#cara-penggunaan)
- [Mode Instalasi](#mode-instalasi)
- [Data yang Diperlukan](#data-yang-diperlukan)
- [Struktur Hasil Instalasi](#struktur-hasil-instalasi)
- [Langkah Setelah Instalasi](#langkah-setelah-instalasi)
- [Uninstall](#uninstall)
- [Log & Troubleshooting](#log--troubleshooting)
- [Catatan Keamanan](#catatan-keamanan)
- [Lisensi](#lisensi)

---

## Fitur Utama

| Komponen | Deskripsi |
|---|---|
| Panel | Instalasi Pterodactyl Panel terbaru beserta dependensi (Composer, Laravel) |
| Wings | Instalasi Wings Daemon dengan deteksi arsitektur otomatis (amd64/arm64) |
| Web Server | Konfigurasi Nginx dengan aturan anti 404 & 502 untuk Laravel + PHP-FPM |
| Database | MariaDB dengan pembuatan database & user otomatis |
| Cache & Queue | Redis untuk cache, session, dan queue worker (pteroq) |
| SSL | Sertifikat Let's Encrypt otomatis untuk domain Panel maupun Wings |
| Theme | Opsi instalasi theme Reviactyl |
| Uninstall | Penghapusan aman dengan backup database & konfigurasi sebelum dihapus |

---

## Persyaratan Sistem

- Sistem operasi: Ubuntu 20.04 / 22.04 / 24.04 atau Debian 11 / 12
- Akses root (dijalankan via `sudo`)
- Domain yang sudah mengarah ke IP server (jika ingin menggunakan SSL)
- Koneksi internet aktif
- Port 80 dan 443 tidak digunakan oleh service lain saat proses SSL berjalan

---

## Clone Repositori

```bash
git clone https://github.com/ohmygod-king/Pterodactyl-Installer
cd Pterodactyl-Installer
bash svogtero.sh
```

## Tanpa Clone
```bash
bash <(curl -s https://raw.githubusercontent.com/ohmygod-king/Pterodactyl-Installer/main/svogtero.sh)
```


---

## Mode Instalasi

| Mode | Deskripsi |
|---|---|
| 1 | Install Panel saja |
| 2 | Install Wings saja |
| 3 | Install Panel + Wings dalam satu server |
| 4 | Install Theme (Reviactyl) pada Panel yang sudah ada |
| 5 | Uninstall Panel & Wings (dengan backup otomatis) |
| 6 | Keluar dari script |

---

## Data yang Diperlukan

### Konfigurasi Panel (Mode 1 / 3)

| Input | Keterangan |
|---|---|
| Domain Panel | Contoh: `panel.domain.com` |
| Email Admin | Digunakan untuk akun admin dan SSL |
| Username Admin | Default: `admin` |
| Nama Depan / Belakang | Identitas akun admin |
| Password Admin | Minimal 3 karakter |
| Timezone | Default: `Asia/Jakarta` |
| SSL Otomatis | Opsional, memerlukan domain yang sudah mengarah ke server |

### Konfigurasi Wings (Mode 2 / 3)

| Input | Keterangan |
|---|---|
| Domain Node | Contoh: `node.domain.com` |
| SSL Otomatis | Opsional, menggunakan Certbot standalone |

Password database (`DB_PASSWORD`) dibuat secara otomatis dan acak untuk setiap instalasi.

---

## Struktur Hasil Instalasi

| Path | Keterangan |
|---|---|
| `/var/www/pterodactyl` | Direktori source Panel |
| `/etc/pterodactyl` | Konfigurasi Wings (`config.yml`) |
| `/usr/local/bin/wings` | Binary Wings |
| `/etc/systemd/system/pteroq.service` | Queue worker Panel |
| `/etc/systemd/system/wings.service` | Service daemon Wings |
| `/etc/nginx/sites-available/pterodactyl.conf` | Virtual host Panel |
| `/root/vogue-pterodactyl-info.txt` | Ringkasan kredensial hasil instalasi |
| `/var/log/vogue-installer.log` | Log lengkap proses instalasi |

---

## Langkah Setelah Instalasi

1. Login ke Panel menggunakan akun admin yang dibuat saat instalasi.
2. Buat **Location** melalui menu Admin > Locations.
3. Buat **Node** baru melalui menu Admin > Nodes, isi FQDN sesuai domain Wings.
4. Tambahkan **Allocation** (IP dan range port) pada Node yang baru dibuat.
5. Salin command **Automatic Deploy** dari tab Configuration pada Node tersebut.
6. Jalankan command tersebut di server Wings untuk membuat `/etc/pterodactyl/config.yml`.
7. Aktifkan dan jalankan Wings:

```bash
systemctl start wings
```

---

## Uninstall

Pilih Mode 5 pada menu untuk menjalankan proses uninstall. Proses ini akan:

- Membuat backup `.env` Panel, `config.yml` Wings, dan dump database `panel` ke `/root/pterodactyl-backup-<timestamp>/`
- Menghentikan dan menonaktifkan service Wings serta Queue Worker
- Menghapus entri cron job Pterodactyl
- Menghapus database dan user MySQL terkait Panel
- Menghapus seluruh file Panel dan Wings beserta binary-nya
- Menghapus unit systemd terkait Pterodactyl
- Menghapus virtual host Nginx Panel tanpa mengganggu konfigurasi global Nginx
- Menawarkan opsi penghapusan sertifikat SSL milik domain Panel/Wings

Konfigurasi Nginx global, website lain, dan sertifikat SSL milik domain lain **tidak akan disentuh**.

---

## Log & Troubleshooting

Seluruh output command dicatat ke:

```
/var/log/vogue-installer.log
```

Jika sebuah langkah gagal, script akan berhenti dan menampilkan lokasi log untuk diperiksa lebih lanjut. Beberapa pengecekan tambahan yang berguna:

```bash
systemctl status wings
systemctl status pteroq
systemctl status nginx
nginx -t
```

---

## Catatan Keamanan

- Password admin dan database hanya ditampilkan sekali pada ringkasan akhir (`/root/vogue-pterodactyl-info.txt`), pastikan untuk menyimpannya di tempat yang aman.
- File ringkasan memiliki permission `600` (hanya dapat dibaca oleh root).
- Disarankan menggunakan SSL untuk komunikasi antara Panel dan Wings di lingkungan produksi.
- Selalu lakukan backup manual tambahan sebelum menjalankan proses uninstall pada server produksi.

---

## Lisensi

Script ini dirilis di bawah lisensi MIT. Bebas digunakan, dimodifikasi, dan didistribusikan dengan tetap menyertakan atribusi kepada pembuat aslinya.
