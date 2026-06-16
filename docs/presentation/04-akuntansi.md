# Akuntansi (Accounting)

> Menu sidebar: **Keuangan** (Bagan Akun, Jurnal Manual, Penguncian Transaksi, Tarif Pajak), **Penjualan & Persediaan** (Penyesuaian Persediaan, Transfer Gudang), **Biaya**.
> Istilah dasar: lihat **[08-konsep-dasar.md](08-konsep-dasar.md)**.

Modul ini adalah "mesin akuntansi" di belakang layar. Sebagian besar dipakai oleh **Akuntan** atau **Admin**. Sementara modul Penjualan/Pembelian membuat jurnal otomatis, modul ini menangani struktur akun, jurnal manual, dan kontrol periode.

---

## 1. Bagan Akun (Chart of Accounts / COA)

Menu: **Keuangan → Bagan Akun**

### Apa ini?
Daftar lengkap semua **akun** ("laci" pencatatan) perusahaan. Ini adalah kerangka pembukuan — setiap transaksi pasti masuk ke salah satu akun di sini.

### Mengapa perlu dibuat?
Tanpa akun, tidak ada tempat mengelompokkan uang. Akun memungkinkan sistem menjawab "berapa total biaya listrik tahun ini?" atau "berapa saldo bank saya?".

### Lima kelompok akun (dengan contoh nyata)

| Kelompok | Arti awam | Contoh akun di sistem |
| --- | --- | --- |
| **Aset** (`1-xxxx`) | Yang **dimiliki** perusahaan | Kas (`1-1100`), Bank BCA (`1-1200`), Piutang Usaha (`1-1300`), Persediaan Barang (`1-1400`), Peralatan Kantor (`1-2100`) |
| **Kewajiban** (`2-xxxx`) | Yang masih **harus dibayar** | Hutang Usaha (`2-1100`), Hutang PPN (`2-1200`) |
| **Ekuitas** (`3-xxxx`) | **Modal** pemilik & laba ditahan | Modal Disetor (`3-1100`), Laba Ditahan (`3-1200`) |
| **Pendapatan** (`4-xxxx`) | Uang **masuk dari penjualan** | Pendapatan Jasa (`4-1100`), Pendapatan Penjualan Barang (`4-1200`) |
| **HPP** (`5-xxxx`) | Biaya **barang yang terjual** | HPP Barang (`5-1100`), Biaya Langsung Jasa (`5-1200`) |
| **Beban** (`6-xxxx`) | Biaya **operasional** | Biaya Gaji (`6-1100`), Biaya Sewa Kantor (`6-1200`), Biaya Utilitas (`6-1300`) |

### Mengapa pakai nomor (kode)?
Kode seperti `1-1100` memudahkan **pengurutan otomatis** dan **pengelompokan**. Awalan angka pertama menandai kelompok besar (1=Aset, 2=Kewajiban, dst.). Ini standar yang membuat laporan tersusun rapi tanpa perlu menyortir manual.

### Catatan
- Akun **tidak di-seed kosong**; perusahaan membuat & menyesuaikan sendiri. Daftar di atas adalah contoh standar Indonesia.
- Beberapa akun bersifat **akun sistem** yang wajib ada agar dokumen bisa jalan: **Piutang Usaha**, **Hutang Usaha**, **Pajak Terutang**. Bila belum dibuat, sistem akan memberi pesan jelas saat Anda mencoba menerbitkan dokumen.
- Tersedia juga fitur **impor** akun untuk membuat banyak akun sekaligus.

---

## 2. Jurnal Manual (Manual Journal)

Menu: **Keuangan → Jurnal Manual**

### Apa ini?
Cara **mencatat transaksi langsung ke buku besar** untuk kejadian yang tidak punya dokumen khusus (bukan penjualan/pembelian/biaya). Di sini Anda mengetik sendiri baris Debit dan Kredit.

### Prinsip double-entry
Setiap jurnal manual harus punya minimal dua baris, dan **total Debit harus sama dengan total Kredit**. Sistem **menolak menyimpan/menerbitkan** jika tidak seimbang — ini pengaman utama.

### Contoh nyata: bayar sewa kantor tunai Rp 5.000.000
```
Debit  Biaya Sewa Kantor   Rp 5.000.000   (beban bertambah)
Kredit Kas                 Rp 5.000.000   (kas berkurang)
```
Total Debit = Total Kredit = Rp 5.000.000 ✓

### Contoh lain: penyusutan peralatan bulanan Rp 1.000.000
```
Debit  Biaya Penyusutan        Rp 1.000.000
Kredit Akumulasi Penyusutan    Rp 1.000.000
```

### Kapan perlu jurnal manual? (vs jurnal otomatis)
- **Jurnal otomatis** muncul sendiri dari Faktur, Tagihan, Pembayaran, Biaya.
- **Jurnal manual** untuk: penyusutan aset, akrual gaji, koreksi kesalahan, setoran modal, penyesuaian akhir periode, dan transaksi non-rutin lain.

### Status
- **Draft** — tersimpan tapi belum memengaruhi laporan (belum di-posting).
- **Diterbitkan (Published)** — sudah masuk buku besar & laporan. Saat publish, sistem mengecek keseimbangan dan periode terkunci.

---

## 3. Biaya / Pengeluaran (Expense)

Menu: **Biaya**

### Apa ini?
Cara cepat mencatat **pengeluaran tunai/langsung** yang langsung membebani kas — tanpa melalui tagihan vendor.

### Bedanya dengan Pembayaran Tagihan
- **Pembayaran Tagihan** = melunasi hutang (Bill) yang sudah dicatat sebelumnya.
- **Biaya/Expense** = pengeluaran langsung yang **tidak** pernah jadi hutang. Bayar sekarang, catat sekarang, selesai.

