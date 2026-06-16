# Konsep Dasar & Kamus Istilah

> Dokumen ini adalah **fondasi** untuk memahami semua menu dan laporan dalam sistem.
> Bacalah ini terlebih dahulu. Istilah-istilah di sini dipakai berulang di seluruh dokumen presentasi.
> Semua penjelasan dipasangkan dengan **cara sistem ini bekerja secara nyata** (bukan teori akuntansi umum).

---

## Mengapa ada akuntansi sama sekali?

Bayangkan Anda punya warung. Setiap hari uang masuk dan keluar. Di akhir bulan Anda ingin tahu:

- Berapa untung saya bulan ini?
- Siapa saja yang masih berhutang ke saya?
- Saya masih harus bayar siapa?
- Uang kas saya sekarang berapa, dan ke mana perginya?

Akuntansi adalah **cara mencatat semua kejadian uang** secara rapi sehingga keempat pertanyaan di atas bisa dijawab kapan saja dengan akurat. Sistem ini melakukan pencatatan itu secara otomatis setiap kali Anda membuat dokumen (faktur, tagihan, pembayaran, dll.).

---

## Double-entry / Pembukuan Berpasangan

### Apa ini?

Prinsip paling penting dalam sistem ini: **setiap transaksi selalu dicatat di DUA tempat sekaligus** — satu sisi "Debit" dan satu sisi "Kredit" — dengan **nilai yang sama persis**.

Analogi: setiap kali uang bergerak, ia selalu **datang dari suatu tempat** dan **pergi ke suatu tempat**. Kalau Anda membayar sewa Rp 5.000.000 tunai, maka:
- Kas Anda **berkurang** Rp 5.000.000 (uang pergi)
- Beban Sewa Anda **bertambah** Rp 5.000.000 (manfaat datang)

Dua sisi, nilai sama. Itulah double-entry.

### Mengapa penting?

Karena dua sisi selalu sama, **total Debit harus selalu sama dengan total Kredit** di seluruh sistem. Jika tidak sama, berarti ada kesalahan pencatatan. Ini adalah "rem pengaman" otomatis: laporan keuangan tidak mungkin "bocor" tanpa ketahuan.

Dalam sistem ini, aturan ini dijaga ketat: setiap dokumen yang membuat jurnal selalu menghasilkan baris-baris yang seimbang (lihat istilah **Jurnal** di bawah).

---

## Debit vs Kredit

Ini sumber kebingungan terbesar bagi orang awam. **Debit BUKAN berarti "bertambah" dan Kredit BUKAN berarti "berkurang".** Artinya **tergantung jenis akun**.

Setiap akun punya "saldo normal" — sisi di mana saldonya bertambah:

| Kelompok Akun | Contoh | Saldo Normal | Debit berarti | Kredit berarti |
| --- | --- | --- | --- | --- |
| **Aset** (yang dimiliki) | Kas, Bank, Piutang, Persediaan, Peralatan | **Debit** | Bertambah ⬆ | Berkurang ⬇ |
| **Kewajiban/Hutang** | Hutang Usaha, Hutang PPN | **Kredit** | Berkurang ⬇ | Bertambah ⬆ |
| **Ekuitas/Modal** | Modal Disetor, Laba Ditahan | **Kredit** | Berkurang ⬇ | Bertambah ⬆ |
| **Pendapatan** | Penjualan, Pendapatan Jasa | **Kredit** | Berkurang ⬇ | Bertambah ⬆ |
| **Beban/Biaya** | Gaji, Sewa, Listrik, HPP | **Debit** | Bertambah ⬆ | Berkurang ⬇ |

> **Cara mudah mengingat:** Aset dan Beban "suka" Debit (saldo normalnya di kiri). Kewajiban, Ekuitas, dan Pendapatan "suka" Kredit (saldo normalnya di kanan).

Contoh nyata di sistem ini saat Anda **menerbitkan faktur penjualan** Rp 11.100.000:
- **Debit** Piutang Usaha Rp 11.100.000 → aset (piutang) bertambah
- **Kredit** Pendapatan Penjualan Rp 10.000.000 → pendapatan bertambah
- **Kredit** Hutang PPN Rp 1.100.000 → kewajiban pajak bertambah

Total Debit (11.100.000) = Total Kredit (10.000.000 + 1.100.000). Seimbang. ✓

---

## Akun (Account) & Bagan Akun (Chart of Accounts / COA)

### Apa ini?

**Akun** adalah "laci" atau "kantong" tempat satu jenis nilai dikumpulkan. Contoh: semua uang di rekening BCA masuk ke akun "Bank BCA". Semua biaya listrik masuk ke akun "Biaya Utilitas".

**Bagan Akun (COA)** adalah **daftar lengkap semua laci** yang Anda punya. Ini adalah kerangka pembukuan Anda.

### Kode akun

Setiap akun punya nomor agar mudah dikelompokkan dan diurutkan. Sistem ini memakai pola Indonesia:

| Awalan | Kelompok | Contoh akun di sistem |
| --- | --- | --- |
| `1-xxxx` | **Aset** | `1-1100` Kas, `1-1200` Bank BCA, `1-1300` Piutang Usaha, `1-1400` Persediaan Barang |
| `2-xxxx` | **Kewajiban** | `2-1100` Hutang Usaha, `2-1200` Hutang PPN |
| `3-xxxx` | **Ekuitas** | `3-1100` Modal Disetor, `3-1200` Laba Ditahan, `3-1300` Laba Berjalan |
| `4-xxxx` | **Pendapatan** | `4-1100` Pendapatan Jasa, `4-1200` Pendapatan Penjualan Barang |
| `5-xxxx` | **HPP** | `5-1100` HPP Barang, `5-1200` Biaya Langsung Jasa |
| `6-xxxx` | **Beban** | `6-1100` Biaya Gaji, `6-1200` Biaya Sewa Kantor, `6-1300` Biaya Utilitas |

> Catatan: akun **tidak dibuat otomatis** oleh sistem dari nol — daftar di atas adalah contoh standar yang disiapkan untuk demo. Perusahaan membuat/menyesuaikan akunnya sendiri lewat menu Bagan Akun.

---

## Jurnal & Jurnal Otomatis vs Manual

### Apa itu Jurnal?

**Jurnal** adalah catatan satu transaksi lengkap dengan kedua sisinya (Debit & Kredit). Contoh jurnal "bayar sewa tunai":

```
Tanggal 15 Jun 2026 — Bayar sewa kantor
  Debit  : Biaya Sewa Kantor    Rp 5.000.000
  Kredit : Kas                  Rp 5.000.000
```

### Jurnal Otomatis vs Manual

- **Jurnal Otomatis**: dibuat **oleh sistem** ketika Anda menyimpan/menerbitkan dokumen bisnis. Saat Anda menerbitkan faktur, membayar tagihan, atau mencatat biaya, sistem otomatis membuat jurnalnya. Anda tidak perlu tahu Debit/Kredit-nya — sistem yang mengurus. **Inilah cara utama jurnal terbentuk dalam sistem ini.**
- **Jurnal Manual**: Anda buat sendiri lewat menu **Jurnal Manual**, untuk kejadian yang tidak punya dokumen khusus. Contoh: penyusutan aset bulanan, koreksi, setoran modal. Di sini Anda mengetik sendiri sisi Debit dan Kredit, dan sistem **menolak menyimpan jika Debit ≠ Kredit**.

---

## Buku Besar (General Ledger)

Semua jurnal — baik otomatis maupun manual — pada akhirnya mengalir ke satu tabel inti bernama **Buku Besar**. Inilah "buku catatan utama" yang menjadi sumber semua laporan keuangan. Setiap baris di buku besar adalah satu sisi (debit atau kredit) dari sebuah transaksi, lengkap dengan tanggal, akun, dan dokumen sumbernya.

> Secara teknis tabel ini bernama `account_transactions`. Anda tidak mengaksesnya langsung — Anda melihatnya lewat laporan **Buku Besar**, **Neraca Saldo**, dan **Jurnal**.

---

## Piutang Usaha (Accounts Receivable / AR)

**Uang yang BELUM dibayar oleh pelanggan kepada Anda.**

Analogi: Anda sudah mengirim barang ke pelanggan, tapi mereka belum bayar. Itu seperti "bon hutang" dari pelanggan — secara akuntansi itu **aset** Anda (hak menagih uang).

Di sistem ini, piutang terbentuk otomatis saat Anda **menerbitkan faktur**, dan berkurang saat pelanggan **membayar** (Penerimaan Pembayaran) atau Anda memberi **nota kredit**.

---

## Hutang Usaha (Accounts Payable / AP)

**Uang yang BELUM Anda bayar kepada pemasok/vendor.** Kebalikan dari Piutang.

Analogi: pemasok sudah mengirim barang ke Anda, tapi Anda belum bayar. Itu **kewajiban** Anda.

Di sistem ini, hutang terbentuk otomatis saat Anda **membuka tagihan (Bill)**, dan berkurang saat Anda **membayar** (Pembayaran Tagihan) atau menerima **kredit vendor**.

---

## HPP — Harga Pokok Penjualan (Cost of Goods Sold / COGS)

**Biaya langsung untuk menghasilkan/membeli barang yang TERJUAL.**

Analogi: jika Anda menjual laptop seharga Rp 12 juta yang dibeli Rp 9 juta, maka Rp 9 juta itu adalah HPP. Keuntungan kotor Anda Rp 3 juta.

HPP penting karena memisahkan **omzet** (pendapatan) dari **untung sebenarnya**. Pendapatan besar tidak berarti untung besar kalau HPP-nya juga besar.

---

## Ekuitas (Modal)

**Hak pemilik atas perusahaan setelah semua hutang dilunasi.**

Rumus: **Ekuitas = Total Aset − Total Kewajiban**

