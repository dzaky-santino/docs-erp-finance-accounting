# Perbankan (Banking)

> Menu sidebar: **Perbankan** → Akun Bank, Transaksi untuk Ditinjau, Aturan Bank, Tambah Uang Masuk/Keluar. (Khusus admin.)
> Istilah dasar: lihat **[08-konsep-dasar.md](08-konsep-dasar.md)**.

Modul Perbankan mengelola **kas & rekening bank** secara langsung — untuk uang masuk/keluar yang **bukan** dari faktur/tagihan, serta untuk **mencocokkan** catatan sistem dengan mutasi bank asli (rekonsiliasi).

---

## 1. Akun Bank / Kas (Bank Accounts)

Menu: **Perbankan → Akun Bank**

### Apa ini?
Daftar semua **rekening bank dan kas** perusahaan, masing-masing dengan **saldo terkini**.

### Nilai yang tampil & dari mana
- **Saldo per akun**: dihitung dari buku besar = total (Debit − Kredit) seluruh transaksi pada akun kas/bank tersebut. Karena kas/bank bersaldo normal Debit, uang masuk (debit) menaikkan saldo, uang keluar (kredit) menurunkannya.
- **Aktivitas terakhir** & jumlah transaksi yang masih perlu ditinjau juga ditampilkan.

### Cara menambah uang masuk / keluar manual
Dari sini Anda bisa langsung mencatat **Uang Masuk** atau **Uang Keluar** (lihat bagian 2 & 3).

---

## 2. Transaksi Masuk / Uang Masuk (Money In)

Menu: **Perbankan → Tambah Uang Masuk**

### Apa ini?
Mencatat **uang yang masuk ke kas/bank** yang **bukan** berasal dari pembayaran pelanggan (Payment Receive). Untuk pembayaran pelanggan, gunakan menu Penjualan → Penerimaan Pembayaran.

### Contoh
- **Setoran modal** dari pemilik.
- **Pinjaman bank** cair.
- **Pendapatan bunga** dari rekening.
- **Hibah/pengembalian** dari pihak lain.

### Efek ke sistem
Jurnal otomatis: **Debit** akun Kas/Bank (uang masuk) dan **Kredit** akun lawan yang Anda pilih (mis. Modal Disetor untuk setoran modal, atau Hutang Bank untuk pinjaman).

Contoh setoran modal Rp 50.000.000 ke Bank:
```
Debit  Bank BCA        Rp 50.000.000   (kas/bank naik)
Kredit Modal Disetor   Rp 50.000.000   (ekuitas naik)
```

---

## 3. Transaksi Keluar / Uang Keluar (Money Out)

Menu: **Perbankan → Tambah Uang Keluar**

### Apa ini?
Mencatat **uang keluar dari kas/bank** yang bukan untuk membayar tagihan vendor (Bill Payment) dan bukan biaya operasional rutin (Expense).

### Contoh
- **Penarikan pemilik** (prive).
- **Pembayaran cicilan pinjaman** bank.
- **Transfer antar rekening** sendiri.

### Efek ke sistem
Jurnal otomatis: **Kredit** akun Kas/Bank (uang keluar) dan **Debit** akun lawan yang dipilih.

> **Catatan pemilihan menu yang tepat:**
> - Bayar tagihan vendor → **Pembayaran Tagihan** (modul Pembelian).
> - Beli sesuatu tunai langsung sebagai beban → **Biaya** (modul Akuntansi).
> - Uang keluar lain (cicilan, prive, transfer) → **Uang Keluar** di sini.

---

## 4. Tinjau Transaksi / Rekonsiliasi Bank (Bank Review)

Menu: **Perbankan → Transaksi untuk Ditinjau**

### Apa itu rekonsiliasi?
**Mencocokkan catatan sistem dengan mutasi rekening bank yang sebenarnya.** Bank Anda mengeluarkan daftar mutasi (uang masuk/keluar nyata). Tugas rekonsiliasi: memastikan setiap baris mutasi bank itu sudah tercatat dengan benar di sistem.

Analogi: seperti mencocokkan struk belanja dengan catatan pengeluaran pribadi — memastikan tidak ada yang terlewat atau dobel.

### Alur status
```
Belum Dikategorikan (Open)  ──(dikategorikan / dicocokkan)──►  Selesai (Resolved)
                            └──(diabaikan)──►  Dikecualikan (Excluded)
```

- **Belum Dikategorikan (Open):** transaksi bank masuk tapi belum jelas masuk ke akun/dokumen apa.
- **Dikategorikan / Dicocokkan (Resolved):** sudah dipasangkan ke akun yang benar atau dicocokkan dengan dokumen yang sudah ada di sistem.
- **Dikecualikan (Excluded):** sengaja diabaikan (mis. duplikat atau bukan transaksi perusahaan).

### Cara kerja
1. Tambahkan/impor transaksi bank yang perlu ditinjau.
2. Untuk tiap transaksi, **kategorikan** (tentukan akun lawannya) atau **cocokkan** dengan dokumen yang sudah ada (mis. faktur yang pembayarannya sudah masuk).
3. Setelah dikategorikan, sistem membuat catatan kas yang sesuai dan transaksi berpindah ke status Selesai.

---

## 5. Aturan Bank (Bank Rules)

Menu: **Perbankan → Aturan Bank**

### Apa ini?
**Otomatisasi** untuk mempercepat rekonsiliasi. Anda membuat aturan: "jika transaksi bank cocok dengan kondisi tertentu, kategorikan otomatis ke akun ini."

### Contoh
- "Setiap transaksi keluar yang keterangannya mengandung kata 'PLN' → kategorikan otomatis ke **Biaya Utilitas**."
- "Setiap transaksi masuk dari rekening tertentu → kategorikan ke **Pendapatan Jasa**."

### Mengapa berguna?
Bila Anda punya ratusan transaksi bank per bulan, aturan ini menghemat waktu besar dengan mengategorikan transaksi rutin secara otomatis, menyisakan hanya yang tidak biasa untuk ditinjau manual.

---

## Ringkasan modul Perbankan

| Fitur | Untuk apa | Jurnal |
| --- | --- | --- |
| Akun Bank | Lihat saldo kas & rekening | — (menampilkan saldo) |
| Uang Masuk | Kas masuk non-penjualan (modal, pinjaman) | Debit Kas/Bank, Kredit akun lawan |
| Uang Keluar | Kas keluar non-pembelian (cicilan, prive) | Kredit Kas/Bank, Debit akun lawan |
| Tinjau Transaksi | Rekonsiliasi dengan mutasi bank | Membuat catatan kas saat dikategorikan |
| Aturan Bank | Otomatisasi kategorisasi | — (mempercepat proses) |