### Contoh
Beli alat tulis tunai Rp 500.000; bayar listrik Rp 2.000.000 lewat transfer; isi bensin operasional.

### Efek ke sistem
Saat diterbitkan, jurnal otomatis:
- **Debit** Akun Beban (per kategori biaya, mis. Biaya Utilitas)
- **Kredit** Akun Pembayaran (Kas/Bank yang dipakai)

Contoh bayar listrik Rp 2.000.000 dari Bank:
```
Debit  Biaya Utilitas   Rp 2.000.000   (beban bertambah)
Kredit Bank BCA          Rp 2.000.000   (kas/bank berkurang)
```

> Satu dokumen biaya bisa berisi beberapa kategori sekaligus (mis. listrik + air + internet), masing-masing dibebankan ke akun beban-nya, dengan total dikreditkan dari satu akun pembayaran.

---

## 4. Penyesuaian Persediaan (Inventory Adjustment)

Menu: **Penjualan & Persediaan → Penyesuaian Persediaan**

### Apa ini?
Cara **mengoreksi jumlah stok barang** di sistem agar cocok dengan kenyataan fisik di gudang.

### Kapan perlu?
- **Stok opname**: setelah hitung fisik, ternyata jumlah sistem ≠ jumlah nyata.
- **Barang rusak/hilang/kedaluwarsa**: stok harus dikurangi.
- **Stok awal**: saat pertama memakai sistem, memasukkan stok yang sudah ada.

### Dua jenis penyesuaian
- **Increment (Tambah stok):** menambah jumlah barang.
  Jurnal: **Debit** Persediaan (aset naik), **Kredit** Akun Penyesuaian.
- **Decrement (Kurangi stok):** mengurangi jumlah barang.
  Jurnal: **Debit** Akun Penyesuaian, **Kredit** Persediaan (aset turun).

### Efek ke sistem
Selain mengubah jumlah stok (`quantity_on_hand`), penyesuaian yang diterbitkan juga membuat **jurnal seimbang** sehingga **nilai persediaan di Neraca** ikut menyesuaikan.

> Contoh nyata di data demo: stok awal 3 barang dimasukkan lewat penyesuaian "Increment" → Debit Persediaan, Kredit Modal Disetor.

---

## 5. Transfer Gudang (Warehouse Transfer)

Menu: **Penjualan & Persediaan → Transfer Gudang** (khusus admin)

### Apa ini?
Memindahkan stok barang **dari satu gudang ke gudang lain**. Contoh: kirim 20 unit dari Gudang Jakarta ke Gudang Surabaya.

### Efek ke sistem
Jumlah total stok perusahaan **tidak berubah** — hanya lokasinya yang berpindah. Berguna untuk perusahaan dengan banyak gudang/cabang.

---

## 6. Penguncian Transaksi (Transaction Locking)

Menu: **Keuangan → Penguncian Transaksi** (Akuntan/Admin)

### Apa ini?
Fitur untuk **"menutup buku" sebuah periode** dengan mengunci tanggal tertentu ke belakang, sehingga **tidak ada lagi yang bisa membuat/mengubah/menghapus transaksi** di periode itu.

### Mengapa penting?
Setelah laporan keuangan suatu bulan/tahun difinalisasi (dan mungkin sudah dilaporkan ke pajak/manajemen), angkanya **tidak boleh berubah lagi**. Tanpa penguncian, seseorang bisa diam-diam mengedit transaksi lama dan merusak laporan yang sudah final.

### Cara kerja
- Tetapkan tanggal kunci. Semua transaksi pada/sebelum tanggal itu menjadi "beku".
- Saat seseorang mencoba mengubah dokumen di periode terkunci (mis. menerbitkan faktur bertanggal lama, menghapus tagihan lama), sistem **menolak** dengan pesan jelas.
- Penguncian dicek per modul (Penjualan, dll.) di banyak titik — termasuk saat hapus, edit, deliver, refund, dan void.

### Siapa yang bisa?
Hanya peran dengan izin `setting.edit` — yaitu **Akuntan** dan **Admin**.

---

## 7. Tarif Pajak (Tax Rates)

Menu: **Keuangan → Tarif Pajak** (admin). Detail di **[07-pengaturan.md](07-pengaturan.md)**.

Mendefinisikan tarif pajak (mis. PPN 11%, PPN 12%, PPh) yang dipakai otomatis saat membuat faktur/tagihan. Sistem menghitung nilai pajak = tarif % × nilai baris.

---

## 8. Audit Log (Jejak Audit)

Sistem otomatis mencatat **siapa** membuat/mengubah/menghapus tiap dokumen dan **kapan**, lengkap dengan email pengguna. Ini bukan menu yang dipakai harian, tapi menjadi bukti pertanggungjawaban saat ada pemeriksaan atau perselisihan data.

---

## Ringkasan modul Akuntansi

| Fitur | Untuk apa | Membuat jurnal? |
| --- | --- | --- |
| Bagan Akun | Struktur akun | Tidak (hanya kerangka) |
| Jurnal Manual | Transaksi non-rutin, koreksi, penyusutan | Ya (manual, wajib seimbang) |
| Biaya | Pengeluaran tunai langsung | Ya (Debit beban, Kredit kas/bank) |
| Penyesuaian Persediaan | Koreksi/stok awal barang | Ya (menyesuaikan nilai persediaan) |
| Transfer Gudang | Pindah stok antar gudang | Tidak mengubah total stok |
| Penguncian Transaksi | Tutup buku periode | Tidak (kontrol, bukan transaksi) |
| Tarif Pajak | Definisi pajak otomatis | Tidak (acuan perhitungan) |
