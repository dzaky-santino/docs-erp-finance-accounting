# Penjualan (Sales)

> Menu sidebar: **Penjualan** → Estimasi, Faktur, Penerimaan Penjualan, Nota Kredit, Penerimaan Pembayaran.
> Istilah dasar: lihat **[08-konsep-dasar.md](08-konsep-dasar.md)**.

Modul Penjualan mencatat seluruh siklus "menjual ke pelanggan": dari penawaran harga, penagihan, sampai penerimaan uang. Berikut alur besarnya:

```
Estimasi ──(disetujui, dikonversi)──► Faktur ──(dibayar)──► Penerimaan Pembayaran ──► LUNAS
(penawaran)                            (piutang)             (kas masuk)
   │                                      │                       │
[tanpa jurnal]                  [jurnal: Piutang naik]   [jurnal: Kas naik, Piutang turun]

Alternatif tunai langsung:  Penerimaan Penjualan  →  langsung LUNAS (tanpa piutang)
Koreksi/retur:              Nota Kredit            →  mengurangi piutang / kembalikan uang
```

---

## 1. Estimasi (Quotation / Estimate)

### Apa ini?
Estimasi adalah **surat penawaran harga** kepada calon pelanggan. Ibarat Anda berkata: "Kalau Anda beli ini, harganya sekian." Belum ada transaksi uang — hanya tawaran.

### Kapan digunakan?
Saat pelanggan minta penawaran sebelum memutuskan membeli. Contoh: kontraktor minta estimasi biaya proyek; pelanggan minta quotation untuk 10 unit barang.

### Cara kerja
1. Pilih pelanggan, tanggal estimasi, dan tambahkan baris barang/jasa beserta jumlah & harga.
2. Simpan (status **Draft**) → kirim ke pelanggan (PDF/email).
3. Jika pelanggan setuju, estimasi bisa **dikonversi menjadi Faktur** dengan satu klik.

### Nilai/angka yang tampil
- **Subtotal**: jumlah dari (kuantitas × harga) tiap baris, dikurangi diskon per baris.
- **Diskon**: bisa per baris atau per dokumen (nominal atau persentase).
- **Pajak (PPN)**: tarif pajak × nilai baris (lihat penjelasan pajak di bagian Faktur).
- **Total**: Subtotal − Diskon + Pajak. Sistem menghitung total estimasi dengan rumus yang **sama persis** dengan faktur (sudah termasuk diskon & pajak), agar konsisten saat dikonversi.

### Status dokumen
- **Draft (Konsep)** — masih disusun.
- **Terkirim** — sudah dikirim ke pelanggan.
- **Disetujui / Dikonversi** — pelanggan setuju, biasanya berlanjut menjadi Faktur.

### Efek ke sistem
**TIDAK ADA jurnal / tidak memengaruhi laporan keuangan.** Ini disengaja: penawaran belum tentu jadi, jadi tidak boleh mencemari pendapatan. Estimasi baru "berdampak" setelah dikonversi menjadi Faktur.

---

## 2. Faktur (Invoice)

### Apa ini?
Faktur adalah **tagihan resmi** kepada pelanggan atas barang/jasa yang sudah Anda serahkan. Ini dokumen yang mengikat secara keuangan.

### Bedanya dengan Estimasi
Estimasi = "kira-kira segini" (tanpa dampak akuntansi). Faktur = "ini tagihan Anda" (langsung membentuk **piutang** dan **pendapatan**).

### Kapan piutang terbentuk?
Saat faktur **diterbitkan/dikirim (Deliver)** — bukan saat masih Draft. Pada saat itu sistem otomatis membuat jurnal:

- **Debit** Piutang Usaha (sebesar total faktur termasuk PPN)
- **Kredit** Akun Pendapatan (per baris barang/jasa)
- **Kredit** Hutang PPN (bila ada pajak)

