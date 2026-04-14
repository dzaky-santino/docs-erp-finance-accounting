# ERP Finance & Accounting — Presentasi Sistem

---

## BAGIAN 1 — PEMBUKA

### Apa Itu Sistem Ini?

**ERP Finance & Accounting** adalah aplikasi berbasis web untuk mengelola keuangan perusahaan secara lengkap dan otomatis. Sistem ini mencatat setiap transaksi keuangan — mulai dari penagihan pelanggan, pembayaran ke vendor, hingga pembuatan laporan keuangan — dalam satu platform terpadu yang dapat diakses melalui browser.

Dengan sistem ini, perusahaan tidak lagi perlu mencatat transaksi secara manual di spreadsheet atau buku besar fisik. Semua data keuangan tersimpan aman, terstruktur, dan siap dianalisis kapan saja.

---

### Masalah Bisnis yang Diselesaikan

Perusahaan dagang dan jasa menengah di Indonesia sering menghadapi masalah berikut:

| Masalah | Solusi dalam Sistem |
|---------|---------------------|
| Invoice pelanggan tidak terlacak | Modul Piutang (AR) dengan status real-time |
| Tagihan vendor terlewat dibayar | Modul Hutang (AP) dengan peringatan jatuh tempo |
| Laporan keuangan dibuat manual | 7 jenis laporan otomatis dari data transaksi |
| Tidak tahu saldo kas saat ini | Dashboard real-time dengan ringkasan keuangan |
| Transaksi multi-mata uang rumit | Dukungan 50+ mata uang dengan kurs historis |
| Tidak ada jejak siapa yang mengubah data | Audit trail otomatis di setiap transaksi |
| Data mudah dihapus secara tidak sengaja | Soft delete — data tidak pernah hilang permanen |

---

### Siapa yang Menggunakan Sistem Ini?

Sistem ini dirancang untuk **perusahaan dagang dan jasa menengah di Indonesia**, khususnya yang:

- Memiliki tim finance/akuntansi internal
- Bertransaksi dalam Rupiah (IDR) dan/atau mata uang asing
- Membutuhkan laporan keuangan yang akurat dan cepat
- Perlu membatasi akses data keuangan berdasarkan jabatan

**Pengguna per peran:**

| Peran | Siapa | Aktivitas Utama |
|-------|-------|-----------------|
| **Admin** | Manajer Keuangan / CFO | Kelola semua modul, pengaturan sistem, hapus data |
| **Staff** | Staf Akuntansi / Finance | Buat dan edit invoice, bill, jurnal, pengeluaran |
| **Viewer** | Direksi / Auditor | Lihat semua data dan laporan tanpa bisa mengubah |

---

### Manfaat Utama untuk Bisnis

- **Hemat waktu** — Jurnal akuntansi dibuat otomatis saat invoice/tagihan diproses
- **Akurasi tinggi** — Sistem double-entry memastikan debit selalu sama dengan kredit
- **Visibilitas penuh** — Dashboard dan laporan real-time untuk pengambilan keputusan
- **Kepatuhan pajak** — Laporan ringkasan pajak (PPN/PPh) siap pakai
- **Multi-mata uang** — Transaksi dalam 50+ mata uang dengan kurs otomatis
- **Keamanan data** — Riwayat transaksi tidak dapat dihapus permanen

---

## BAGIAN 2 — FITUR UTAMA

### 1. Dashboard — Ringkasan Keuangan Real-Time

**Kegunaan:** Halaman pertama yang dilihat saat login. Menampilkan kondisi keuangan perusahaan secara sekilas.

**Pengguna:** Semua peran (Admin, Staff, Viewer)

**Yang bisa dilihat:**
- Total piutang pelanggan yang belum dibayar
- Total hutang ke vendor yang belum dibayar
- Grafik pendapatan vs pengeluaran bulanan
- Invoice dan tagihan yang akan jatuh tempo
- Ringkasan arus kas

**Manfaat nyata:** Manajer dapat mengambil keputusan cepat tanpa harus menggali data satu per satu.

---

### 2. Finance — Piutang (Accounts Receivable / AR)

Modul ini mengelola semua transaksi **penjualan ke pelanggan**.

#### 2a. Invoice (Faktur Penjualan)

