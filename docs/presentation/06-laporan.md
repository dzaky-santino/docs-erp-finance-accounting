# Laporan (Reports)

> Menu sidebar: **Laporan**. Ini dokumen **terpenting bagi pemilik bisnis** — di sini setiap angka laporan dijelaskan dari mana asalnya dan bagaimana dihitung.
> Istilah dasar (Debit, Kredit, Aset, dll.): lihat **[08-konsep-dasar.md](08-konsep-dasar.md)**.

## Hal umum yang berlaku untuk semua laporan

- **Sumber data:** hampir semua laporan dibangun dari **Buku Besar** (`account_transactions`) — catatan inti semua transaksi. Beberapa laporan operasional (saldo pelanggan, aging) langsung dari dokumen faktur/tagihan.
- **Periode:** laporan posisi (Neraca, Penilaian Persediaan) dilihat **per satu tanggal**; laporan aktivitas (Laba Rugi, Arus Kas, Buku Besar) untuk **rentang tanggal**.
- **Basis Akrual vs Kas:** beberapa laporan (Neraca, Laba Rugi, Neraca Saldo) bisa ditampilkan dengan basis **Akrual** (default) atau **Kas**. Lihat penjelasan di [08-konsep-dasar.md](08-konsep-dasar.md).
- **Ekspor:** laporan dapat diunduh sebagai **CSV, XLSX (Excel), atau PDF**.

---

## 1. Neraca (Balance Sheet)

Menu: **Neraca**

### Apa ini?
Foto **kondisi keuangan perusahaan pada satu tanggal**: apa yang dimiliki, apa yang masih dihutang, dan berapa modal pemiliknya.

### Untuk apa?
Menjawab "Seberapa kaya/sehat perusahaan saya sekarang?". Dipakai bank, investor, dan pemilik untuk menilai posisi keuangan.

### Rumus utama
> **Total Aset = Total Kewajiban + Total Ekuitas**

Persamaan ini **selalu seimbang** karena prinsip double-entry. Inilah inti akuntansi.

### Tiga bagian
- **Aset** (yang dimiliki): Kas, Bank, Piutang, Persediaan, Peralatan, dll.
- **Kewajiban** (yang dihutang): Hutang Usaha, Hutang PPN, pinjaman.
- **Ekuitas** (modal): Modal Disetor, Laba Ditahan, **+ Laba Berjalan periode ini**.

### Data diambil dari mana & cara hitung
- Sistem menjumlahkan seluruh transaksi buku besar **sampai dengan** tanggal laporan, dikelompokkan per akun.
- Saldo tiap akun = sesuai saldo normalnya:
  - Akun aset (saldo normal Debit): **saldo = total Debit − total Kredit**.
  - Akun kewajiban & ekuitas (saldo normal Kredit): **saldo = total Kredit − total Debit**.
- **Laba bersih periode berjalan** dihitung (Pendapatan − Beban sampai tanggal itu) dan **dimasukkan ke Ekuitas**, karena laba menambah modal pemilik.

### Cara membaca (contoh)
```
ASET
  Kas & Bank          Rp  80.000.000
  Piutang Usaha       Rp  40.000.000
  Persediaan          Rp  30.000.000
  Peralatan           Rp  50.000.000
  TOTAL ASET          Rp 200.000.000

KEWAJIBAN
  Hutang Usaha        Rp  35.000.000
  Hutang PPN          Rp   5.000.000
  TOTAL KEWAJIBAN     Rp  40.000.000

EKUITAS
  Modal Disetor       Rp 100.000.000
  Laba Ditahan        Rp  40.000.000
  Laba Berjalan       Rp  20.000.000
  TOTAL EKUITAS       Rp 160.000.000

TOTAL KEWAJIBAN + EKUITAS = Rp 200.000.000  ✓ (= TOTAL ASET)
```
Jika kedua sisi tidak sama, ada masalah data — namun karena sistem menjaga keseimbangan otomatis, ini seharusnya tidak terjadi.