Contoh nyata: pelanggan beli 2 laptop @ Rp 12.000.000, PPN 11%.

| Komponen | Nilai |
| --- | --- |
| Subtotal (2 × 12.000.000) | Rp 24.000.000 |
| PPN 11% (11% × 24.000.000) | Rp 2.640.000 |
| **Total Faktur** | **Rp 26.640.000** |

Jurnal otomatis:
```
Debit  Piutang Usaha          Rp 26.640.000
Kredit Pendapatan Penjualan   Rp 24.000.000
Kredit Hutang PPN             Rp  2.640.000
```
(Debit = Kredit = Rp 26.640.000 ✓)

### Penjelasan pajak (PPN)
- Tarif pajak diambil dari master **Tarif Pajak** (mis. PPN 11%).
- **PPN eksklusif** (umum): pajak **ditambahkan** di atas harga. Harga Rp 24 juta → PPN Rp 2,64 juta → total Rp 26,64 juta.
- **PPN inklusif**: pajak sudah **termasuk** dalam harga. Sistem akan "mengeluarkan" porsi pajaknya dari harga.

### Balance Due (Sisa Tagihan)
Cara menghitung sisa yang masih harus dibayar pelanggan:

> **Sisa Tagihan = Total Faktur − Pembayaran yang sudah diterima − Nota Kredit yang sudah diterapkan**

Contoh: faktur Rp 26.640.000, pelanggan sudah bayar Rp 10.000.000 → sisa tagihan Rp 16.640.000.

### Status dokumen
- **Draft (Konsep)** — belum diterbitkan, belum ada jurnal.
- **Belum Dibayar (Unpaid)** — sudah terbit, belum ada pembayaran.
- **Dibayar Sebagian (Partially Paid)** — sebagian sudah dibayar.
- **Lunas (Paid)** — sisa tagihan = 0.
- **Jatuh Tempo (Overdue)** — melewati tanggal jatuh tempo dan belum lunas.
- **Dihapusbukukan (Written Off)** — sisa tagihan dihapus sebagai piutang tak tertagih (bad debt).

### Fitur tambahan
- **Hapus Buku (Write-off):** bila pelanggan dipastikan tidak akan membayar, sisa piutang dihapus dan dicatat sebagai beban (Debit akun beban write-off, Kredit Piutang).
- **Duplikat:** menyalin faktur sebagai Draft baru.
- **PDF & Email:** kirim faktur ke pelanggan.

### Efek ke sistem
Saat diterbitkan: piutang & pendapatan & hutang PPN bertambah. Saat diedit setelah terbit: jurnal lama dibatalkan dan dibuat ulang otomatis (selama belum ada pembayaran yang membatasi).

---

## 3. Penerimaan Penjualan (Sale Receipt)

### Apa ini?
Dokumen untuk **penjualan tunai langsung** — pelanggan bayar di tempat, tidak ada piutang.

### Bedanya dengan Faktur
- **Faktur** = jual dulu, bayar belakangan → menciptakan **piutang**.
- **Penerimaan Penjualan** = jual & dibayar sekaligus → **langsung masuk kas/bank**, tanpa piutang.

### Kapan dipakai?
Penjualan ritel/tunai. Contoh: pelanggan datang ke toko, beli, langsung bayar cash/transfer.

### Cara kerja & efek ke sistem
Saat ditutup/diterbitkan, jurnal otomatis:
- **Debit** Kas/Bank (akun tujuan uang, sebesar total)
- **Kredit** Akun Pendapatan (per baris)
- **Kredit** Hutang PPN (bila ada pajak)

Bandingkan dengan faktur: bedanya hanya sisi Debit-nya **Kas/Bank** (bukan Piutang), karena uang langsung diterima.

---

## 4. Penerimaan Pembayaran (Payment Receive)

### Apa ini?
Dokumen untuk **mencatat pelunasan faktur** yang sudah terbit sebelumnya. Dipakai khusus untuk faktur kredit (penjualan tidak tunai).

