# Pembelian (Purchases)

> Menu sidebar: **Pembelian** → Tagihan, Kredit Vendor, Pembayaran Keluar. Biaya Tambahan (Landed Cost) diakses dari dalam Tagihan.
> Istilah dasar: lihat **[08-konsep-dasar.md](08-konsep-dasar.md)**.

Modul Pembelian adalah **cermin** dari modul Penjualan, tetapi dari sisi Anda sebagai **pembeli**. Di sini Anda mencatat hutang ke pemasok/vendor dan pelunasannya.

```
Tagihan (Bill) ──(dibayar)──► Pembayaran Tagihan ──► LUNAS
(hutang ke vendor)            (kas keluar)
   │                              │
[jurnal: Hutang naik]   [jurnal: Kas turun, Hutang turun]

Koreksi/retur dari vendor:  Kredit Vendor  →  mengurangi hutang / terima pengembalian
Biaya impor/kirim:          Biaya Tambahan (Landed Cost)  →  menambah nilai persediaan
```

---

## 1. Tagihan (Bill)

### Apa ini?
Tagihan adalah **dokumen hutang Anda kepada pemasok** atas barang/jasa yang sudah Anda terima tetapi belum dibayar. Ini kebalikan dari Faktur: kalau Faktur adalah tagihan yang **Anda kirim**, Bill adalah tagihan yang **Anda terima**.

### Kapan digunakan?
Saat Anda membeli secara kredit. Contoh: membeli 50 unit barang dagangan dari supplier dengan tempo bayar 30 hari.

### Cara kerja
1. Pilih vendor, tanggal tagihan, jatuh tempo, dan baris barang/jasa beserta harga beli.
2. Simpan sebagai **Draft**.
3. **Buka (Open)** tagihan → saat ini hutang resmi tercatat dan jurnal terbentuk.

### Kapan hutang usaha terbentuk?
Saat tagihan **dibuka (Open)** — bukan saat Draft. Jurnal otomatis:

- **Kredit** Hutang Usaha (sebesar total tagihan termasuk PPN)
- **Debit** Akun Biaya/Beban atau Persediaan (per baris barang/jasa)
- **Debit** Hutang PPN / pajak masukan (bila ada pajak)

Contoh: beli barang dagangan Rp 20.000.000, PPN 11% (Rp 2.200.000), total Rp 22.200.000.
```
Debit  Persediaan Barang   Rp 20.000.000
Debit  Hutang PPN (masukan) Rp  2.200.000
Kredit Hutang Usaha         Rp 22.200.000
```
(Debit = Kredit = Rp 22.200.000 ✓)

> Untuk pembelian **barang dagangan**, sisi Debit masuk ke **Persediaan** (aset bertambah). Untuk pembelian **jasa/biaya** (mis. jasa konsultan), sisi Debit masuk ke **akun beban** terkait.

### Sisa Hutang (Balance)
> **Sisa Hutang = Total Tagihan − Pembayaran − Kredit Vendor yang diterapkan**

### Status dokumen
- **Draft (Konsep)** — belum dibuka, belum ada jurnal.
- **Belum Dibayar (Unpaid)** — sudah dibuka, belum dibayar.
- **Dibayar Sebagian (Partially Paid)** — sebagian sudah dibayar.
- **Lunas (Paid)** — sisa hutang = 0.

### Efek ke sistem
Saat dibuka: hutang & persediaan/beban bertambah. Mengubah tagihan setelah dibuka membuat sistem membatalkan & membangun ulang jurnalnya otomatis (selama belum ada pembayaran/kredit yang membatasi).

---

## 2. Pembayaran Keluar / Pembayaran Tagihan (Bill Payment)

### Apa ini?
Dokumen untuk **melunasi tagihan vendor** yang sudah dibuka.

### Cara kerja
1. Pilih vendor dan akun sumber dana (Kas/Bank).
2. Pilih tagihan-tagihan vendor tersebut yang masih ada sisa.
3. Tentukan jumlah bayar per tagihan (bisa melunasi beberapa tagihan sekaligus dengan satu pembayaran).