**Kegunaan:** Mengirimkan tagihan resmi kepada pelanggan untuk barang/jasa yang sudah diberikan.

**Pengguna:** Admin, Staff

**Fitur utama:**
- Buat invoice dengan banyak baris item (barang/jasa)
- Tambahkan diskon, pajak PPN, dan penyesuaian
- Kirim ke pelanggan (status berubah dari Draft ke Delivered)
- Terima pembayaran sebagian atau lunas
- Hapus piutang tak tertagih (write-off)
- Duplikasi invoice yang serupa
- Cetak dalam format siap kirim

**Status invoice:** `Draft → Terkirim → Belum Dibayar → Dibayar Sebagian → Lunas`

**Manfaat nyata:** Tidak ada invoice yang terlewat atau lupa ditagih. Saldo piutang selalu akurat.

---

#### 2b. Estimasi / Penawaran Harga

**Kegunaan:** Mengirimkan penawaran harga kepada calon pelanggan sebelum deal terjadi.

**Pengguna:** Admin, Staff

**Fitur utama:**
- Buat penawaran dengan rincian item dan harga
- Kirim ke pelanggan untuk disetujui atau ditolak
- Jika disetujui, konversi langsung menjadi invoice

**Status:** `Draft → Terkirim → Disetujui/Ditolak → Dikonversi`

**Manfaat nyata:** Proses dari penawaran ke invoice hanya satu klik — tidak perlu input ulang data.

---

#### 2c. Nota Kredit

**Kegunaan:** Mengembalikan sebagian atau seluruh nilai invoice kepada pelanggan (misalnya untuk retur barang atau koreksi harga).

**Pengguna:** Admin, Staff

**Fitur utama:**
- Buat nota kredit dengan alasan pengembalian
- Terapkan nota kredit untuk mengurangi saldo invoice
- Atau refund langsung ke rekening pelanggan

**Manfaat nyata:** Proses retur dan koreksi tercatat rapi tanpa mengubah data invoice asli.

---

#### 2d. Penerimaan Pembayaran

**Kegunaan:** Mencatat pembayaran yang diterima dari pelanggan, bisa untuk satu atau beberapa invoice sekaligus.

**Pengguna:** Admin, Staff

**Fitur utama:**
- Pilih pelanggan → sistem otomatis tampilkan invoice yang belum lunas
- Alokasikan pembayaran ke invoice tertentu
- Mendukung pembayaran parsial

**Manfaat nyata:** Saldo piutang diperbarui otomatis. Tidak perlu hitung manual.

---

### 3. Finance — Hutang (Accounts Payable / AP)

Modul ini mengelola semua transaksi **pembelian dari vendor**.

#### 3a. Tagihan (Bill)

**Kegunaan:** Mencatat tagihan yang diterima dari vendor atas barang/jasa yang sudah diterima.

**Pengguna:** Admin, Staff

**Fitur utama:**
- Catat tagihan dengan rincian barang/jasa dan jumlah
- Buka tagihan untuk meng-aktifkan kewajiban pembayaran
- Lacak jatuh tempo dan status pembayaran
- Duplikasi tagihan berulang (misalnya sewa bulanan)

**Status:** `Draft → Dibuka → Belum Dibayar → Jatuh Tempo → Dibayar Sebagian → Lunas`

**Manfaat nyata:** Tidak ada tagihan vendor yang terlewat. Arus kas terencana dengan baik.

---

#### 3b. Kredit Vendor

**Kegunaan:** Mencatat kredit atau pengembalian dari vendor (misalnya barang yang diretur).

**Pengguna:** Admin, Staff

**Fitur utama:**
- Buat kredit vendor dengan rincian item
- Terapkan ke tagihan yang ada untuk mengurangi hutang
- Atau terima refund tunai dari vendor

---

#### 3c. Pembayaran Tagihan

**Kegunaan:** Mencatat pembayaran yang dilakukan ke vendor untuk satu atau beberapa tagihan.

**Pengguna:** Admin, Staff

**Fitur utama:**
- Pilih vendor → tampilkan tagihan yang belum lunas
- Alokasikan pembayaran ke tagihan tertentu
- Mendukung pembayaran parsial

---

### 4. Akuntansi

#### 4a. Bagan Akun (Chart of Accounts)

