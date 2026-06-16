# Pengaturan (Preferences)

> Menu sidebar: **Sistem → Preferensi** (Umum, Pengguna, Estimasi/Faktur/Penerimaan/Nota Kredit, Mata Uang, Cabang, Gudang, Akuntansi, Barang/Jasa). Sebagian besar **khusus admin**.
> Istilah dasar: lihat **[08-konsep-dasar.md](08-konsep-dasar.md)**.

Modul Pengaturan adalah tempat menyiapkan "fondasi" sistem sebelum dipakai harian: siapa penggunanya, siapa pelanggan/vendornya, barang apa yang dijual, pajak & mata uang apa yang berlaku, serta preferensi tampilan dokumen. Diisi sekali di awal, lalu disesuaikan sesekali.

---

## 1. Pengguna & Peran (Users & Roles)

Menu: **Pengguna**

### Apa ini?
Mengelola **siapa saja yang boleh login** dan **apa yang boleh mereka lakukan**.

### Cara menambah pengguna baru
Pengguna baru ditambahkan lewat **undangan**: sistem mengirim email undangan, lalu calon pengguna membuat kata sandinya sendiri saat menerima undangan. Setiap pengguna diberi **satu/lebih peran**.

### Peran yang tersedia
Sistem menyediakan peran siap pakai (lihat juga ringkasan di [00-pengantar.md](00-pengantar.md#siapa-saja-penggunanya-peran--role)):

| Peran | Ringkas |
| --- | --- |
| **Super Admin / Admin** | Akses penuh, termasuk Pengaturan |
| **Manajer Keuangan** | Penuh kecuali Pengaturan sistem |
| **Akuntan** | Akuntansi & Laporan penuh; Penjualan/Pembelian hanya lihat |
| **Staf Penjualan** | Dokumen penjualan + tambah kontak/barang |
| **Staf Pembelian** | Dokumen pembelian + tambah kontak/barang |
| **Kasir** | Catat pembayaran & penerimaan; tidak bisa buat faktur/tagihan |
| **Pembaca Laporan** | Hanya melihat + ekspor laporan |

Peran kustom baru juga bisa dibuat dengan memilih izin (permission) secara rinci per modul & aksi.

### Mengapa hak akses penting?
- **Keamanan:** data keuangan sensitif tidak dilihat semua orang.
- **Pengendalian internal:** pemisahan tugas mengurangi risiko kecurangan (mis. yang membuat faktur berbeda dari yang menerima uang).
- **Mencegah kesalahan:** pengguna tidak sengaja mengubah hal di luar tanggung jawabnya.

---

## 2. Kontak (Contacts)

Menu: **Kontak → Pelanggan / Vendor**

### Apa ini?
Daftar semua **pelanggan** (yang membeli dari Anda) dan **vendor** (tempat Anda membeli). Satu kontak menyimpan nama, alamat penagihan & pengiriman, telepon, email, NPWP/catatan, website, dll.

### Customer vs Vendor
- **Customer (Pelanggan):** muncul saat membuat Faktur, Estimasi, Penerimaan Penjualan, Nota Kredit, Penerimaan Pembayaran.
- **Vendor (Pemasok):** muncul saat membuat Tagihan, Pembayaran Tagihan, Kredit Vendor.

### Opening Balance (Saldo Awal)
Saat **pindah dari sistem lama** ke sistem ini, pelanggan/vendor mungkin sudah punya saldo piutang/hutang berjalan. **Opening Balance** mencatat saldo awal itu agar laporan tidak dimulai dari nol.

Contoh: pelanggan A sudah berhutang Rp 15 juta sebelum Anda pakai sistem ini → masukkan sebagai opening balance per tanggal mulai.

---

## 3. Barang/Jasa (Items)

Menu: **Barang/Jasa** (dan **Kategori**)

### Apa ini?
Daftar **produk/jasa** yang Anda jual atau beli. Tiap item menyimpan harga jual, harga beli, akun pendapatan/biaya terkait, dan deskripsi.

### Tipe item
- **Inventaris (Inventory):** barang fisik yang **stoknya dilacak**. Punya jumlah di tangan (`quantity_on_hand`), nilai persediaan, dan HPP saat terjual.
- **Jasa / Non-Inventaris (Service):** layanan/barang yang **stoknya tidak dilacak** (mis. jasa konsultasi). Selalu bisa dijual tanpa batasan stok.

> Aturan penting: pada **form penjualan baru**, item Inventaris dengan stok habis (≤ 0) **disembunyikan** agar tidak menjual barang yang tidak ada; item Jasa selalu tampil.

### Harga jual vs harga beli
- **Harga jual (sell price):** otomatis terisi saat item dipilih di Faktur/Estimasi.
- **Harga beli (cost price):** otomatis terisi saat dipilih di Tagihan.
- Untuk dokumen mata uang asing, harga otomatis dikonversi dengan kurs.

---

## 4. Tarif Pajak (Tax Rates)

Menu: **Keuangan → Tarif Pajak**

### Apa ini?
Mendefinisikan tarif pajak yang berlaku (mis. **PPN 11%**, **PPN 12%**, PPh, dll.).

### Cara sistem menghitung pajak otomatis
Saat Anda memilih tarif pajak pada baris faktur/tagihan, sistem menghitung **Pajak = tarif % × nilai baris**.

Contoh: baris Rp 10.000.000 dengan PPN 11% → pajak Rp 1.100.000.

- **Eksklusif:** pajak ditambahkan di atas harga (total Rp 11.100.000).
- **Inklusif:** pajak sudah termasuk dalam harga (sistem mengeluarkan porsinya).

Nilai pajak ini otomatis tercatat ke akun **Hutang PPN / Pajak Terutang** di buku besar, dan muncul di laporan **Ringkasan Pajak**.

---

## 5. Mata Uang (Currencies)

Menu: **Mata Uang**

### Apa ini?
Mengelola mata uang yang dipakai & **kurs** terhadap mata uang dasar (Rupiah).

### Multi-currency: cara kerja konversi
- Mata uang dasar perusahaan adalah **IDR (Rupiah)** — ini disetel di konfigurasi sistem dan **tidak diubah lewat UI** (untuk menjaga konsistensi seluruh data historis).
- Dokumen dalam mata uang asing menyimpan nilainya dalam mata uang itu, tetapi **buku besar selalu mencatat ekuivalen Rupiah** (nilai × kurs).

### Kurs historis: mengapa per tanggal?
Kurs berubah tiap hari. Sistem menyimpan **kurs per tanggal** sehingga transaksi lama tetap memakai kurs saat itu — laporan historis akurat dan tidak berubah hanya karena kurs hari ini berbeda.

- Saat membuat dokumen mata uang asing, sistem **otomatis mengisi kurs** dari kurs terdekat ≤ tanggal transaksi.
- Tersedia **Riwayat Kurs** per mata uang. Kurs **terakhir** sebuah mata uang tidak boleh dihapus (pengaman agar selalu ada acuan).

---

## 6. Cabang & Gudang (Branches & Warehouses)

Menu: **Cabang**, **Gudang**

### Apa ini?
- **Cabang (Branch):** lokasi/unit usaha (mis. Kantor Pusat Jakarta, Cabang Surabaya). Transaksi bisa ditandai cabangnya.
- **Gudang (Warehouse):** lokasi penyimpanan stok (mis. Gudang Utama Jakarta). Dipakai untuk pelacakan persediaan & transfer gudang.

Masing-masing punya satu lokasi **utama (primary)** dan diberi **kode** unik.

---

## 7. Preferensi Dokumen & Umum (Preferences)

Menu: **Umum**, dan **Estimasi/Faktur/Penerimaan/Nota Kredit** (teks default dokumen)

### Penomoran dokumen otomatis
Sistem memberi **nomor urut otomatis** ke setiap dokumen (mis. Faktur `INV-000123`, Estimasi `EST-00045`). Anda tidak perlu menomori manual — sistem menjamin tidak ada nomor dobel.

### Format tanggal
Anda memilih format tampilan tanggal (mis. `30 Juni 2026` atau `30/06/2026`). Pilihan ini diterapkan di **seluruh halaman** secara dinamis.

### Identitas organisasi
Nama perusahaan, NPWP, industri, alamat lengkap, telepon, dan teks footer PDF — muncul di kop dokumen PDF (faktur, estimasi, dll.).

### Teks default dokumen
Catatan & syarat-ketentuan default (mis. "Pembayaran jatuh tempo 30 hari", info rekening) yang otomatis muncul saat membuat dokumen baru, sehingga tidak perlu mengetik ulang tiap kali.

### Bahasa
Bahasa organisasi (Indonesia/Inggris) memengaruhi tampilan UI **dan** pesan validasi/flash backend.

---

## Ringkasan modul Pengaturan

| Pengaturan | Untuk apa |
| --- | --- |
| Pengguna & Peran | Siapa boleh login & melakukan apa |
| Kontak | Data pelanggan & vendor (+ saldo awal) |
| Barang/Jasa | Produk/jasa, harga, pelacakan stok |
| Tarif Pajak | Perhitungan PPN/PPh otomatis |
| Mata Uang | Multi-currency & kurs historis |
| Cabang & Gudang | Lokasi usaha & penyimpanan stok |
| Preferensi | Penomoran, format tanggal, identitas, teks default, bahasa |