### Filter
- **Per tanggal** (as of date).
- **Basis** Akrual / Kas.

---

## 2. Laporan Laba/Rugi (Income Statement / Profit & Loss)

Menu: **Laporan Laba/Rugi**

### Apa ini?
Ringkasan **untung atau rugi** selama satu periode. Menunjukkan dari mana pendapatan datang dan ke mana beban pergi.

### Untuk apa?
Menjawab "Bulan/tahun ini saya untung berapa?". Laporan paling sering dilihat pemilik untuk menilai kinerja.

### Rumus
> **Laba Bersih = Total Pendapatan − Total Beban**

### Data diambil dari mana
- **Pendapatan**: saldo akun bertipe **Pendapatan** (`4-xxxx`) selama periode. Bertambah saat faktur/penerimaan penjualan diterbitkan.
- **Beban**: saldo akun bertipe **HPP** (`5-xxxx`) dan **Beban** (`6-xxxx`) selama periode.
- Sistem menjumlahkan transaksi buku besar dalam rentang tanggal, dikelompokkan per akun pendapatan/beban.

### Laba Kotor vs Laba Bersih
- **Laba Kotor** = Pendapatan − HPP. (Untung dari penjualan barang sebelum biaya operasional.)
- **Laba Bersih** = Laba Kotor − Beban Operasional. (Untung sesungguhnya setelah semua biaya.)