### Efek ke sistem
- **Kas/Bank berkurang** (uang keluar).
- **Hutang Usaha berkurang**.
- Status tagihan otomatis berubah (Belum Dibayar → Dibayar Sebagian → Lunas).

> Ini kebalikan dari Penerimaan Pembayaran di modul Penjualan: di sana kas masuk & piutang turun; di sini kas keluar & hutang turun.

---

## 3. Kredit Vendor (Vendor Credit)

### Apa ini?
Kredit Vendor adalah **nota kredit dari sisi pembelian** — yaitu ketika **vendor memberi Anda kredit**, biasanya karena Anda meretur barang ke vendor, vendor memberi potongan, atau ada kelebihan tagih.

Ini cermin dari **Nota Kredit** di modul Penjualan, tapi posisinya terbalik: di sini **Anda** yang menerima kredit.

### Dua cara memakai
1. **Diterapkan ke tagihan (Apply):** kredit dipakai mengurangi sisa hutang ke vendor tersebut — hutang berkurang tanpa uang keluar.
2. **Pengembalian uang (Refund):** vendor mengembalikan uang ke Kas/Bank Anda.

### Efek ke sistem
Saat dibuka: **Debit** Hutang Usaha (hutang berkurang), **Kredit** akun biaya/persediaan terkait. Mengurangi kewajiban Anda ke vendor.

### Status dokumen
- **Draft** — belum dibuka.
- **Open / Closed** — sudah dibuka; Closed bila seluruh kreditnya sudah habis dipakai/dikembalikan.

---

## 4. Biaya Tambahan (Landed Cost)

### Apa ini?
Landed Cost adalah **biaya tambahan untuk mendatangkan barang** sampai ke gudang Anda — seperti **ongkos kirim, bea cukai, asuransi pengiriman, biaya bongkar muat**. Biaya ini bukan harga barang itu sendiri, tapi tetap bagian dari "biaya sebenarnya" memiliki barang tersebut.

### Kapan digunakan?
Saat Anda mengimpor/membeli barang dan ada biaya logistik yang harus dibebankan ke nilai barang. Contoh: beli barang Rp 100 juta dari luar negeri, ongkos kirim + bea cukai Rp 10 juta.

### Bagaimana ini memengaruhi nilai persediaan?
Biaya tambahan **dialokasikan (dibagi) ke barang-barang** dalam tagihan, sehingga **nilai per unit persediaan naik**. Pada contoh di atas, total nilai persediaan menjadi Rp 110 juta, bukan Rp 100 juta.

**Mengapa penting?** Agar HPP (Harga Pokok Penjualan) saat barang terjual mencerminkan biaya sebenarnya — termasuk ongkos mendatangkannya. Tanpa ini, laba akan terlihat lebih besar dari kenyataan karena ongkos kirim tidak dihitung.

### Cara kerja
Landed Cost diakses dari dalam dokumen Tagihan: Anda mengalokasikan biaya (dari Bill atau Biaya) ke baris-baris barang pada tagihan tujuan, dengan metode pembagian tertentu (mis. berdasarkan nilai atau kuantitas).

---

## Ringkasan: kapan jurnal terbentuk di modul Pembelian

| Dokumen | Jurnal terbentuk saat | Sisi Debit | Sisi Kredit |
| --- | --- | --- | --- |
| Tagihan (Bill) | Dibuka (Open) | Persediaan/Beban + Pajak | Hutang Usaha |
| Pembayaran Tagihan | Disimpan | Hutang Usaha | Kas/Bank |
| Kredit Vendor (apply) | Dibuka & diterapkan | Hutang Usaha | Biaya/Persediaan |
| Kredit Vendor (refund) | Refund | Kas/Bank | Biaya/Persediaan |
| Biaya Tambahan | Dialokasikan | Persediaan (nilai naik) | Sumber biaya |

Semua jurnal otomatis & selalu seimbang (Debit = Kredit).