**Kegunaan:** Daftar lengkap semua akun keuangan perusahaan yang menjadi dasar pencatatan setiap transaksi.

**Pengguna:** Admin

**Fitur utama:**
- Tampilkan bagan akun dalam format pohon (hierarki) atau tabel datar
- Tambah akun baru dengan tipe yang sesuai
- Kelola hierarki induk-anak akun (maksimal 5 level)
- Akun sistem (AR, AP, PPN) dilindungi dari penghapusan

**Jenis akun yang didukung:** Kas, Bank, Piutang, Persediaan, Aset Tetap, Hutang Usaha, PPN, Modal, Pendapatan, HPP, Beban

---

#### 4b. Jurnal Manual

**Kegunaan:** Mencatat transaksi akuntansi yang tidak berasal dari invoice/tagihan — misalnya penyesuaian akhir bulan, depresiasi aset, atau koreksi kesalahan.

**Pengguna:** Admin, Staff

**Fitur utama:**
- Buat jurnal dengan banyak baris debit/kredit
- Sistem validasi: total debit harus = total kredit
- Simpan sebagai draft, lalu posting ke buku besar
- Edit jurnal yang belum diposting

**Status:** `Draft → Diposting`

---

#### 4c. Pengeluaran (Expense)

**Kegunaan:** Mencatat pengeluaran operasional yang dibayar langsung (bukan via tagihan vendor), misalnya biaya perjalanan, listrik, atau operasional kantor.

**Pengguna:** Admin, Staff

**Fitur utama:**
- Kategori beban yang fleksibel per baris pengeluaran
- Pilih akun kas/bank yang digunakan untuk pembayaran
- Draft → Posting ke buku besar

---

### 5. Laporan Keuangan

**Kegunaan:** Menghasilkan laporan keuangan resmi dari data transaksi yang sudah tercatat.

**Pengguna:** Admin, Staff (lihat), Viewer (lihat)

| Laporan | Kegunaan |
|---------|---------|
| **Neraca (Balance Sheet)** | Posisi keuangan pada tanggal tertentu: Aset = Liabilitas + Modal |
| **Laba Rugi (Income Statement)** | Pendapatan vs Pengeluaran dalam periode tertentu |
| **Neraca Saldo (Trial Balance)** | Daftar semua akun dengan saldo debit/kredit untuk verifikasi |
| **Arus Kas (Cash Flow)** | Aliran masuk/keluar kas dari operasional, investasi, pendanaan |
| **Aging Piutang** | Analisis umur piutang: pelanggan mana yang terlambat bayar |
| **Aging Hutang** | Analisis umur hutang: tagihan mana yang hampir jatuh tempo |
| **Ringkasan Pajak** | PPN dipungut vs PPN dibayar dalam periode tertentu |

**Manfaat nyata:** Tidak perlu menunggu akuntan eksternal. Laporan tersedia seketika kapan saja dibutuhkan.

---

### 6. Pengaturan Sistem

**Pengguna:** Admin saja

| Pengaturan | Fungsi |
|------------|--------|
| **Organisasi** | Nama perusahaan, mata uang dasar, format tanggal, zona waktu, dasar akuntansi |
| **Mata Uang** | Tambah/edit mata uang, atur kurs harian |
| **Tarif Pajak** | Kelola tarif PPN (11%/12%) dan PPh (21/23/4(2)) |
| **Item/Produk** | Katalog barang dan jasa yang bisa digunakan di invoice/tagihan |
| **Kontak** | Direktori pelanggan dan vendor |
| **Pengguna** | Tambah/kelola akun pengguna dan tetapkan peran (Admin/Staff/Viewer) |

---

## BAGIAN 3 — ALUR BISNIS UTAMA

### Alur 1 — Proses Penjualan Lengkap

```
1. BUAT PENAWARAN
   └── Isi item, harga, pelanggan → Simpan sebagai Draft

2. KIRIM PENAWARAN ke Pelanggan
   └── Status: Draft → Terkirim

3. PELANGGAN SETUJU
   └── Status: Terkirim → Disetujui

4. KONVERSI ke Invoice
   └── Sistem otomatis buat Invoice dari data Penawaran
   └── Invoice status: Draft

5. KIRIM INVOICE ke Pelanggan
   └── Status: Draft → Terkirim → Belum Dibayar
   └── Sistem otomatis catat: Piutang (+) dan Pendapatan (+)

6. TERIMA PEMBAYARAN
   └── Status: Belum Dibayar → Dibayar Sebagian (jika partial)
                             → Lunas (jika full)
   └── Sistem otomatis catat: Kas/Bank (+) dan Piutang (-)
```

