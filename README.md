# Validiasi Biometrik BPJS Otomatis (AutoValBiom) RSUD JAILOLO

Layanan API lokal untuk automasi login & pengisian data pada aplikasi verifikasi
biometrik BPJS Kesehatan (Sidik Jari & Face Recognition/Frista) yang terpasang di
komputer ini. Verifikasi biometrik tetap membutuhkan jari/wajah asli ditempelkan ke
alat seperti biasa — layanan ini hanya mempercepat pengisian username, password, dan
nomor kartu.

**Author:** IT RSUD JAILOLO

---

## Daftar Isi

1. [Instalasi](#instalasi)
2. [Konfigurasi (`config.json`)](#konfigurasi-configjson)
3. [Dokumentasi API](#dokumentasi-api)
4. [Autostart (Jalan Otomatis saat Komputer Login)](#autostart-jalan-otomatis-saat-komputer-login)
5. [Troubleshooting](#troubleshooting)

---

## Instalasi

1. Taruh `autovalbiom.exe` dan `config.json` dalam **satu folder yang sama**, contoh:
   `C:\AutoValBiom\`
2. Sesuaikan isi `config.json` (lihat bagian [Konfigurasi](#konfigurasi-configjson))
3. Jalankan `autovalbiom.exe` (double-click). Tidak akan muncul jendela apapun — layanan
   berjalan diam-diam di background.
4. Untuk memastikan layanan aktif, buka browser dan akses:
   `http://localhost:5000/health` (ganti `5000` kalau `port` di `config.json` diubah)
   → harus muncul `{"status": "ok"}`

---

## Konfigurasi (`config.json`)

File ini menentukan lokasi aplikasi yang diotomasi dan perilaku tampilan window-nya.
Bisa diedit langsung tanpa perlu install ulang — cukup simpan lalu jalankan ulang
`autovalbiom.exe`.

```json
{
    "host": "0.0.0.0",
    "port": 5000,

    "pathface": "C:\\frista.v.3.0.1\\frista.exe",
    "pathfinger": "C:\\Program Files (x86)\\BPJS Kesehatan\\Aplikasi Sidik Jari BPJS Kesehatan\\After.exe",
    "window_title_finger": "Aplikasi Registrasi Sidik Jari",
    "window_title_frista": "Login Frista (Face Recognition BPJS Kesehatan)",
    "window_title_frista_noka": "Frista (Face Recognition BPJS Kesehatan)",
    "start_wait_seconds": 8,
    "force_close_wait_seconds": 5,
    "noka_wait_seconds": 5,
    "frista_noka_wait_seconds": 5,
    "frista_login_button_offset_x": 403,
    "frista_login_button_offset_y": 399,

    "keep_on_top": true,
    "prevent_minimize": true,
    "window_position": "top-right",
    "window_margin_px": 10,
    "watchdog_interval_ms": 500
}
```

| Key | Fungsi |
|---|---|
| `host` | Alamat network tempat layanan menerima koneksi. `0.0.0.0` = bisa diakses dari komputer lain di jaringan; `127.0.0.1` = hanya dari komputer ini sendiri |
| `port` | Port layanan API. Ganti kalau `5000` bentrok dengan aplikasi lain di komputer ini |
| `pathface` | Lokasi file `.exe` aplikasi Frista di komputer ini |
| `pathfinger` | Lokasi file `.exe` aplikasi Sidik Jari BPJS di komputer ini |
| `start_wait_seconds` | Batas waktu maksimum menunggu aplikasi terbuka |
| `force_close_wait_seconds` | Batas waktu tunggu sebelum aplikasi yang macet ditutup paksa |
| `keep_on_top` | `true`/`false` — window aplikasi selalu tampil di depan |
| `prevent_minimize` | `true`/`false` — mencegah window ke-minimize/tertutup layar lain |
| `window_position` | Posisi window saat dibuka: `top`, `bottom`, `left`, `right`, `center`, `top-left`, `top-right`, `bottom-left`, `bottom-right`, atau `null` (posisi default) |
| `window_margin_px` | Jarak window dari tepi layar (untuk posisi pojok/tepi) |

*(key lain sebaiknya tidak diubah kecuali diminta oleh tim IT)*

---

## Dokumentasi API

Alamat layanan: `http://localhost:<port>` (`port` default `5000`, sesuai
`config.json`)

### Cek Status Layanan

```
GET /health
```
Response:
```json
{ "status": "ok" }
```

---

### Login — Sidik Jari

```
POST /autovalbiom/finger
```
Body:
```json
{
    "username": "...",
    "password": "...",
    "noka": "..."
}
```
- `username`, `password` — **wajib**
- `noka` (nomor kartu BPJS) — **opsional**, isi kalau ingin nomor kartu langsung
  terisi setelah login berhasil

Response berhasil:
```json
{ "status": "success", "message": "Login berhasil dan nomor kartu berhasil diisi" }
```

---

### Tutup Aplikasi — Sidik Jari

```
POST /autovalbiom/finger/close
```
Body (opsional):
```json
{ "force": true }
```
- Tanpa body → coba tutup normal, otomatis tutup paksa kalau aplikasi macet/tidak merespons
- `"force": true` → langsung tutup paksa

Response:
```json
{ "status": "success", "message": "Aplikasi berhasil ditutup secara normal" }
```

---

### Login — Face Recognition (Frista)

```
POST /autovalbiom/face
```
Body:
```json
{
    "username": "...",
    "password": "...",
    "noka": "..."
}
```
- `username`, `password` — **wajib**
- `noka` — **opsional**, diisi di layar berikutnya setelah login berhasil. Tombol
  pencarian nomor kartu tetap perlu diklik manual oleh petugas.

Response berhasil:
```json
{ "status": "success", "message": "Login berhasil dan nomor kartu berhasil diisi" }
```

---

### Tutup Aplikasi — Face Recognition (Frista)

```
POST /autovalbiom/face/close
```
Body & response sama seperti [Tutup Aplikasi — Sidik Jari](#tutup-aplikasi--sidik-jari).

---

### Respons Error (berlaku untuk semua endpoint di atas)

| Kode | Kondisi |
|---|---|
| `400` | `username`/`password` tidak dikirim |
| `429` | Ada proses login lain yang sedang berjalan — coba lagi beberapa detik kemudian |
| `500` | Gagal menjalankan automation (lihat `message` untuk detail) |

---

## Autostart (Jalan Otomatis saat Komputer Login)

Supaya layanan otomatis aktif tiap kali komputer dinyalakan/user login, tanpa perlu
dijalankan manual:

1. Klik kanan `autovalbiom.exe` → **Create shortcut**
2. Tekan `Win + R`, ketik `shell:startup`, tekan Enter
3. Pindahkan shortcut yang dibuat tadi ke folder yang terbuka
4. Restart komputer atau logout–login untuk memastikan layanan otomatis berjalan

Untuk memverifikasi layanan aktif setelah startup: akses
`http://localhost:5000/health` lewat browser, atau cek Task Manager untuk proses
`autovalbiom.exe`.

---

## Troubleshooting

- **Layanan tidak merespons** — pastikan `autovalbiom.exe` benar-benar berjalan (cek Task
  Manager). Kalau tidak ada di daftar proses, jalankan ulang secara manual.
- **Aplikasi target (Sidik Jari/Frista) tidak terbuka otomatis** — periksa kembali path
  di `config.json` (`pathfinger`/`pathface`), pastikan sesuai lokasi instalasi aplikasi
  di komputer ini.
- **Ada error saat login/proses gagal** — buka file `app.log` di folder yang sama
  dengan `autovalbiom.exe` untuk melihat detail error, lalu hubungi tim IT.
- **Layanan diblokir antivirus** — tambahkan folder `AutoValBiom` ke pengecualian
  (exclusion) antivirus/Windows Defender yang terpasang.

## Lisensi
[GNU](./LICENSE)

## Lainnya

- [Pemecahan Masalah](https://github.com/rizkifreao/autovalbiom/issues)
- [Laporkan Bug](https://github.com/rizkifreao/autovalbiom/issues/new)