Analogi: rumah Anda bernilai Rp 1 miliar (aset), tapi KPR-nya masih Rp 400 juta (kewajiban). Maka "kekayaan bersih" Anda di rumah itu Rp 600 juta — itulah ekuitas.

Ekuitas bertambah dari **setoran modal** dan **laba**, berkurang dari **penarikan** dan **rugi**.

---

## Basis Akrual vs Basis Kas

Dua cara berbeda menentukan **kapan** sebuah pendapatan/beban dicatat. Sistem ini mendukung **keduanya** (bisa dipilih di laporan).

| | **Basis Akrual (Accrual)** | **Basis Kas (Cash)** |
| --- | --- | --- |
| Kapan pendapatan dicatat | Saat **faktur dikirim** (barang/jasa diserahkan), walau belum dibayar | Saat **uang benar-benar diterima** |
| Kapan beban dicatat | Saat **tagihan diterima**, walau belum dibayar | Saat **uang benar-benar dibayar** |
| Gambaran | Lebih akurat menggambarkan kinerja bisnis | Lebih akurat menggambarkan posisi kas |

Contoh: Anda kirim faktur Rp 10 juta tanggal 28 Juni, pelanggan bayar 5 Juli.
- **Akrual**: pendapatan Rp 10 juta masuk laporan **Juni**.
- **Kas**: pendapatan Rp 10 juta masuk laporan **Juli**.

> Default sistem (dan semua jurnal otomatis) bekerja secara **akrual**. Basis kas tersedia sebagai opsi tampilan pada laporan tertentu.

---

## Periode Akuntansi

Laporan keuangan selalu mengacu pada **rentang tanggal** (mis. 1–30 Juni 2026) atau **per tanggal tertentu** (mis. posisi keuangan per 30 Juni 2026). Ini disebut periode akuntansi.

- Laporan **Neraca** dilihat **per satu tanggal** (foto kondisi keuangan saat itu).
- Laporan **Laba Rugi** dan **Arus Kas** dilihat **untuk rentang tanggal** (rekaman aktivitas selama periode).

---

## Rekonsiliasi

**Mencocokkan catatan internal sistem dengan kenyataan di rekening bank.**

Analogi: Anda mencatat di buku bahwa saldo BCA Anda Rp 50 juta, tapi mutasi resmi BCA menunjukkan Rp 49,5 juta. Rekonsiliasi adalah proses menemukan dan menjelaskan selisih Rp 500 ribu itu (mungkin ada biaya admin yang belum dicatat).

Di sistem ini, fitur **Tinjau Transaksi (Bank Review)** membantu mencocokkan transaksi bank yang masuk dengan catatan yang sudah ada.

---

## Soft Delete & Jejak Audit (Audit Trail)

- **Soft delete**: saat Anda "menghapus" dokumen, data **tidak benar-benar hilang** dari database — hanya ditandai terhapus. Ini melindungi integritas laporan historis dan memungkinkan pemulihan.
- **Jejak audit**: sistem otomatis mencatat **siapa** membuat/mengubah/menghapus tiap dokumen dan **kapan**. Penting untuk pertanggungjawaban dan pemeriksaan.

---

## Status Dokumen

Banyak dokumen punya **status** yang menggambarkan tahap hidupnya. Yang paling umum:

- **Draft (Konsep)**: dokumen masih disusun, **belum** memengaruhi laporan keuangan (belum ada jurnal). Bisa diedit/dihapus bebas.
- **Terkirim/Terbit/Terbuka (Delivered/Open)**: dokumen sudah final, jurnal sudah terbentuk, sudah memengaruhi laporan.
- **Dibayar Sebagian / Lunas (Partially Paid / Paid)**: untuk faktur & tagihan, menggambarkan progres pembayaran.

> Detail status tiap dokumen dijelaskan di masing-masing dokumen menu (Penjualan, Pembelian, Akuntansi).

---

## Mata Uang & Kurs

Sistem mendukung **multi-mata uang**. Mata uang dasar (base currency) perusahaan adalah **IDR (Rupiah)**.

- Dokumen dalam mata uang asing (mis. USD) menyimpan nilainya dalam mata uang asing tersebut, **tetapi** buku besar selalu mencatat ekuivalen Rupiah-nya (dikalikan kurs).
- **Kurs disimpan per tanggal** sehingga laporan historis tetap akurat — kurs yang dipakai adalah kurs pada tanggal transaksi, bukan kurs hari ini.

---

## Ringkasan Aliran Konsep

```
Dokumen bisnis (faktur/tagihan/pembayaran/biaya)
        │  (Anda buat & terbitkan)
        ▼
Jurnal otomatis (Debit = Kredit)
        │
        ▼
Buku Besar (catatan inti semua transaksi)
        │
        ▼
Laporan keuangan (Neraca, Laba Rugi, Arus Kas, dll.)
```

Setiap istilah di atas akan muncul lagi di dokumen-dokumen berikutnya. Kembalilah ke sini bila ada yang lupa.