**Catatan:** Setiap panah otomatis membuat jurnal akuntansi tanpa input manual.

---

### Alur 2 — Proses Pembelian Lengkap

```
1. TERIMA TAGIHAN dari Vendor
   └── Input data tagihan → Simpan sebagai Draft

2. BUKA TAGIHAN
   └── Status: Draft → Dibuka/Belum Dibayar
   └── Sistem otomatis catat: Beban (+) dan Hutang (+)

3. BAYAR TAGIHAN
   └── Status: Belum Dibayar → Dibayar Sebagian (partial)
                             → Lunas (full)
   └── Sistem otomatis catat: Hutang (-) dan Kas/Bank (-)

4. TAGIHAN SELESAI ✓
```

---

### Alur 3 — Pencatatan Jurnal Manual

```
1. BUAT JURNAL
   └── Isi baris debit dan kredit beserta akun dan nominal

2. VALIDASI OTOMATIS
   └── Sistem cek: Total Debit = Total Kredit
   └── Jika tidak seimbang → ditolak dengan pesan error

3. SIMPAN sebagai Draft
   └── Bisa diedit sebelum diposting

4. POSTING ke Buku Besar
   └── Status: Draft → Diposting
   └── Muncul di semua laporan keuangan
```

---

### Alur 4 — Melihat Laporan Keuangan

```
1. PILIH JENIS LAPORAN
   └── Neraca / Laba Rugi / Arus Kas / dll.

2. ATUR PERIODE
   └── Pilih tanggal atau rentang tanggal

3. LIHAT HASIL
   └── Laporan dihitung real-time dari data transaksi

4. CETAK atau EXPORT
   └── Tersedia tampilan cetak yang siap dikirim ke auditor/direksi
```

---

## BAGIAN 4 — ARSITEKTUR SISTEM `[TEKNIS]`

### Tech Stack

| Komponen | Teknologi | Versi |
|----------|-----------|-------|
| **Backend Framework** | Laravel | 13.x (PHP 8.3+) |
| **Frontend Library** | React | 19.x |
| **Bahasa Frontend** | TypeScript | 5.7+ |
| **Bridge SPA** | Inertia.js | 2.x |
| **CSS Framework** | Tailwind CSS | 4.x |
| **Build Tool** | Vite | 8.x |
| **Database** | MySQL | 8.0+ |
| **Autentikasi** | Laravel Sanctum | 4.3+ |
| **Otorisasi** | Spatie Laravel Permission | 7.2+ |
| **Chart Library** | Recharts | 2.15+ |
| **Form Management** | React Hook Form | 7.54+ |
| **Icon Library** | Lucide React | 0.460+ |

---

### Arsitektur Monolith Modular

Sistem ini adalah **modular monolith** — satu unit deployable dengan modul-modul internal yang terpisah jelas:

```
┌─────────────────────────────────────────────────────┐
│                   ERP Finance & Accounting           │
├─────────────────────────────────────────────────────┤
│  Core         │  Auth, Settings, User Management    │
│  Master Data  │  Accounts, Contacts, Items, Taxes   │
│  Finance AR   │  Invoices, Estimates, Credit Notes  │
│  Finance AP   │  Bills, Bill Payments, Vendor Credits│
│  Accounting   │  Journals, Expenses, GL             │
│  Reports      │  7 Financial Reports                │
└─────────────────────────────────────────────────────┘
         ↓ semua berbagi database yang sama
```

---

### Lapisan Aplikasi

Setiap request melewati 4 lapisan berurutan:

```
HTTP Request dari Browser
        │
        ▼
┌─────────────────────────────────────┐
│  FORM REQUEST                       │
│  - Validasi input (Zod di frontend) │
│  - Cek permission (authorize())     │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  CONTROLLER                         │
│  - Terima request HTTP              │
│  - Delegate ke Service              │
│  - Return Inertia/JSON response     │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  SERVICE                            │
│  - Business rules & validasi domain │
│  - Buat GL entries (double-entry)   │
│  - Throw domain exceptions          │
│  - Fire events                      │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  MODEL (Eloquent ORM)               │
│  - Mapping ke tabel database        │
│  - Relasi antar entitas             │
│  - Scopes, casts, Auditable trait   │
└─────────────┬───────────────────────┘
              │
              ▼
         MySQL Database
```

---

### Mengapa Laravel + Inertia.js + React?

**Laravel** dipilih karena:
- Ekosistem terlengkap untuk aplikasi bisnis di PHP
- Eloquent ORM untuk relasi kompleks antar tabel keuangan
- Sanctum untuk session-based auth yang aman
- Spatie Permission untuk RBAC yang fleksibel

**Inertia.js** dipilih karena:
- Menghilangkan kebutuhan API terpisah (REST API tidak diperlukan)
- Routing tetap di server (Laravel) — keamanan lebih baik
- Pengalaman SPA (Single Page Application) tanpa full page reload
- Controllers melayani SPA dan API eksternal secara bersamaan

**React + TypeScript** dipilih karena:
- Type safety mencegah bug di form kompleks (invoice dengan banyak baris item)
- Component reuse — `DataTable`, `LineItemsTable`, `DatePicker` dipakai di semua modul
- Inertia adapter React yang matang dan stabil

---

### Sistem Autentikasi (Sanctum)

```
Web (Inertia SPA):
  Login → Session cookie (database driver) → Auth middleware

API External:
  POST /api/login → Personal Access Token → Bearer token header
```

- Self-registration **dinonaktifkan** — user hanya dibuat oleh admin
- Session tersimpan di tabel `sessions` di database (bukan file)
- Token API di tabel `personal_access_tokens`

---

### Sistem Otorisasi RBAC (Spatie Permission)

```
User → syncRoles(['admin']) → Role has permissions
                                      ↓
FormRequest::authorize()  → $this->user()->can('sale-invoice.create')
                                      ↓
AppLayout.tsx sidebar     → usePermission().can('account.create')
                                      ↓
routes/web.php            → middleware('permission:sale-invoice.view')
```

Format permission: `{module}.{action}` — contoh: `sale-invoice.create`, `bill.delete`, `report.view`

Permission list di-cache per user selama 30 menit di `user.permissions.{id}`.

---

### Double-Entry Accounting Engine

Inti sistem ada di tabel `account_transactions` (General Ledger). Setiap event keuangan menghasilkan jurnal yang seimbang:

```
Invoice Delivered:
  DEBIT  → Piutang Usaha (12001)     [full amount]
  CREDIT → Pendapatan Penjualan      [amount ex-tax per baris]
  CREDIT → PPN Keluaran (22001)      [tax amount]

Payment Received:
  DEBIT  → Kas/Bank                  [payment amount]
  CREDIT → Piutang Usaha (12001)     [same amount]

Bill Opened:
  DEBIT  → Akun Beban/HPP            [amount ex-tax per baris]
  DEBIT  → PPN Masukan               [tax amount]
  CREDIT → Hutang Usaha (21001)      [full amount]
```

GL entries dibuat di service layer saat dokumen berpindah status. Saat dokumen dihapus, GL entries di-revert (dihapus dari `account_transactions`).

---

### Fitur Teknis Lainnya

| Fitur | Implementasi |
|-------|-------------|
| **Multi-currency** | Setiap dokumen menyimpan `currency_code` + `exchange_rate`. GL entries disimpan dalam base currency |
| **Soft delete** | Semua model financial pakai `SoftDeletes` — data tidak pernah hilang permanen |
| **Audit trail** | `created_by`, `updated_by`, `deleted_by` di semua 46 model via `Auditable` trait |
| **URL Masking** | Address bar selalu tampilkan `/` — monkey-patch `history.pushState` di `app.tsx` |
| **Persistent Layout** | `AppLayout` tidak unmount saat navigasi — sidebar state tetap terjaga |
| **Permission Cache** | Permission per user di-cache 30 menit, setting organisasi di-cache 60 menit |
| **Chunk Splitting** | Vite split vendor chunks: react, inertia, recharts, icons — loading lebih cepat |
| **Lazy Loading** | Setiap halaman adalah chunk JS terpisah — hanya didownload saat dikunjungi |