### HPP (Harga Pokok Penjualan)
Biaya langsung barang yang terjual. Memisahkan omzet dari untung sebenarnya. Lihat [08-konsep-dasar.md](08-konsep-dasar.md#hpp).

### Cara membaca (contoh)
```
PENDAPATAN
  Penjualan Barang        Rp 100.000.000
  Pendapatan Jasa         Rp  20.000.000
  TOTAL PENDAPATAN        Rp 120.000.000

HARGA POKOK PENJUALAN
  HPP Barang              Rp  60.000.000
  LABA KOTOR              Rp  60.000.000   (120jt − 60jt)

BEBAN OPERASIONAL
  Biaya Gaji              Rp  25.000.000
  Biaya Sewa              Rp   5.000.000
  Biaya Utilitas          Rp   3.000.000
  TOTAL BEBAN             Rp  33.000.000

LABA BERSIH               Rp  27.000.000   (60jt − 33jt)
```

### Perbedaan basis Akrual vs Kas
- **Akrual:** pendapatan diakui saat faktur dikirim (walau belum dibayar).
- **Kas:** pendapatan diakui saat uang diterima.
Contoh: faktur Juni dibayar Juli → akrual mencatat di Juni, kas mencatat di Juli.

### Filter
- **Rentang tanggal** (dari–sampai).
- **Basis** Akrual / Kas.

---

## 3. Laporan Arus Kas (Cash Flow Statement)

Menu: **Laporan Arus Kas**

### Apa ini?
Menunjukkan **ke mana uang kas mengalir** selama periode — dikelompokkan ke tiga jenis aktivitas.

### Untuk apa?
Menjawab "Untung kok kas saya tetap tipis?". **Untung tidak sama dengan punya uang** — bisa untung di atas kertas tapi kas habis karena uang tertahan di piutang/persediaan. Laporan ini menjelaskan selisihnya.

### Tiga bagian
- **Operasional (Operating):** kas dari kegiatan utama — penjualan, pembayaran pelanggan, pembelian, pembayaran tagihan, biaya. Dari tipe transaksi: Faktur, Penerimaan Pembayaran, Tagihan, Pembayaran Tagihan, Biaya, Penerimaan Penjualan.
- **Investasi (Investing):** kas untuk aset/persediaan jangka panjang. Dari tipe: Penyesuaian Persediaan, Biaya Tambahan (Landed Cost).
- **Pendanaan (Financing):** kas dari modal & pinjaman. Dari tipe: Jurnal Manual, Transaksi Kas (Cashflow).

### Data diambil dari mana & cara hitung
- Sistem mengambil **hanya transaksi pada akun Kas & Bank** dalam periode.
- Untuk tiap jenis transaksi: **arus bersih = total Debit − total Kredit** (Debit = kas masuk, Kredit = kas keluar).
- Dikelompokkan ke tiga bagian di atas, lalu dijumlahkan menjadi arus bersih per bagian.

### Net Change in Cash (Perubahan Bersih Kas)
Jumlah ketiga bagian = perubahan total kas selama periode. Positif = kas bertambah; negatif = kas berkurang.

### Cara membaca (contoh)
```
Arus Kas Operasional     Rp  +45.000.000   (kas masuk dari operasi)
Arus Kas Investasi       Rp  −10.000.000   (beli persediaan)
Arus Kas Pendanaan       Rp  +50.000.000   (setoran modal)
PERUBAHAN BERSIH KAS     Rp  +85.000.000
```

### Filter
- **Rentang tanggal**.

---

## 4. Buku Besar (General Ledger)

Menu: **Buku Besar**

### Apa ini?
Daftar **setiap transaksi pada tiap akun**, lengkap dengan saldo awal, mutasi, dan saldo akhir. Ini "catatan paling detail" — tempat audit satu akun spesifik.

### Untuk apa?
Menelusuri "akun ini isinya transaksi apa saja?". Contoh: melihat semua pergerakan akun Bank BCA bulan ini untuk mencocokkan dengan rekening koran.

### Data & cara hitung
- **Saldo Awal:** seluruh transaksi **sebelum** tanggal mulai, diringkas per akun.
- **Transaksi periode:** semua baris dalam rentang tanggal, urut per tanggal.
- **Saldo Berjalan (Running Balance):** saldo yang diperbarui baris demi baris. Untuk akun saldo-Debit, tiap Debit menambah & tiap Kredit mengurangi; untuk akun saldo-Kredit, sebaliknya.
- **Saldo Akhir** = Saldo Awal + mutasi periode.

### Cara membaca: "Debit menambah / Kredit menambah?"
Tergantung jenis akun (lihat tabel Debit/Kredit di [08-konsep-dasar.md](08-konsep-dasar.md#debit-vs-kredit)):
- Akun **Kas/Bank/Piutang** (aset): Debit menambah saldo.
- Akun **Hutang/Pendapatan** (kewajiban/pendapatan): Kredit menambah saldo.

### Filter
- **Rentang tanggal**.
- **Per akun** tertentu (untuk audit satu akun).
- **Per proyek** (bila fitur Proyek aktif).

---

## 5. Neraca Saldo (Trial Balance)

Menu: **Neraca Saldo**

### Apa ini?
Ringkasan **semua akun** dengan total Debit & total Kredit-nya dalam satu tabel.

### Untuk apa?
**Memverifikasi keseimbangan pembukuan** sebelum menyusun laporan keuangan. Total kolom Debit harus = total kolom Kredit. Bila tidak, ada kesalahan.

### Data & cara hitung
- Untuk tiap akun: jumlahkan seluruh Debit dan seluruh Kredit dalam periode.
- Tampilkan berdampingan, urut kode akun.
- Di bawah: **total Debit** dan **total Kredit** keseluruhan.

### Cara membaca (contoh)
```
Kode    Akun                  Debit          Kredit
1-1100  Kas                   80.000.000          0
1-1300  Piutang Usaha         40.000.000          0
2-1100  Hutang Usaha                   0   35.000.000
4-1200  Penjualan Barang              0  100.000.000
...
TOTAL                        255.000.000  255.000.000   ← harus sama!
```

### Filter
- **Rentang tanggal**, **Basis** Akrual/Kas.

---

## 6. Jurnal (Journal Sheet)

Menu: **Jurnal**

### Apa ini?
Daftar **semua jurnal** yang pernah terbentuk — baik otomatis (dari dokumen) maupun manual — dikelompokkan per transaksi sehingga tiap jurnal tampil sebagai satu unit Debit–Kredit yang seimbang.

### Untuk apa?
Melihat "transaksi apa saja yang terjadi dan jurnal apa yang dihasilkannya". Berguna untuk audit dan pembelajaran.

### Data & cara hitung
- Mengambil seluruh baris buku besar dalam periode, **dikelompokkan per dokumen sumber** (mis. semua baris dari Faktur #INV-000123 jadi satu jurnal).
- Tiap jurnal menampilkan baris-barisnya + total Debit & Kredit (selalu sama).
- Di bawah: **grand total** Debit & Kredit.

### Filter
- **Rentang tanggal**.

---

## 7. Ringkasan Umur Piutang (Receivables Aging Summary)

Menu: **Ringkasan Umur Piutang**

### Apa ini?
Mengelompokkan **piutang pelanggan berdasarkan berapa lama sudah lewat jatuh tempo**. Membantu mengenali tagihan yang menua/berisiko macet.

### Untuk apa?
Menjawab "Piutang mana yang sudah terlalu lama dan perlu ditagih keras?". Piutang yang makin tua makin berisiko tidak tertagih.

### Data & cara hitung
- Diambil dari **faktur yang sudah terbit** dan masih punya sisa (Sisa = balance − pembayaran − nota kredit > 0).
- Tiap sisa dimasukkan ke **bucket umur** berdasarkan selisih hari antara tanggal laporan dan **tanggal jatuh tempo**:

| Bucket | Arti |
| --- | --- |
| **Lancar (Current)** | Belum jatuh tempo |
| **1–30 hari** | Lewat jatuh tempo 1–30 hari |
| **31–60 hari** | Lewat 31–60 hari |
| **61–90 hari** | Lewat 61–90 hari |
| **>90 hari** | Lewat lebih dari 90 hari (risiko tinggi) |

- Dikelompokkan per pelanggan, dengan total per bucket dan total keseluruhan.

### Cara membaca
Pelanggan dengan saldo besar di kolom ">90 hari" perlu perhatian khusus — kemungkinan macet.

### Filter
- **Per tanggal**, **per pelanggan** (opsional).

---

## 8. Ringkasan Umur Utang (Payables Aging Summary)

Menu: **Ringkasan Umur Utang**

### Apa ini?
Kebalikan dari Umur Piutang: mengelompokkan **hutang Anda ke vendor** berdasarkan umur lewat jatuh tempo.

### Untuk apa?
Menjawab "Hutang mana yang sudah/akan jatuh tempo dan harus segera dibayar?". Membantu mengatur prioritas pembayaran agar tidak telat.

### Data & cara hitung
- Diambil dari **tagihan yang sudah dibuka** dan masih punya sisa (Sisa = amount − pembayaran − kredit vendor > 0).
- Bucket umur sama dengan Umur Piutang (Lancar, 1–30, 31–60, 61–90, >90), dihitung dari tanggal jatuh tempo tagihan.
- Dikelompokkan per vendor.

### Filter
- **Per tanggal**, **per vendor** (opsional).

---

## 9. Ringkasan Saldo Pelanggan (Customer Balance Summary)

Menu: **Ringkasan Saldo Pelanggan**

### Apa ini?
Per pelanggan: **total ditagih, total dibayar, total kredit, dan sisa piutang**.

### Untuk apa?
Melihat siapa pelanggan dengan piutang terbesar secara ringkas (tanpa rincian umur).

### Data & cara hitung
- Dari seluruh **faktur terbit** per pelanggan sampai tanggal laporan:
  - Total Ditagih = Σ nilai faktur
  - Total Dibayar = Σ pembayaran
  - Total Kredit = Σ nota kredit
  - **Sisa Piutang = Total Ditagih − Total Dibayar − Total Kredit**

### Filter
- **Per tanggal**.

---

## 10. Ringkasan Saldo Vendor (Vendor Balance Summary)

Menu: **Ringkasan Saldo Vendor**

### Apa ini?
Kebalikan dari Saldo Pelanggan — per vendor: total ditagihkan ke Anda, total dibayar, total kredit, dan **sisa hutang**.

### Data & cara hitung
- Dari seluruh **tagihan dibuka** per vendor:
  - **Sisa Hutang = Total Ditagihkan − Total Dibayar − Total Kredit Vendor**

### Filter
- **Per tanggal**.

---

## 11. Penjualan per Barang/Jasa (Sales by Items)

Menu: **Penjualan per Barang/Jasa**

### Apa ini?
Merangkum **kuantitas terjual dan nilai penjualan per barang/jasa**.

### Untuk apa?
Menjawab "Produk apa yang paling laku / paling besar omzetnya?". Untuk keputusan stok & strategi penjualan.

### Data & cara hitung
- Dari baris item pada **faktur yang sudah terbit** dalam periode:
  - Total Kuantitas = Σ kuantitas terjual per item
  - Total Nilai = Σ (kuantitas × harga) per item
- Diurutkan dari nilai terbesar.

### Filter
- **Rentang tanggal**.

---

## 12. Pembelian per Barang/Jasa (Purchases by Items)

Menu: **Pembelian per Barang/Jasa**

### Apa ini?
Kebalikan dari Penjualan per Barang — merangkum **kuantitas & nilai pembelian per barang/jasa** dari tagihan yang dibuka.

### Untuk apa?
Menjawab "Barang apa yang paling banyak/mahal saya beli?". Untuk negosiasi vendor & kendali biaya.

### Filter
- **Rentang tanggal**.

---

## 13. Transaksi per Kontak (Transactions by Contact)

Menu: **Transaksi per Kontak**

### Apa ini?
Semua transaksi (faktur, pembayaran, nota kredit, tagihan, pembayaran tagihan, kredit vendor) yang melibatkan **satu kontak tertentu**, urut tanggal.

### Untuk apa?
Melihat **riwayat lengkap hubungan keuangan** dengan satu pelanggan/vendor di satu tempat.

### Filter
- **Kontak** tertentu, **rentang tanggal**.

---

## 14. Transaksi per Referensi (Transactions by Reference)

Menu: **Transaksi per Referensi**

### Apa ini?
Menelusuri entri buku besar berdasarkan **nomor/jenis referensi dokumen** atau akun.

### Untuk apa?
Mencari semua baris jurnal yang terkait dengan satu dokumen atau nomor tertentu — untuk audit & penelusuran.

### Filter
- **Rentang tanggal**, **kata kunci referensi**, **per akun** (opsional).

---

## 15. Ringkasan Kewajiban Pajak Penjualan (Tax Summary)

Menu: **Ringkasan Kewajiban Pajak Penjualan**

### Apa ini?
Membandingkan **pajak yang dipungut** (dari penjualan) vs **pajak yang dibayar** (dari pembelian) untuk menghitung **pajak bersih yang harus disetor**.

### Untuk apa?
Menjawab "Bulan ini saya harus setor PPN berapa ke negara?".

### Data & cara hitung
- **Pajak Dipungut** = PPN dari transaksi penjualan (Faktur, Penerimaan Penjualan, Nota Kredit) = Σ Kredit − Σ Debit pada akun Pajak Terutang.
- **Pajak Dibayar** = PPN masukan dari pembelian (Tagihan, Kredit Vendor, Biaya) = Σ Debit − Σ Kredit pada akun Pajak Terutang.
- **Pajak Bersih Terutang = Pajak Dipungut − Pajak Dibayar**.

### Cara membaca
```
Pajak Dipungut (keluaran)   Rp 11.000.000
Pajak Dibayar (masukan)     Rp  4.000.000
PAJAK BERSIH DISETOR        Rp  7.000.000
```

### Filter
- **Rentang tanggal**.

---

## 16. Penilaian Persediaan (Inventory Valuation Sheet)

Menu: **Penilaian Persediaan**

### Apa ini?
**Nilai stok barang yang Anda miliki saat ini** — kuantitas dan nilai rupiahnya per item.

### Untuk apa?
Menjawab "Berapa nilai persediaan saya?" — angka ini muncul sebagai Aset di Neraca.

### Data & cara hitung
- Dari pergerakan persediaan sampai tanggal laporan:
  - **Kuantitas di tangan** = total masuk − total keluar.
  - **Biaya rata-rata** = total biaya pembelian masuk ÷ total kuantitas masuk (metode rata-rata).
  - **Nilai Total = Kuantitas × Biaya rata-rata**.
- Hanya item dengan stok/nilai > 0 yang ditampilkan.

### Cara membaca (contoh)
```
Barang           Qty    Biaya rata-rata   Nilai Total
Laptop            10    Rp  9.000.000     Rp 90.000.000
Mouse            100    Rp     50.000     Rp  5.000.000
TOTAL                                     Rp 95.000.000
```

### Filter
- **Per tanggal**.

---

## 17. Detail Barang Persediaan (Inventory Item Details)

Menu: **Detail Barang Persediaan**

### Apa ini?
Riwayat **setiap pergerakan stok satu barang tertentu** (masuk/keluar) dengan saldo berjalan dan gudangnya.

### Untuk apa?
Menelusuri "kenapa stok barang ini sekarang segini?" — melihat semua mutasinya satu per satu.

### Data & cara hitung
- Saldo awal sebelum tanggal mulai + setiap transaksi (IN/OUT) dalam periode, dengan **saldo berjalan** diperbarui tiap baris.

### Filter
- **Barang** tertentu, **rentang tanggal**.

---

## 18. Profitabilitas Proyek (Project Profitability) — opsional

Menu: **Profitabilitas Proyek** (hanya muncul bila fitur Proyek diaktifkan)

### Apa ini?
Laba/rugi **per proyek**: pendapatan proyek − beban proyek = laba proyek.

### Untuk apa?
Bagi perusahaan berbasis proyek (kontraktor, agensi): menilai proyek mana yang menguntungkan.

### Data & cara hitung
- Dari transaksi buku besar yang **ditandai proyek tertentu**:
  - Pendapatan proyek = Σ (Kredit − Debit) akun pendapatan.
  - Beban proyek = Σ (Debit − Kredit) akun beban.
  - **Laba = Pendapatan − Beban**.

### Filter
- **Rentang tanggal**, **per proyek** (opsional), **Basis** Akrual/Kas.

> Fitur Proyek dapat dimatikan lewat konfigurasi. Bila dimatikan, menu ini & kolom proyek di dokumen disembunyikan.

---

## Ringkasan: laporan mana untuk pertanyaan apa?

| Pertanyaan bisnis | Laporan |
| --- | --- |
| Seberapa sehat perusahaan saya sekarang? | **Neraca** |
| Untung/rugi periode ini? | **Laba/Rugi** |
| Kas saya ke mana? | **Arus Kas** |
| Pembukuan saya seimbang? | **Neraca Saldo** |
| Detail satu akun? | **Buku Besar** |
| Semua jurnal yang terjadi? | **Jurnal** |
| Piutang mana yang menua? | **Umur Piutang** + **Saldo Pelanggan** |
| Hutang mana yang jatuh tempo? | **Umur Utang** + **Saldo Vendor** |
| Produk terlaris / paling banyak dibeli? | **Penjualan/Pembelian per Barang** |
| Riwayat satu pelanggan/vendor? | **Transaksi per Kontak** |
| PPN yang harus disetor? | **Ringkasan Pajak** |
| Nilai & pergerakan stok? | **Penilaian/Detail Persediaan** |
| Proyek mana yang untung? | **Profitabilitas Proyek** |
