# Dashboard (Beranda)

> Menu: **Beranda** — halaman pertama yang muncul setelah login.
> Lihat istilah dasar di **[08-konsep-dasar.md](08-konsep-dasar.md)**.

---

## Apa ini?

Dashboard adalah **ringkasan kondisi keuangan perusahaan dalam satu layar**. Ibarat dashboard mobil: tanpa membuka mesin, Anda langsung melihat indikator-indikator penting. Tujuannya memberi gambaran cepat tanpa perlu membuka laporan satu per satu.

Halaman ini terdiri dari tiga bagian: **4 kartu ringkasan angka**, **2 grafik**, dan **2 daftar aktivitas terbaru**.

---

## Bagian 1 — Empat Kartu Ringkasan

Empat kotak besar di bagian atas. Masing-masing menampilkan satu angka kunci:

### 1. Piutang (Total Receivables)

- **Artinya:** total uang yang masih harus dibayar oleh **pelanggan** kepada Anda.
- **Diambil dari:** seluruh faktur penjualan yang sudah terbit tapi belum lunas (sisa = nilai faktur − pembayaran − nota kredit).
- **Cara membaca:** angka **naik** = banyak penjualan kredit yang belum tertagih (perlu ditindak agar tidak menumpuk). Angka **turun** = pelanggan rajin membayar / penjualan tunai.

### 2. Hutang (Total Payables)

- **Artinya:** total uang yang masih harus Anda bayar ke **pemasok/vendor**.
- **Diambil dari:** seluruh tagihan (Bill) yang sudah dibuka tapi belum lunas.
- **Cara membaca:** angka **tinggi** = banyak kewajiban yang harus disiapkan kasnya. Pantau agar tidak telat bayar.

### 3. Kas & Bank (Cash & Bank)

- **Artinya:** total saldo uang tunai + seluruh rekening bank perusahaan **saat ini**.
- **Diambil dari:** saldo seluruh akun bertipe **Kas** dan **Bank** di buku besar.
- **Cara membaca:** ini "uang yang benar-benar ada". Bila kecil padahal piutang besar, artinya banyak uang "tertahan" di pelanggan — perlu penagihan.

### 4. Laba Bersih Bulan Ini (Net Income)

- **Artinya:** untung atau rugi periode berjalan.
- **Cara hitung (bahasa awam):** Total Pendapatan − Total Beban.
- **Cara membaca:** angka **positif** = untung; **negatif** = rugi. Ini ringkasan dari Laporan Laba/Rugi.

> Semua nilai diformat mengikuti mata uang dasar perusahaan (Rupiah) dan preferensi format angka di Pengaturan.

---

## Bagian 2 — Dua Grafik

### Grafik Arus Kas (Cash Flow)

- Menampilkan **uang masuk (inflow)** vs **uang keluar (outflow)** dari waktu ke waktu.
- **Gunanya:** melihat tren — apakah belakangan ini lebih banyak uang masuk atau keluar.

### Grafik Pendapatan vs Beban (Revenue vs Expenses)

- Menampilkan **pendapatan** dibanding **beban** per bulan.
- **Gunanya:** melihat apakah pendapatan konsisten di atas beban (sehat) atau sebaliknya.

> Grafik dimuat sedikit setelah halaman tampil (untuk kecepatan). Wajar bila muncul animasi "memuat" sesaat.

---

## Bagian 3 — Aktivitas Terbaru

Dua daftar berdampingan:

### Faktur Terbaru (Recent Invoices)

Beberapa faktur penjualan terakhir, lengkap dengan: nomor faktur, nama pelanggan, tanggal, **status** (mis. Belum Dibayar/Lunas), dan nilainya.

### Tagihan Terbaru (Recent Bills)

Beberapa tagihan pembelian terakhir: nomor tagihan, nama vendor, tanggal, status, dan nilainya.

- **Gunanya:** pintasan cepat untuk memantau dokumen yang baru dibuat tanpa membuka menu Penjualan/Pembelian.
- Bila kosong, muncul pesan "belum ada faktur/tagihan terbaru".

---

## Mengapa dashboard penting untuk bisnis?

Dashboard menjawab pertanyaan **"Bagaimana kondisi perusahaan saya hari ini?"** dalam 5 detik:

- Apakah banyak uang tertahan di pelanggan? → lihat **Piutang**
- Apakah saya punya cukup kas untuk bayar hutang? → bandingkan **Kas & Bank** vs **Hutang**
- Apakah bulan ini saya untung? → lihat **Laba Bersih**
- Apa tren keuangan saya? → lihat **dua grafik**

Untuk analisis mendalam, lanjut ke menu **[Laporan](06-laporan.md)**.