---

## BAGIAN 5 — KEAMANAN SISTEM

### Tiga Level Akses

Sistem memiliki 3 peran dengan hak akses yang berbeda:

| Hak Akses | Admin | Staff | Viewer |
|-----------|:-----:|:-----:|:------:|
| **Lihat semua data dan laporan** | ✓ | ✓ | ✓ |
| **Buat invoice, tagihan, jurnal** | ✓ | ✓ | — |
| **Edit invoice, tagihan, jurnal** | ✓ | ✓ | — |
| **Hapus transaksi** | ✓ | — | — |
| **Akses pengaturan sistem** | ✓ | — | — |
| **Kelola pengguna** | ✓ | — | — |
| **Buat/edit akun di bagan akun** | ✓ | — | — |
| **Lihat laporan keuangan** | ✓ | ✓ | ✓ |

---

### Tabel Hak Akses Per Modul

| Modul | Admin | Staff | Viewer |
|-------|-------|-------|--------|
| Invoice | Lihat, Buat, Edit, Hapus, Write-Off | Lihat, Buat, Edit | Lihat |
| Estimasi | Lihat, Buat, Edit, Hapus | Lihat, Buat, Edit | Lihat |
| Nota Kredit | Lihat, Buat, Hapus | Lihat, Buat | Lihat |
| Penerimaan Pembayaran | Lihat, Buat, Hapus | Lihat, Buat | Lihat |
| Tagihan (Bill) | Lihat, Buat, Edit, Hapus | Lihat, Buat, Edit | Lihat |
| Kredit Vendor | Lihat, Buat, Hapus | Lihat, Buat | Lihat |
| Pembayaran Tagihan | Lihat, Buat, Hapus | Lihat, Buat | Lihat |
| Jurnal Manual | Lihat, Buat, Edit, Hapus | Lihat, Buat, Edit | Lihat |
| Pengeluaran | Lihat, Buat, Edit, Hapus | Lihat, Buat, Edit | Lihat |
| Bagan Akun | Lihat, Buat, Edit, Hapus | Lihat | Lihat |
| Pengaturan | Lihat, Edit | Lihat | Lihat |
| Laporan | Lihat | Lihat | Lihat |

---

### Audit Trail — Siapa Mengubah Apa

Setiap data di sistem memiliki 3 kolom audit:

| Kolom | Isi | Contoh |
|-------|-----|--------|
| `created_by` | ID user yang membuat data | "Dibuat oleh: Budi (Staff)" |
| `updated_by` | ID user yang terakhir mengubah | "Diubah oleh: Ani (Admin)" |
| `deleted_by` | ID user yang menghapus data | "Dihapus oleh: Admin" |

Kolom ini diisi **otomatis** oleh sistem — staf tidak perlu input manual. Ini memungkinkan manajemen untuk menelusuri siapa yang membuat perubahan pada setiap transaksi.

---

### Perlindungan Data Keuangan

**Soft Delete — Data Tidak Pernah Hilang Permanen**

Ketika pengguna "menghapus" sebuah transaksi, data tidak benar-benar dihapus dari database. Sistem menandainya dengan `deleted_at` timestamp. Data bisa dipulihkan jika diperlukan, dan riwayat GL entries tetap utuh.

**Validasi Berlapis**

```
Pengguna isi form
      │
      ▼ Validasi di Frontend (React Hook Form + Zod)
      │   - Format tanggal, angka wajib positif, field wajib diisi
      │
      ▼ Validasi di Form Request (Laravel)
      │   - Permission check, tipe data, batas nilai
      │
      ▼ Validasi di Service (Business Rules)
          - Saldo cukup, status valid, tidak duplikat nomor
```

**Aturan Penghapusan Akun Keuangan**

Akun di bagan akun tidak dapat dihapus jika:
1. Akun adalah akun sistem (AR, AP, PPN, dll.) — dilindungi permanen
2. Akun masih memiliki sub-akun — harus bersihkan sub-akun dulu
3. Akun sudah memiliki transaksi GL — riwayat keuangan tidak dapat dihapus

---