### Cara kerja
1. Pilih pelanggan dan akun tujuan uang (Kas/Bank).
2. Sistem menampilkan daftar faktur pelanggan tersebut yang masih ada sisa.
3. Tentukan berapa yang dibayarkan ke masing-masing faktur.

### Satu pembayaran untuk banyak faktur
Pelanggan bisa membayar beberapa faktur sekaligus dengan satu transfer. Contoh: pelanggan transfer Rp 30 juta untuk melunasi Faktur A (Rp 10 juta) + Faktur B (Rp 20 juta). Sistem membagi pembayaran ke kedua faktur dan memperbarui sisa masing-masing.

### Efek ke sistem
- **Kas/Bank bertambah** (uang masuk).
- **Piutang berkurang** (tagihan lunas/berkurang).
- Status faktur otomatis berubah (Belum Dibayar → Dibayar Sebagian → Lunas).

---

## 5. Nota Kredit (Credit Note)

### Apa ini?
Nota kredit adalah **dokumen koreksi** ke pelanggan, biasanya karena **retur barang**, **kelebihan tagih**, atau **diskon setelah faktur**. Intinya: "Anda berhak atas kredit/pengurangan sebesar sekian."

### Dua cara memakai nota kredit
1. **Diterapkan ke faktur (Apply):** kredit dipakai untuk mengurangi sisa tagihan faktur pelanggan tersebut. Piutang berkurang tanpa uang berpindah.
2. **Pengembalian uang (Refund):** uang benar-benar dikembalikan ke pelanggan dari Kas/Bank.

### Void (Pembatalan permanen)
Nota kredit yang sudah terbit bisa **dibatalkan (Void)** bila dibuat keliru. Efeknya:
- Semua penerapan ke faktur **dibalik** → **piutang faktur pulih** ke kondisi semula.
- Semua refund dibalik → uang yang dikembalikan dicatat ulang.
- **Penting:** pembayaran pelanggan (Payment Receive) **tidak** ikut dibatalkan — kas dari pembayaran tetap valid. Void hanya menyentuh bagian kredit, bukan pembayaran.
- Nota kredit yang sudah Void bersifat **final** — tidak bisa diedit, diterapkan, atau di-void lagi.

> **Kenapa piutang bisa pulih setelah void?** Karena saat nota kredit diterapkan, ia mengurangi piutang faktur. Ketika dibatalkan, pengurangan itu dibalik, jadi tagihan ke pelanggan kembali seperti sebelum nota kredit ada.

### Status dokumen
- **Draft** — belum terbit (belum ada jurnal, cukup dihapus bila salah).
- **Open / Closed** — sudah terbit; Closed bila seluruh kreditnya sudah habis dipakai/dikembalikan.
- **Voided** — dibatalkan permanen.

### Efek ke sistem
Mengurangi pendapatan/piutang (saat apply) atau mengeluarkan kas (saat refund), dengan jurnal seimbang otomatis.

---

## Ringkasan: kapan jurnal terbentuk di modul Penjualan

| Dokumen | Jurnal terbentuk saat | Sisi Debit | Sisi Kredit |
| --- | --- | --- | --- |
| Estimasi | **Tidak pernah** | — | — |
| Faktur | Diterbitkan (Deliver) | Piutang Usaha | Pendapatan + Hutang PPN |
| Penerimaan Penjualan | Ditutup | Kas/Bank | Pendapatan + Hutang PPN |
| Penerimaan Pembayaran | Disimpan | Kas/Bank | Piutang Usaha |
| Nota Kredit (apply) | Diterapkan | Pendapatan/koreksi | Piutang Usaha |
| Nota Kredit (refund) | Refund | Pendapatan/koreksi | Kas/Bank |

Semua jurnal di atas otomatis dan selalu seimbang (Debit = Kredit).
