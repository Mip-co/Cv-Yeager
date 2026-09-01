# 💰 DuTrack

> Aplikasi pembukuan keuangan beasiswa untuk mencatat transaksi, memantau penggunaan dana, scan struk, dan membuat laporan LPJ otomatis berdasarkan semester akademik.

[![Live Demo](https://img.shields.io/badge/🌐_Live_Demo-dutrack.vercel.app-7C6AF5?style=for-the-badge)](https://dutrack.vercel.app)
[![Vercel](https://img.shields.io/badge/Deployed_on-Vercel-000000?style=for-the-badge\&logo=vercel)](https://vercel.com)
[![Supabase](https://img.shields.io/badge/Backend-Supabase-3ECF8E?style=for-the-badge\&logo=supabase)](https://supabase.com)
[![Gemini](https://img.shields.io/badge/AI-Gemini-4285F4?style=for-the-badge\&logo=google)](https://aistudio.google.com)

---

## ✨ Fitur

| Fitur                       | Deskripsi                                                                            |
| --------------------------- | ------------------------------------------------------------------------------------ |
| 📊 **Dashboard Bento**      | Monitoring dana beasiswa, saldo, pemasukan, pengeluaran, tren keuangan, dan kategori |
| 💸 **Manajemen Transaksi**  | Tambah, edit, hapus, pencarian, filter kategori/tipe, dan pagination                 |
| 📅 **Filter Semester**      | Memisahkan transaksi berdasarkan rentang semester akademik                           |
| 📈 **Analitik Semester**    | Grafik dan ringkasan keuangan otomatis mengikuti semester yang dipilih               |
| 🤖 **Scan Struk AI**        | Ekstraksi nominal, tanggal, keterangan, dan kategori dari foto struk                 |
| 📷 **OCR Tesseract.js**     | OCR alternatif untuk membaca struk langsung di browser                               |
| ☁️ **Cloud Sync**           | Sinkronisasi transaksi antar perangkat melalui Supabase                              |
| 📋 **Export LPJ Beasiswa**  | Generate file XLSX LPJ berdasarkan semester aktif                                    |
| 📄 **Export PDF**           | Generate laporan keuangan lengkap dalam format PDF                                   |
| 🌙 **Dark / Light Mode**    | Pilihan tema gelap dan terang                                                        |
| 📱 **Responsive Mobile UI** | Layout, navigasi, dashboard, dan kontrol dioptimalkan untuk smartphone               |
| 🔒 **Mode Lokal**           | Gunakan aplikasi tanpa akun dengan penyimpanan `localStorage`                        |
| 📲 **PWA Support**          | Manifest dan icon agar aplikasi lebih nyaman digunakan di perangkat mobile           |

---

## 📅 Sistem Semester

DuTrack menggunakan **rentang bulan semester** untuk memisahkan data transaksi.

Setiap semester memiliki:

```text
Nomor Semester
Label Semester
Bulan Mulai
Bulan Selesai
```

Contoh:

```text
Semester 4
Maret 2026 — Agustus 2026

Semester 5
September 2026 — Februari 2027
```

Saat semester dipilih dari dropdown, DuTrack hanya menggunakan transaksi yang tanggalnya berada di dalam rentang semester tersebut.

Filter ini digunakan pada:

* Dashboard
* Daftar transaksi
* Scholarship Monitoring
* Analitik
* Grafik pemasukan & pengeluaran
* Ringkasan bulanan
* Export LPJ

Semester terbaru otomatis digunakan sebagai semester aktif apabila belum ada semester lain yang dipilih.

---

## 🛠️ Tech Stack

```text
Frontend   → HTML, CSS, Vanilla JavaScript
Charts     → Chart.js
AI / OCR   → Gemini Vision
OCR        → Tesseract.js
Auth & DB  → Supabase
Database   → PostgreSQL
Security   → Supabase Row Level Security
Storage    → Supabase Storage
Export     → SheetJS (xlsx-js-style), jsPDF, html2canvas
Hosting    → Vercel
PWA        → Web App Manifest
```

---

# 🚀 Setup Supabase

> Setiap pengguna dapat menggunakan project Supabase sendiri.

## 1. Buat Project Supabase

1. Buka [supabase.com](https://supabase.com)
2. Login atau buat akun
3. Klik **New Project**
4. Tentukan nama project dan password database
5. Pilih region terdekat, misalnya **Singapore**
6. Tunggu proses provisioning selesai

---

## 2. Buat Tabel Database

Buka:

```text
SQL Editor → New Query
```

Kemudian jalankan SQL berikut.

### Transactions

```sql
create table transactions (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users(id),
  type text not null check (type in ('income','expense')),
  amount numeric not null,
  description text,
  category text,
  date date not null,
  receipt_url text,
  created_at timestamptz default now()
);

alter table transactions enable row level security;

create policy "Users can manage own transactions"
  on transactions
  for all
  using (auth.uid() = user_id);
```

### Semesters

```sql
create table semesters (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users(id),
  number integer not null,
  label text not null,
  start_month text not null,
  end_month text not null,
  created_at timestamptz default now()
);

alter table semesters enable row level security;

create policy "Users can manage own semesters"
  on semesters
  for all
  using (auth.uid() = user_id);
```

Jika berhasil, tabel berikut akan muncul:

```text
Database
├── transactions
└── semesters
```

---

## 3. Buat Storage Bucket untuk Struk

Buka:

```text
Storage → New Bucket
```

Gunakan konfigurasi:

```text
Bucket name   : receipts
Public bucket : ON
```

Tambahkan policy berikut.

### Upload

```sql
create policy "Users can upload receipts"
  on storage.objects
  for insert
  to authenticated
  with check (
    bucket_id = 'receipts'
    and auth.uid()::text = (storage.foldername(name))[1]
  );
```

### Read

```sql
create policy "Public can view receipts"
  on storage.objects
  for select
  to public
  using (bucket_id = 'receipts');
```

---

## 4. Ambil Supabase URL & Anon Key

| Field           | Lokasi                              |
| --------------- | ----------------------------------- |
| **Project URL** | Settings → Integrations → Data API  |
| **Anon Key**    | Settings → API Keys → `anon public` |

> ⚠️ Jangan menggunakan `service_role` key di frontend.

---

## 5. Hubungkan Supabase ke DuTrack

1. Buka [dutrack.vercel.app](https://dutrack.vercel.app)
2. Masuk ke **Pengaturan**
3. Buka **Konfigurasi Supabase**
4. Isi **Supabase URL**
5. Isi **Anon Key**
6. Klik **Simpan & Hubungkan**

Jika berhasil:

```text
✅ Koneksi berhasil!
```

---

## 6. Konfigurasi Authentication

Untuk mempermudah penggunaan personal, konfirmasi email dapat dinonaktifkan:

```text
Authentication
→ Sign In / Providers
→ Email
→ Confirm Email
→ OFF
```

Set juga Site URL:

```text
Authentication
→ URL Configuration
→ Site URL
→ https://dutrack.vercel.app
```

---

## 7. Register & Login

1. Buka DuTrack
2. Klik **Daftar**
3. Masukkan email dan password
4. Login
5. Data otomatis disinkronkan dengan Supabase ☁️

---

# 🤖 Setup Gemini AI

Gemini dapat digunakan untuk meningkatkan proses pembacaan struk sehingga aplikasi dapat memahami data dari gambar secara lebih fleksibel.

## 1. Buat API Key

Buka:

[Google AI Studio](https://aistudio.google.com/apikey)

Kemudian:

```text
Create API Key
→ Copy API Key
```

---

## 2. Hubungkan ke DuTrack

Buka:

```text
Pengaturan
→ Konfigurasi Gemini AI
```

Masukkan API Key kemudian simpan.

API Key disimpan pada browser pengguna melalui `localStorage`.

---

## 🤖 Gemini vs OCR Lokal

| Kemampuan               |          Gemini | Tesseract.js |
| ----------------------- | --------------: | -----------: |
| Membaca nominal         |               ✅ |            ✅ |
| Membaca tanggal         |               ✅ |            ✅ |
| Memahami nama toko      |               ✅ |           ⚠️ |
| Menentukan kategori     |               ✅ |            ❌ |
| Foto kurang rapi        | ✅ Lebih toleran |           ⚠️ |
| Berjalan tanpa internet |               ❌ |            ✅ |
| Membutuhkan API Key     |               ✅ |            ❌ |

---

# 📅 Setup Semester

## 1. Tambah Semester

Masuk ke:

```text
Pengaturan
→ Manajemen Semester
```

Isi:

```text
Nomor Semester
Label
Bulan Mulai
Bulan Selesai
```

Contoh:

```text
Semester 4
Mar 2026 → Agu 2026
```

```text
Semester 5
Sep 2026 → Feb 2027
```

Kemudian klik **Simpan**.

---

## 2. Pilih Semester Aktif

Gunakan dropdown semester pada bagian atas aplikasi.

Misalnya:

```text
Semester 5 ▼
```

Setelah semester diganti, seluruh data pada aplikasi otomatis mengikuti semester tersebut.

Artinya data **Semester 4 tidak akan tercampur dengan Semester 5** pada dashboard, analitik, maupun laporan LPJ.

---

# 📖 Cara Pakai

## 💸 Tambah Transaksi

Klik:

```text
+ Transaksi
```

Isi:

```text
Tipe
Nominal
Keterangan
Kategori
Tanggal
```

Kemudian klik **Simpan**.

---

## ⌨️ Shortcut

| Shortcut   | Aksi                        |
| ---------- | --------------------------- |
| `Ctrl + K` | Tambah transaksi            |
| `Cmd + K`  | Tambah transaksi pada macOS |

---

# 🤖 Scan Struk

1. Buka halaman **Scan Struk**
2. Upload atau drag & drop foto struk
3. Sistem membaca isi gambar
4. Nominal, tanggal, keterangan, dan data lainnya akan diisi otomatis
5. Periksa hasil scan
6. Edit jika diperlukan
7. Klik **Simpan Transaksi**

Tips:

> Gunakan foto dengan pencahayaan baik, tulisan jelas, dan posisi struk tidak terlalu miring agar hasil pembacaan lebih akurat.

---

# 📋 Export LPJ Beasiswa

Export LPJ menggunakan **semester yang sedang aktif**.

## Cara Export

1. Pilih semester pada dropdown
2. Klik **Export**
3. Pilih **LPJ Beasiswa**
4. Masukkan nominal dana beasiswa
5. Masukkan link bukti jika diperlukan
6. Klik **Generate XLSX**

Default dana beasiswa:

```text
Rp 8.400.000
```

---

## 📊 Isi File LPJ

File XLSX terdiri dari **3 sheet**.

| Sheet                   | Isi                                                        |
| ----------------------- | ---------------------------------------------------------- |
| 📊 **Dashboard**        | KPI dana, penggunaan dana, kategori, dan ringkasan bulanan |
| 📂 **Detail Transaksi** | Seluruh transaksi pada semester aktif                      |
| 📋 **LPJ**              | Format laporan pertanggungjawaban beasiswa                 |

Karena export mengikuti semester aktif:

```text
Semester 4 → hanya transaksi Semester 4
Semester 5 → hanya transaksi Semester 5
```

Data antarsemeseter tidak bercampur.

---

## 🪟 Catatan Windows

File XLSX hasil download browser terkadang diberi proteksi oleh Windows.

Jika Excel menolak file:

```text
Klik kanan file
→ Properties
→ Unblock
→ Apply
→ OK
```

Atau buka melalui:

```text
Excel
→ File
→ Open
```

---

# 📄 Export PDF

DuTrack juga dapat membuat laporan dalam format PDF yang berisi ringkasan keuangan dan transaksi sesuai data yang sedang ditampilkan.

---

# 🔒 Mode Lokal

DuTrack tetap dapat digunakan tanpa akun.

Pada halaman login pilih:

```text
Lanjut tanpa akun
```

Data akan disimpan menggunakan:

```text
localStorage
```

Mode ini tidak melakukan sinkronisasi ke Supabase.

> ⚠️ Data dapat hilang apabila browser cache/site data dihapus.

---

# 📱 Responsive Mobile

DuTrack dirancang agar dapat digunakan baik melalui desktop maupun smartphone.

Versi mobile memiliki penyesuaian pada:

* Sidebar dan navigasi
* Topbar
* Dropdown semester
* Tombol transaksi dan export
* Bento dashboard
* Daftar transaksi
* Grafik analitik
* Modal
* Account panel
* Layout dan spacing

Sehingga pencatatan pengeluaran maupun scan struk dapat dilakukan langsung dari HP.

---

# 🌙 Dark & Light Mode

Tema dapat diganti melalui menu:

```text
Mode Terang / Mode Gelap
```

Pilihan tampilan disesuaikan menggunakan CSS variables sehingga seluruh komponen mengikuti tema aktif.

---

# ⚙️ Catatan Teknis

* DuTrack menggunakan **Vanilla JavaScript** tanpa framework frontend.
* `script.js` dimuat setelah library pendukung agar fungsi export dan chart tersedia saat aplikasi dijalankan.
* Supabase digunakan untuk authentication, database, dan cloud storage.
* Row Level Security membatasi data berdasarkan akun pengguna.
* Filter semester dilakukan menggunakan rentang `start_month` dan `end_month`.
* Dashboard dan analitik menggunakan data hasil filter semester aktif.
* Mode lokal menggunakan `localStorage`.
* Gemini API Key disimpan di browser pengguna.
* Chrome direkomendasikan untuk kompatibilitas terbaik.
* Project Supabase Free dapat pause setelah tidak aktif dalam periode tertentu dan perlu di-resume melalui dashboard Supabase.

---

# 📦 Struktur Project

```text
DuTrack/
├── index.html
├── script.js
├── style.css
├── manifest.json
├── icon.png
└── README.md
```

### `index.html`

Struktur halaman aplikasi, dashboard, transaksi, analitik, scanner, pengaturan, modal, dan navigasi.

### `script.js`

Menangani:

```text
Authentication
Supabase Sync
Transaction CRUD
Semester Management
Semester Filtering
Charts & Analytics
OCR / AI Scan
Export XLSX
Export PDF
Theme
Navigation
Local Storage
```

### `style.css`

Berisi styling:

```text
Dark / Light Theme
Bento Dashboard
Desktop Layout
Responsive Mobile
Sidebar
Topbar
Transaction List
Charts
Modal
Scanner
Settings
```

### `manifest.json`

Konfigurasi Web App Manifest untuk pengalaman penggunaan aplikasi di perangkat mobile.

---

## 🌐 Live Demo

**[dutrack.vercel.app](https://dutrack.vercel.app)**

---

*Made with ☕ for scholarship finance tracking · Deployed on [Vercel](https://vercel.com) · Backend by [Supabase](https://supabase.com) · AI powered by [Gemini](https://aistudio.google.com)*