## BAGIAN 6 — DATABASE DAN DATA `[TEKNIS]`

### Statistik Database

| Kategori | Jumlah |
|----------|--------|
| Total tabel | 43+ tabel |
| Migration files | 39 custom migrations |
| Model Eloquent | 46 model |
| Soft deletes | Semua model transaksi keuangan |
| Audit columns | Semua 46 model |

---

### Pengelompokan Tabel

| Kelompok | Tabel Utama | Fungsi |
|----------|-------------|--------|
| **Sistem** | `users`, `sessions`, `personal_access_tokens` | Auth & sessions |
| **RBAC** | `roles`, `permissions`, `model_has_roles`, `model_has_permissions` | Otorisasi |
| **Master Data** | `accounts`, `currencies`, `contacts`, `items`, `tax_rates`, `settings` | Data referensi |
| **AR** | `sale_invoices`, `sale_estimates`, `payment_receives`, `credit_notes` | Piutang |
| **AP** | `bills`, `bill_payments`, `vendor_credits` | Hutang |
| **Baris Item** | `item_entries` | Baris detail invoice/tagihan/estimasi (polymorphic) |
| **Akuntansi** | `account_transactions`, `manual_journals`, `expenses` | GL & jurnal |
| **Pendukung** | `exchange_rates`, `branches`, `warehouses` | Data operasional |

---

### Strategi Indexing untuk Performa

```sql
-- Tabel account_transactions (GL terbesar):
INDEX (reference_type, reference_id)    -- cari semua jurnal untuk invoice #42
INDEX (account_id, date)               -- hitung saldo akun dalam rentang tanggal

-- Tabel sale_invoices:
INDEX (customer_id, deleted_at)        -- daftar invoice aktif per pelanggan
INDEX (customer_id, deleted_at, invoice_date) -- sama, urut per tanggal

-- Tabel bills:
INDEX (vendor_id, deleted_at)          -- daftar tagihan aktif per vendor
INDEX (vendor_id, deleted_at, bill_date)
```

---

### Tipe Data Keuangan

```sql
-- Nilai uang: decimal(15,5) — mendukung nilai hingga 999 miliar dengan presisi 5 desimal
balance         DECIMAL(15,5)
amount          DECIMAL(15,5)

-- Kurs tukar: decimal(13,9) — presisi tinggi untuk perhitungan multi-currency
exchange_rate   DECIMAL(13,9)

-- Tarif pajak: decimal(8,4)
tax_rate        DECIMAL(8,4)
```

---

### Data Awal (Seeding)

Setelah `php artisan db:seed` dijalankan, sistem siap pakai dengan:

- **Mata uang:** 50+ mata uang dunia (IDR, USD, EUR, SGD, dll.)
- **Bagan Akun:** 26 akun kelompok + 50+ akun leaf sesuai PSAK
- **Tarif Pajak:** PPN 11%, PPN 12%, PPh 21/23/4(2)
- **Role & Permission:** 3 role (admin/staff/viewer) dengan 79 permission
- **User Demo:** admin@erp.test, staff@erp.test, viewer@erp.test (password: password)

---

## BAGIAN 7 — INTEGRASI DAN DEPLOYMENT `[TEKNIS]`

### Pola Integrasi

Sistem ini dapat diintegrasikan dengan sistem eksternal melalui:

**REST API (Bearer Token)**
```bash
# Login untuk mendapatkan token
POST /api/login   → { user, token }

# Gunakan token di semua request berikutnya
Authorization: Bearer {token}

# Contoh: ambil daftar invoice
GET /api/invoices

# Buat invoice baru
POST /api/invoices
```

**Semua endpoint tersedia di dua jalur:**
- `/api/*` — untuk integrasi eksternal (Bearer token)
- `/` routes — untuk Inertia SPA (session cookie)

---

### Persyaratan Server

| Komponen | Kebutuhan Minimum |
|----------|-------------------|
| **PHP** | 8.3+ dengan ekstensi: BCMath, Ctype, JSON, Mbstring, OpenSSL, PDO, Tokenizer, XML |
| **MySQL** | 8.0+ (atau PostgreSQL) |
| **Node.js** | 18+ (hanya untuk build frontend) |
| **Web Server** | Nginx (disarankan) atau Apache |
| **RAM** | 512 MB minimum, 2 GB disarankan |
| **Disk** | 500 MB untuk aplikasi + ruang untuk database |

