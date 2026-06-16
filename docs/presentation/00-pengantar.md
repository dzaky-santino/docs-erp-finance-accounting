# Pengantar — Gambaran Umum Sistem

> Dokumen ini menjelaskan sistem secara keseluruhan untuk pembaca yang **awam terhadap finance & accounting**.
> Untuk istilah dasar (Debit, Kredit, Piutang, dll.), lihat **[08-konsep-dasar.md](08-konsep-dasar.md)**.

---

## Sistem ini apa?

Ini adalah **aplikasi keuangan & akuntansi berbasis web** untuk perusahaan. Tujuannya satu: **mencatat semua kejadian uang di perusahaan secara otomatis, rapi, dan dapat dipertanggungjawabkan**, lalu mengubahnya menjadi laporan yang bisa dipakai mengambil keputusan.

Sistem ini menggantikan pencatatan manual (buku tulis, atau spreadsheet yang berantakan) dengan satu tempat terpusat yang:

- Membuat dokumen bisnis (faktur, tagihan, kuitansi, dll.) yang rapi dan bisa di-PDF/email.
- **Otomatis** membuat catatan akuntansi (jurnal) di belakang layar — Anda tidak perlu paham Debit/Kredit.
- Menyusun laporan keuangan standar secara instan kapan saja.
- Menjaga agar pembukuan **selalu seimbang** (tidak mungkin "bocor" diam-diam).

---

## Masalah bisnis apa yang dipecahkan?

| Pertanyaan pemilik bisnis | Bagaimana sistem menjawabnya |
| --- | --- |
| "Siapa saja yang masih berhutang ke saya, dan sudah berapa lama?" | Laporan **Ringkasan Saldo Pelanggan** & **Umur Piutang** |
| "Saya masih harus bayar siapa?" | Laporan **Ringkasan Saldo Vendor** & **Umur Utang** |
| "Bulan ini saya untung atau rugi?" | Laporan **Laba/Rugi** |
| "Total kekayaan & hutang perusahaan saya sekarang berapa?" | Laporan **Neraca** |
| "Uang kas saya ke mana saja perginya?" | Laporan **Arus Kas** |
| "Barang apa yang paling laku? Stok saya tinggal berapa?" | Laporan **Penjualan per Barang** & **Persediaan** |
| "Pajak yang harus saya setor berapa?" | Laporan **Ringkasan Pajak** |
| "Siapa yang mengubah data ini?" | **Jejak audit** otomatis pada setiap dokumen |

---

## Alur kerja umum: dari penjualan sampai laporan

Berikut alur sehari-hari yang paling umum. Perhatikan kapan pencatatan akuntansi (jurnal) terbentuk:

```
PENJUALAN (Sales)

  Estimasi  ──(disetujui)──►  Faktur  ──(dibayar)──►  Penerimaan Pembayaran
  (penawaran)                 (tagihan ke pelanggan)   (pelunasan)
      │                            │                         │
      ▼                            ▼                         ▼
 [TIDAK ada jurnal]      [Jurnal terbentuk:        [Jurnal terbentuk:
                          Piutang bertambah]         Kas naik, Piutang turun]


PEMBELIAN (Purchases)

  Tagihan (Bill)  ──(dibayar)──►  Pembayaran Tagihan
  (hutang ke vendor)              (pelunasan ke vendor)
      │                                 │
      ▼                                 ▼
 [Jurnal terbentuk:            [Jurnal terbentuk:
  Hutang bertambah]             Kas turun, Hutang turun]


SEMUA jurnal di atas mengalir ke:

      Buku Besar  ──►  LAPORAN (Neraca, Laba Rugi, Arus Kas, dll.)
```

> **Poin penting untuk awam:** *Estimasi* (penawaran harga) sengaja **tidak** memengaruhi laporan keuangan. Uang baru "diakui" saat menjadi *Faktur*. Ini mencegah laporan tercemar oleh penawaran yang belum tentu jadi.

---

## Modul utama (peta menu besar)

Sistem dikelompokkan menjadi beberapa area besar di sidebar kiri:

| Modul | Untuk apa | Dokumen rinci |
| --- | --- | --- |
| **Beranda (Dashboard)** | Ringkasan kondisi keuangan sekilas | [01-dashboard.md](01-dashboard.md) |
| **Penjualan (Sales)** | Estimasi, Faktur, Penerimaan Penjualan, Nota Kredit, Penerimaan Pembayaran | [02-penjualan.md](02-penjualan.md) |
| **Pembelian (Purchases)** | Tagihan, Pembayaran Tagihan, Kredit Vendor, Biaya Tambahan | [03-pembelian.md](03-pembelian.md) |
| **Akuntansi** | Bagan Akun, Jurnal Manual, Biaya, Penyesuaian Persediaan, Penguncian Transaksi | [04-akuntansi.md](04-akuntansi.md) |
| **Perbankan (Banking)** | Akun Bank/Kas, Uang Masuk/Keluar, Tinjau Transaksi (rekonsiliasi) | [05-perbankan.md](05-perbankan.md) |
| **Laporan (Reports)** | Semua laporan keuangan & operasional | [06-laporan.md](06-laporan.md) |
| **Pengaturan (Preferences)** | Pengguna, Kontak, Barang, Pajak, Mata Uang, Preferensi | [07-pengaturan.md](07-pengaturan.md) |

---

## Siapa saja penggunanya? (Peran / Role)

Sistem membatasi apa yang bisa dilihat & dikerjakan tiap orang berdasarkan **peran (role)** mereka. Ini menjaga keamanan: kasir tidak bisa mengubah bagan akun, sales tidak bisa melihat laporan keuangan rahasia, dst.

| Peran | Bisa apa | Tidak bisa |
| --- | --- | --- |
| **Super Admin / Admin** | Semua akses penuh, termasuk Pengaturan sistem | — |
| **Manajer Keuangan** (finance-manager) | Penuh: Penjualan, Pembelian, Akuntansi, Perbankan, Laporan, dan data master (kontak/barang/proyek) | Pengaturan sistem (kelola pengguna, organisasi, penguncian transaksi) |
| **Akuntan** (accountant) | Akuntansi penuh, Perbankan, semua Laporan, Penguncian Transaksi. Penjualan & Pembelian hanya **lihat** | Membuat/mengubah dokumen Penjualan & Pembelian |
| **Staf Penjualan** (sales-staff) | Seluruh dokumen Penjualan + tambah kontak & barang | Pembelian, Akuntansi, Perbankan, Laporan, Pengaturan |
| **Staf Pembelian** (purchasing-staff) | Seluruh dokumen Pembelian + tambah kontak & barang | Penjualan, Akuntansi, Perbankan, Laporan, Pengaturan |
| **Kasir** (cashier) | Mencatat penerimaan pembayaran, penerimaan penjualan, pembayaran tagihan, kas masuk/keluar; lihat faktur/tagihan/kontak | Membuat faktur/tagihan/estimasi, bagan akun, jurnal, Pengaturan |
| **Pembaca Laporan** (report-viewer) | **Hanya melihat** seluruh modul + semua laporan + ekspor | Membuat/mengubah/menghapus apa pun, Pengaturan |

> Peran tambahan bisa dibuat sendiri lewat menu Pengaturan → Pengguna/Peran. Setiap akses diatur per "izin" granular (mis. "boleh membuat faktur", "boleh melihat laporan neraca").

---

## Prinsip yang dijaga sistem (jaminan kualitas data)

1. **Selalu seimbang** — setiap transaksi otomatis menjaga Debit = Kredit. Mustahil pembukuan timpang tanpa ketahuan.
2. **Tidak ada penghapusan diam-diam** — data dihapus secara "soft delete" (ditandai, bukan dimusnahkan), dan ada jejak audit siapa melakukan apa.
3. **Periode bisa dikunci** — setelah tutup buku, periode lama bisa dikunci agar tidak ada yang mengubah angka historis.
4. **Hak akses berlapis** — tiap peran hanya melihat/mengerjakan yang menjadi tanggung jawabnya.

---

## Cara membaca dokumen presentasi ini

1. Mulai dari **[08-konsep-dasar.md](08-konsep-dasar.md)** bila Anda benar-benar baru.
2. Lanjut ke modul yang Anda pakai sehari-hari (Penjualan/Pembelian).
3. **[06-laporan.md](06-laporan.md)** adalah dokumen terpenting bagi pemilik bisnis — di sana semua angka laporan dijelaskan dari mana asalnya.

Selamat membaca.