---

### Langkah Deployment dari Nol

```bash
# 1. Clone repository
git clone https://github.com/your-org/erp-finance-accounting.git
cd erp-finance-accounting

# 2. Install PHP dependencies
composer install --no-dev --optimize-autoloader

# 3. Install dan build frontend
npm install
npm run build

# 4. Konfigurasi environment
cp .env.example .env
php artisan key:generate
# Edit .env: DB_HOST, DB_DATABASE, DB_USERNAME, DB_PASSWORD, APP_URL

# 5. Jalankan migration database
php artisan migrate

# 6. Seed data awal (WAJIB: AccountSeeder + RolePermissionSeeder)
php artisan db:seed --class=RolePermissionSeeder
php artisan db:seed --class=AccountSeeder

# 7. Buat admin user pertama
php artisan tinker
>>> App\Models\User::create([
...   'name' => 'Administrator',
...   'email' => 'admin@company.com',
...   'password' => bcrypt('password_kuat_anda')
... ])->syncRoles(['admin']);

# 8. Optimasi untuk production
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan optimize

# 9. Set permission folder
chmod -R 775 storage bootstrap/cache
php artisan storage:link
```

---

### Perintah Penting di Production

```bash
php artisan optimize          # Cache semua (config, route, view, events)
php artisan cache:clear       # Hapus semua cache (termasuk settings.organization)
php artisan migrate           # Jalankan migration baru saat update
php artisan down              # Mode maintenance
php artisan up                # Matikan maintenance mode
php artisan tinker            # REPL untuk debugging/data manipulation
```

---

## BAGIAN 8 — PENUTUP

### Ringkasan Nilai Bisnis

ERP Finance & Accounting memberikan nilai nyata bagi perusahaan dalam 3 area utama:

**1. Efisiensi Operasional**
- Jurnal akuntansi dibuat otomatis — staf tidak perlu input ganda
- Laporan keuangan tersedia dalam hitungan detik
- Estimasi dapat dikonversi ke invoice dengan satu klik

**2. Akurasi dan Kepatuhan**
- Double-entry accounting memastikan tidak ada transaksi yang "hilang"
- Laporan pajak (PPN/PPh) tersedia untuk pelaporan SPT
- Bagan akun sesuai standar PSAK Indonesia

**3. Kontrol dan Visibilitas**
- Tiga level akses memastikan data hanya bisa diubah oleh yang berhak
- Audit trail lengkap untuk setiap perubahan data
- Dashboard real-time untuk monitoring kondisi keuangan

---

### Potensi Pengembangan Selanjutnya

| Fitur | Prioritas | Keterangan |
|-------|-----------|------------|
| Multi-tenant | Tinggi | Menambah `organization_id` di semua tabel untuk SaaS |
| Email notifikasi | Sedang | Kirim invoice/reminder jatuh tempo via email |
| File attachment | Sedang | Upload PDF/gambar ke invoice dan tagihan |
| Transaksi berulang | Sedang | Invoice/tagihan otomatis bulanan |
| Paginasi API | Sedang | List endpoint saat ini return semua data |
| Mobile app | Rendah | Akses laporan dari smartphone |
| Integrasi e-Faktur | Tinggi | Submit PPN langsung ke DJP Online |

---

### Mulai Menggunakan

1. Hubungi tim pengembang untuk instalasi dan konfigurasi awal
2. Jalankan seeder untuk data awal (bagan akun + role)
3. Admin membuat akun pengguna untuk tim finance
4. Input data master: kontak pelanggan/vendor, item/produk
5. Mulai catat transaksi pertama

**Akun Demo (Development):**

| Nama | Email | Password | Peran |
|------|-------|----------|-------|
| Administrator | admin@erp.test | password | Admin |
| Staff User | staff@erp.test | password | Staff |
| Viewer User | viewer@erp.test | password | Viewer |

> **Perhatian:** Ganti password demo sebelum digunakan di production.

---

*Dokumentasi ini dibuat untuk keperluan presentasi internal dan onboarding tim.*
*Versi sistem: Laravel 13 + React 19 + Inertia.js 2*
