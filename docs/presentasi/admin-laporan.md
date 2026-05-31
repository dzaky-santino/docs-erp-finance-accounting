# Presentasi Admin Laporan

Dokumen ini adalah panduan presentasi untuk super admin atau admin ketika menjelaskan menu Laporan. Fokusnya adalah fungsi bisnis, cara membaca angka untuk orang awam, filter yang benar-benar tersedia di UI, route export/PDF, status basis Akrual/Kas, batas rentang tanggal, dan catatan fitur yang tidak boleh diklaim bila belum terverifikasi dari kode.

Baca silang sebelum demo:

- Preferensi sistem: [admin-preferensi.md](admin-preferensi.md)
- Keuangan dan Akuntansi: [admin-keuangan.md](admin-keuangan.md)
- Perbankan: [admin-perbankan.md](admin-perbankan.md)
- Biaya: [admin-biaya.md](admin-biaya.md)
- Penjualan dan piutang: [admin-penjualan.md](admin-penjualan.md)
- Pembelian dan utang: [admin-pembelian.md](admin-pembelian.md)
- Barang/Jasa dan persediaan: [admin-barang-jasa.md](admin-barang-jasa.md)
- Proyek: [admin-proyek.md](admin-proyek.md)
- Kontak customer/vendor: [admin-kontak.md](admin-kontak.md)

## 1. Tujuan Dokumen

Dokumen ini dipakai agar presenter dapat menjelaskan menu Laporan tanpa harus membaca route, service, dan komponen React saat sesi demo. Target audiensnya adalah admin finance, accounting, owner, auditor internal, dan operator yang perlu memahami hubungan transaksi dengan angka laporan.

Tujuan praktisnya:

| Tujuan | Hasil yang diharapkan |
| --- | --- |
| Menjelaskan daftar laporan aktual | Presenter hanya menyebut laporan yang memang muncul di halaman `/reports`. |
| Menjelaskan arti angka | Admin dapat membedakan posisi, performa, arus kas, aging, persediaan, dan transaksi detail. |
| Menjelaskan filter | Presenter tahu kapan memakai tanggal per posisi, rentang tanggal, akun, project, kontak, item, atau reference. |
| Menjelaskan export/PDF | Presenter tahu laporan mana yang punya CSV/XLSX/PDF dan mana yang belum punya PDF. |
| Menjaga klaim basis laporan | Presenter hanya mendemokan basis Akrual/Kas pada laporan yang mengeksposnya di UI web. |
| Menghindari demo yang terlalu berat | Presenter tahu report row-level yang memakai guard rentang tanggal. |

Audit teknis phase ini membaca halaman React `resources/js/pages/reports/*`, route web `routes/web.php`, route API `routes/api.php`, `ReportService`, `ReportExportService`, `ReportDateRangeGuard`, `ReportPermissions`, template PDF di `resources/views/reports/pdf`, dan dokumen presentasi admin lain. Hasil route yang diaudit: 18 kartu laporan di halaman `/reports`, 53 named route web `reports.*`, dan 60 path `/reports` bila 7 endpoint API laporan ikut dihitung.

## 2. Gambaran Umum Menu Laporan

Menu Laporan berada di halaman `/reports`. Halaman index menampilkan kartu laporan sesuai permission user. Grup route web memakai `ReportPermissions::anyMiddleware()`, sedangkan setiap halaman laporan memakai permission granular masing-masing.

Ringkasan laporan aktual:

| No | Laporan di UI | Route halaman | Permission granular | Export | PDF |
| --- | --- | --- | --- | --- | --- |
| 1 | Neraca | `/reports/balance-sheet` | `report-balance-sheet.view` | CSV/XLSX | Ya |
| 2 | Laporan Laba Rugi | `/reports/income-statement` | `report-income-statement.view` | CSV/XLSX | Ya |
| 3 | Profitabilitas Proyek | `/reports/project-profitability` | `report-project-profitability.view` | CSV/XLSX | Ya |
| 4 | Neraca Saldo | `/reports/trial-balance` | `report-trial-balance.view` | CSV/XLSX | Ya |
| 5 | Laporan Arus Kas | `/reports/cash-flow` | `report-cash-flow.view` | CSV/XLSX | Ya |
| 6 | Umur Piutang | `/reports/receivables-aging` | `report-receivables-aging.view` | CSV/XLSX | Ya |
| 7 | Umur Utang | `/reports/payables-aging` | `report-payables-aging.view` | CSV/XLSX | Ya |
| 8 | Ringkasan Pajak | `/reports/tax-summary` | `report-tax-summary.view` | CSV/XLSX | Ya |
| 9 | Buku Besar | `/reports/general-ledger` | `report-general-ledger.view` | CSV/XLSX | Ya |
| 10 | Penjualan per Barang/Jasa | `/reports/sales-by-items` | `report-sales-by-items.view` | CSV/XLSX | Ya |
| 11 | Pembelian per Barang/Jasa | `/reports/purchases-by-items` | `report-purchases-by-items.view` | CSV/XLSX | Ya |
| 12 | Ringkasan Saldo Pelanggan | `/reports/customer-balance-summary` | `report-customer-balance-summary.view` | CSV/XLSX | Ya |
| 13 | Ringkasan Saldo Vendor | `/reports/vendor-balance-summary` | `report-vendor-balance-summary.view` | CSV/XLSX | Ya |
| 14 | Lembar Jurnal | `/reports/journal-sheet` | `report-journal-sheet.view` | CSV/XLSX | Ya |
| 15 | Laporan Valuasi Persediaan | `/reports/inventory-valuation-sheet` | `report-inventory-valuation-sheet.view` | CSV/XLSX | Ya |
| 16 | Detail Barang Persediaan | `/reports/inventory-item-details` | `report-inventory-item-details.view` | CSV/XLSX | Tidak terverifikasi |
| 17 | Transaksi per Kontak | `/reports/transactions-by-contact` | `report-transactions-by-contact.view` | CSV/XLSX | Ya |
| 18 | Transaksi per Referensi | `/reports/transactions-by-reference` | `report-transactions-by-reference.view` | CSV/XLSX | Tidak terverifikasi |

Route API laporan yang terverifikasi hanya mencakup 7 laporan inti: balance sheet, income statement, trial balance, cash flow, receivables aging, payables aging, dan tax summary. API ini berbeda dari halaman web presentasi karena tidak mencakup semua kartu UI dan tidak mengekspos semua filter UI web.

## 3. Prinsip Membaca Laporan Untuk Orang Awam

Gunakan prinsip sederhana ini saat menjelaskan angka:

| Prinsip | Penjelasan sederhana |
| --- | --- |
| Per tanggal berbeda dari periode | Neraca, aging, saldo pelanggan/vendor, dan valuasi persediaan memakai posisi pada satu tanggal. Laba Rugi, Arus Kas, Buku Besar, dan laporan transaksi memakai rentang tanggal. |
| Draft biasanya belum masuk laporan akuntansi | Laporan yang membaca GL atau dokumen published/opened/delivered tidak akan menampilkan transaksi yang masih draft. |
| Angka bisa berubah setelah pembayaran | Aging, saldo pelanggan/vendor, cash flow, dan laporan berbasis kas berubah ketika invoice/bill dibayar atau dikreditkan. |
| Laba bukan kas | Laba Rugi menjelaskan performa. Arus Kas menjelaskan uang pada akun Cash/Bank. Keduanya bisa berbeda. |
| Buku Besar adalah sumber audit | Jika angka ringkasan dipertanyakan, buka Buku Besar atau Lembar Jurnal untuk melihat transaksi penyusunnya. |
| Export membantu review, bukan mengganti audit | CSV/XLSX/PDF memudahkan pembagian data, tetapi filter, tanggal, dan status transaksi tetap harus dicek. |
| Basis laporan tidak universal | UI web yang diaudit hanya mengekspos basis Akrual/Kas pada Laporan Laba Rugi dan Profitabilitas Proyek. |

Kalimat demo yang aman: "Laporan ini membaca transaksi yang sudah masuk ke proses akuntansi. Kalau angka belum muncul, pertama cek status transaksi, akun yang dipakai, tanggal dokumen, dan filter laporan."

## 4. Urutan Penggunaan Yang Disarankan

Urutan ini membantu admin menyiapkan data sebelum membuka laporan:

1. Isi Preferensi General: nama organisasi, base currency, timezone, format tanggal, dan tahun fiskal.
2. Siapkan Bagan Akun: Cash/Bank, Piutang, Utang, Pajak, Pendapatan, HPP, Beban, Persediaan, dan Modal.
3. Pastikan akun wajib modul tersedia: Accounts Receivable, Accounts Payable, dan Tax Payable sesuai modul.
4. Buat atau impor master data: kontak, item, project, gudang, dan tax rate.
5. Buat transaksi contoh: invoice delivered, bill opened, payment receive, bill payment, expense published, manual journal published, dan transaksi bank bila perlu.
6. Buka laporan ringkasan manajemen: Laba Rugi, Neraca, dan Arus Kas.
7. Buka laporan kontrol accounting: Neraca Saldo, Buku Besar, dan Lembar Jurnal.
8. Buka laporan AR/AP: Umur Piutang, Umur Utang, Ringkasan Saldo Pelanggan, dan Ringkasan Saldo Vendor.
9. Buka laporan item/persediaan: Sales by Items, Purchases by Items, Valuasi Persediaan, dan Detail Barang Persediaan.
10. Buka laporan pencarian transaksi: Transaksi per Kontak dan Transaksi per Referensi.
11. Uji export CSV/XLSX dan PDF sebelum presentasi formal.

Untuk demo pertama, gunakan basis Akrual sebagai default. Gunakan basis Kas hanya pada Laba Rugi dan Profitabilitas Proyek, dan jelaskan bahwa report lain belum mengekspos toggle tersebut di UI web.

## 5. Daftar Laporan Dalam Sistem

Daftar berikut membantu presenter memilih laporan sesuai pertanyaan bisnis:

| Kategori | Laporan | Pertanyaan yang dijawab | Filter utama |
| --- | --- | --- | --- |
| Posisi keuangan | Neraca | Aset, kewajiban, dan ekuitas pada tanggal tertentu. | Tanggal per posisi. |
| Performa | Laporan Laba Rugi | Perusahaan untung atau rugi selama periode. | Dari tanggal, sampai tanggal, basis. |
| Performa project | Profitabilitas Proyek | Project mana yang memberi profit atau rugi. | Dari tanggal, sampai tanggal, project opsional, basis. |
| Kontrol akuntansi | Neraca Saldo | Apakah total debit dan credit seimbang selama periode. | Dari tanggal, sampai tanggal. |
| Kas dan bank | Laporan Arus Kas | Uang masuk/keluar dari akun Cash/Bank selama periode. | Dari tanggal, sampai tanggal. |
| Piutang | Umur Piutang | Piutang pelanggan mana yang belum dibayar dan sudah berapa lama jatuh tempo. | Tanggal per posisi. |
| Utang | Umur Utang | Tagihan vendor mana yang belum dibayar dan sudah berapa lama jatuh tempo. | Tanggal per posisi. |
| Pajak | Ringkasan Pajak | Pajak terpungut, pajak dibayar, dan net tax payable. | Dari tanggal, sampai tanggal. |
| Audit transaksi | Buku Besar | Mutasi debit/credit dan saldo berjalan per akun. | Dari tanggal, sampai tanggal, akun opsional, project opsional. |
| Penjualan item | Penjualan per Barang/Jasa | Item mana yang paling banyak atau besar nilainya dijual. | Dari tanggal, sampai tanggal. |
| Pembelian item | Pembelian per Barang/Jasa | Item mana yang paling banyak atau besar nilainya dibeli. | Dari tanggal, sampai tanggal. |
| Saldo pelanggan | Ringkasan Saldo Pelanggan | Total invoice, pembayaran, credit, dan outstanding per pelanggan. | Tanggal per posisi. |
| Saldo vendor | Ringkasan Saldo Vendor | Total bill, pembayaran, vendor credit, dan outstanding per vendor. | Tanggal per posisi. |
| Jurnal | Lembar Jurnal | Pasangan debit-credit per transaksi/referensi. | Dari tanggal, sampai tanggal. |
| Persediaan | Laporan Valuasi Persediaan | Kuantitas stok dan nilai persediaan per item. | Tanggal per posisi. |
| Persediaan detail | Detail Barang Persediaan | Pergerakan masuk/keluar dan saldo berjalan untuk satu item. | Item, dari tanggal, sampai tanggal. |
| Pencarian kontak | Transaksi per Kontak | Dokumen dan pembayaran yang terkait satu kontak. | Kontak, dari tanggal, sampai tanggal. |
| Pencarian referensi | Transaksi per Referensi | Baris GL yang cocok dengan nomor/tulisan referensi. | Dari tanggal, sampai tanggal, reference opsional, akun opsional. |

## 6. Ringkasan Filter, Export, PDF, dan Basis

Matriks ini menjadi pegangan cepat saat presentasi:

| Laporan | Filter UI web | Export | PDF | Basis UI web | Guard rentang |
| --- | --- | --- | --- | --- | --- |
| Neraca | `date` | CSV/XLSX | Ya | Tidak | Tidak |
| Laporan Laba Rugi | `from_date`, `to_date`, `basis` | CSV/XLSX | Ya | Akrual/Kas | Tidak |
| Profitabilitas Proyek | `from_date`, `to_date`, `project_id`, `basis` | CSV/XLSX | Ya | Akrual/Kas | Ya |
| Neraca Saldo | `from_date`, `to_date` | CSV/XLSX | Ya | Tidak | Tidak |
| Laporan Arus Kas | `from_date`, `to_date` | CSV/XLSX | Ya | Tidak | Tidak |
| Umur Piutang | `date` | CSV/XLSX | Ya | Tidak | Tidak |
| Umur Utang | `date` | CSV/XLSX | Ya | Tidak | Tidak |
| Ringkasan Pajak | `from_date`, `to_date` | CSV/XLSX | Ya | Tidak | Tidak |
| Buku Besar | `from_date`, `to_date`, `account_id`, `project_id` | CSV/XLSX | Ya | Tidak | Ya |
| Penjualan per Barang/Jasa | `from_date`, `to_date` | CSV/XLSX | Ya | Tidak | Tidak |
| Pembelian per Barang/Jasa | `from_date`, `to_date` | CSV/XLSX | Ya | Tidak | Tidak |
| Ringkasan Saldo Pelanggan | `as_of_date` | CSV/XLSX | Ya | Tidak | Tidak |
| Ringkasan Saldo Vendor | `as_of_date` | CSV/XLSX | Ya | Tidak | Tidak |
| Lembar Jurnal | `from_date`, `to_date` | CSV/XLSX | Ya | Tidak | Ya |
| Laporan Valuasi Persediaan | `as_of_date` | CSV/XLSX | Ya | Tidak | Tidak |
| Detail Barang Persediaan | `item_id`, `from_date`, `to_date` | CSV/XLSX | Tidak terverifikasi | Tidak | Tidak |
| Transaksi per Kontak | `contact_id`, `from_date`, `to_date` | CSV/XLSX | Ya | Tidak | Ya |
| Transaksi per Referensi | `from_date`, `to_date`, `reference`, `account_id` | CSV/XLSX | Tidak terverifikasi | Tidak | Ya |

Status basis yang harus dibawakan:

| Area | Status |
| --- | --- |
| Laba Rugi | UI, route web, export, dan PDF menerima `basis` Akrual/Kas. Jika kosong, route web default ke Akrual. |
| Profitabilitas Proyek | UI, route web, export, dan PDF menerima `basis` Akrual/Kas. Jika kosong, route web default ke Akrual. |
| Neraca | Service internal menerima basis, tetapi route web/API/export/PDF yang diaudit tidak mengekspos basis. Jangan demo basis Kas dari UI. |
| Neraca Saldo | Service internal menerima basis, tetapi route web/API/export/PDF yang diaudit tidak mengekspos basis. Jangan demo basis Kas dari UI. |
| Ringkasan Pajak | Tidak ada basis di UI/route/export/PDF yang diaudit. Cash-basis tax summary tetap design-only sesuai catatan proyek. |
| API laporan | Controller API 7 laporan inti tidak menerima basis sebagai filter presentasi. |

## 7. Laporan Laba Rugi

Laporan Laba Rugi menjawab pertanyaan: "selama periode ini perusahaan untung atau rugi?" Laporan ini cocok untuk manajemen, owner, dan finance lead.

Fungsi bisnis:

| Fungsi | Penjelasan |
| --- | --- |
| Mengukur performa periode | Menampilkan pendapatan, biaya, dan laba/rugi bersih. |
| Membandingkan periode | Admin dapat mengganti rentang tanggal, misalnya bulan ini atau tahun berjalan. |
| Melihat dampak transaksi | Invoice delivered, expense published, bill opened, dan jurnal P&L dapat memengaruhi angka akrual. |
| Membandingkan Akrual/Kas | UI web menyediakan pilihan basis Akrual dan Kas. |

Filter:

| Filter | Status | Cara menjelaskan |
| --- | --- | --- |
| Dari tanggal | Wajib untuk menghasilkan laporan. | Awal periode laporan. |
| Sampai tanggal | Wajib dan harus sama atau setelah dari tanggal. | Akhir periode laporan. |
| Basis | `accrual` atau `cash`. | Akrual mengikuti pengakuan transaksi; Kas mengikuti sumber cash-basis yang didukung. |

Sumber data:

- Basis Akrual membaca `account_transactions` untuk akun ber-root income dan expense dalam rentang tanggal.
- Basis Kas memakai ledger cash-basis internal untuk baris profit and loss yang sudah didukung.
- Total income dikurangi total expenses menghasilkan net income atau net loss.

Cara membaca:

| Bagian | Arti |
| --- | --- |
| Income | Pendapatan yang diakui dalam periode. |
| Expenses | Beban/HPP/biaya yang diakui dalam periode. |
| Net Income | Income lebih besar dari expenses. |
| Net Loss | Expenses lebih besar dari income. |

Contoh alur demo:

1. Buka `/reports/income-statement`.
2. Pilih periode bulan berjalan.
3. Generate laporan dengan basis Akrual.
4. Jelaskan total income, total expenses, dan net income.
5. Ubah basis ke Kas bila data pembayaran tersedia.
6. Jelaskan bahwa perbedaan Akrual dan Kas normal bila transaksi belum dibayar.
7. Export PDF atau CSV untuk bukti pembagian laporan.

Catatan demo:

- Jangan memakai Laba Rugi untuk menjelaskan saldo kas. Untuk itu buka Arus Kas atau Buku Besar akun bank.
- Jika angka pendapatan belum muncul, cek invoice sudah delivered dan akun pendapatan benar.
- Jika angka biaya belum muncul, cek expense sudah published, bill sudah opened, atau jurnal manual sudah published.

## 8. Neraca

Neraca menjawab pertanyaan: "pada tanggal ini, posisi aset, kewajiban, dan ekuitas seperti apa?" Laporan ini adalah laporan posisi, bukan laporan performa periode.

Fungsi bisnis:

| Fungsi | Penjelasan |
| --- | --- |
| Melihat posisi aset | Kas, bank, piutang, persediaan, fixed asset, dan aset lain. |
| Melihat kewajiban | Utang usaha, pajak terutang, dan liabilitas lain. |
| Melihat ekuitas | Modal, retained earnings, dan net income yang dihitung. |
| Kontrol keseimbangan | Total aset idealnya sejalan dengan total liabilitas plus ekuitas. |

Filter:

| Filter | Status | Cara menjelaskan |
| --- | --- | --- |
| Tanggal | Wajib untuk view/export/PDF. | Tanggal posisi laporan. |
| Basis | Tidak tersedia di UI web/export/PDF. | Jangan tawarkan basis Kas untuk Neraca saat demo. |

Sumber data:

- Route web membaca `ReportService::balanceSheet($date)` tanpa parameter basis.
- Service menghitung saldo akun dari `account_transactions` sampai tanggal laporan.
- Akun dikelompokkan dari root type asset, liability, dan equity.
- Net income dihitung sampai tanggal laporan dan ditambahkan ke ekuitas.

Cara membaca:

| Bagian | Arti |
| --- | --- |
| Assets | Sumber daya yang dimiliki atau dikendalikan perusahaan. |
| Liabilities | Kewajiban yang harus dibayar. |
| Equity | Hak pemilik setelah kewajiban. |
| Liabilities + Equity | Sisi pendanaan aset. |

Contoh alur demo:

1. Buka Neraca.
2. Pilih tanggal akhir bulan.
3. Tunjukkan total assets.
4. Tunjukkan total liabilities dan total equity.
5. Jelaskan net income sebagai bagian ekuitas.
6. Export PDF bila perlu laporan posisi formal.

Catatan demo:

- Jangan mendemokan query `basis=cash` dari address bar; route UI/export/PDF tidak mengeksposnya.
- Jika Neraca tampak kosong, cek apakah transaksi sudah dipublish/deliver/open dan memiliki baris GL.
- Jika nilai tidak sesuai ekspektasi, buka Buku Besar untuk akun terkait.

## 9. Neraca Saldo

Neraca Saldo menjawab pertanyaan: "apakah debit dan credit selama periode ini seimbang?" Laporan ini lebih cocok untuk accounting daripada owner non-akuntansi.

Fungsi bisnis:

| Fungsi | Penjelasan |
| --- | --- |
| Kontrol double-entry | Total debit dan total credit harus seimbang. |
| Cek mutasi per akun | Menunjukkan debit/credit per akun dalam periode. |
| Persiapan closing | Membantu review sebelum laporan final dibagikan. |
| Deteksi anomali | Selisih debit/credit menunjukkan ada masalah posting atau data. |

Filter:

| Filter | Status | Cara menjelaskan |
| --- | --- | --- |
| Dari tanggal | Wajib untuk export/PDF dan untuk menampilkan laporan. | Awal periode mutasi. |
| Sampai tanggal | Wajib dan tidak boleh sebelum dari tanggal. | Akhir periode mutasi. |
| Basis | Tidak tersedia di UI web/export/PDF. | Jangan tawarkan basis Kas untuk Neraca Saldo saat demo. |

Sumber data:

- Route web membaca `ReportService::trialBalance($fromDate, $toDate)` tanpa parameter basis.
- Service mengelompokkan `account_transactions` per akun dan menjumlahkan debit/credit.
- UI menampilkan pesan jika total debit dan total credit tidak sama.

Cara membaca:

| Kolom | Arti |
| --- | --- |
| Code | Kode akun dari COA. |
| Account Name | Nama akun. |
| Type | AccountType dari COA. |
| Debit | Total debit periode. |
| Credit | Total credit periode. |

Contoh alur demo:

1. Buka Neraca Saldo.
2. Pilih periode bulan berjalan.
3. Generate laporan.
4. Tunjukkan total debit dan total credit.
5. Jika balance, lanjut ke Laba Rugi/Neraca.
6. Jika tidak balance, jelaskan perlu audit Buku Besar/Lembar Jurnal.

Catatan demo:

- Neraca Saldo tidak menjelaskan untung/rugi. Gunakan Laba Rugi untuk itu.
- Neraca Saldo tidak menjelaskan saldo akhir bank secara rinci. Gunakan Buku Besar akun bank.
- Basis Kas internal tidak diekspos pada UI web/export/PDF.

## 10. Buku Besar

Buku Besar adalah laporan audit utama. Ia menjawab pertanyaan: "angka ini berasal dari transaksi apa?"

Fungsi bisnis:

| Fungsi | Penjelasan |
| --- | --- |
| Melihat saldo awal | Saldo akun sebelum periode laporan. |
| Melihat mutasi | Semua baris debit/credit dalam periode. |
| Melihat saldo berjalan | Saldo setelah setiap transaksi. |
| Audit per akun/project | Filter akun dan project membantu mempersempit pencarian. |

Filter:

| Filter | Status | Cara menjelaskan |
| --- | --- | --- |
| Dari tanggal | Wajib untuk menghasilkan laporan. | Awal periode audit. |
| Sampai tanggal | Wajib dan tidak boleh sebelum dari tanggal. | Akhir periode audit. |
| Akun | Opsional di UI. | Kosong berarti semua akun. |
| Project | Opsional di UI. | Kosong berarti semua project. |

Sumber data:

- `account_transactions` menjadi sumber utama.
- Saldo awal dihitung dari transaksi sebelum `from_date`.
- Mutasi periode dihitung dari transaksi antara `from_date` dan `to_date`.
- Jika `project_id` dipilih, saldo awal dan mutasi dipersempit ke project tersebut.

Cara membaca:

| Bagian | Arti |
| --- | --- |
| Opening Balance | Saldo akun sebelum tanggal awal. |
| Debit/Credit | Mutasi sesuai sisi jurnal. |
| Running Balance | Saldo akun setelah transaksi. |
| Closing Balance | Saldo akun akhir periode. |

Guard rentang:

| Mode | Batas default |
| --- | --- |
| View | 730 hari, dapat diubah via `REPORT_VIEW_MAX_DAYS`. |
| Export/PDF | 366 hari, dapat diubah via `REPORT_EXPORT_MAX_DAYS`. |

Contoh alur demo:

1. Buka Buku Besar.
2. Pilih periode bulan berjalan.
3. Pilih akun `Bank BCA Operasional` atau akun pendapatan.
4. Generate laporan.
5. Tunjukkan opening balance, mutasi, dan closing balance.
6. Export CSV untuk rekonsiliasi spreadsheet.

Catatan demo:

- Buku Besar dapat berisi banyak baris. Gunakan filter akun/project agar cepat.
- Jika user mendapat error rentang terlalu besar, perkecil periode.
- Buku Besar membaca GL; jika transaksi tidak membuat GL, tidak muncul di sini.

## 11. Arus Kas

Laporan Arus Kas menjawab pertanyaan: "selama periode ini uang di akun Cash/Bank bergerak karena apa?"

Fungsi bisnis:

| Fungsi | Penjelasan |
| --- | --- |
| Melihat aktivitas operasional | Pergerakan kas dari transaksi operasional seperti pembayaran, bill, expense, dan sale receipt. |
| Melihat aktivitas investasi | Pergerakan yang diklasifikasikan sebagai inventory adjustment atau landed cost. |
| Melihat aktivitas pendanaan | Pergerakan yang diklasifikasikan dari manual journal atau cashflow transaction. |
| Menjelaskan selisih dengan Laba Rugi | Laba akrual belum tentu sudah menjadi kas. |

Filter:

| Filter | Status | Cara menjelaskan |
| --- | --- | --- |
| Dari tanggal | Wajib untuk export/PDF dan menghasilkan laporan. | Awal periode arus kas. |
| Sampai tanggal | Wajib dan tidak boleh sebelum dari tanggal. | Akhir periode arus kas. |
| Basis | Tidak tersedia. | Laporan ini memang fokus akun Cash/Bank. |

Sumber data:

- Service membaca `account_transactions` yang account type-nya `cash` atau `bank`.
- Mutasi dikelompokkan berdasarkan `transaction_type`.
- Route web menampilkan operating, investing, financing, net change, opening balance, dan closing balance.

Cara membaca:

| Bagian | Arti |
| --- | --- |
| Operating Activities | Aktivitas kas operasional. |
| Investing Activities | Aktivitas kas yang diklasifikasikan sebagai investasi. |
| Financing Activities | Aktivitas kas pendanaan. |
| Net Change in Cash | Perubahan kas bersih periode. |

Catatan penting:

- Route web saat audit mengisi `opening_balance` dengan 0 dan `closing_balance` sebagai net change periode. Jangan presentasikan angka opening/closing di laporan ini sebagai saldo kas historis lengkap.
- Untuk saldo bank historis per akun, gunakan Buku Besar akun bank atau halaman Perbankan.
- Jika transaksi bank belum diposting ke GL, Arus Kas tidak akan berubah.

## 12. Ringkasan Pajak

Ringkasan Pajak menjawab pertanyaan: "berapa pajak yang dipungut, pajak yang dibayar, dan saldo pajak bersih selama periode?"

Fungsi bisnis:

| Fungsi | Penjelasan |
| --- | --- |
| Rekonsiliasi pajak internal | Membantu finance melihat posisi pajak dari transaksi. |
| Membandingkan pajak keluaran dan masukan | Tax collected dibandingkan dengan tax paid. |
| Menentukan net tax | Net positif berarti tax payable; net negatif dibaca sebagai potensi refund/lebih bayar internal. |

Filter:

| Filter | Status | Cara menjelaskan |
| --- | --- | --- |
| Dari tanggal | Wajib untuk export/PDF dan menghasilkan laporan. | Awal periode pajak. |
| Sampai tanggal | Wajib dan tidak boleh sebelum dari tanggal. | Akhir periode pajak. |
| Basis | Tidak tersedia di UI web/export/PDF/API. | Jangan demo cash-basis Tax Summary. |

Sumber data:

- Service membaca `account_transactions` yang akun terkait bertipe `TaxPayable`.
- Tax collected dihitung dari transaction type penjualan seperti `SaleInvoice`, `SaleReceipt`, dan `CreditNote`.
- Tax paid dihitung dari transaction type pembelian/biaya seperti `Bill`, `VendorCredit`, dan `Expense`.

Cara membaca:

| Bagian | Arti |
| --- | --- |
| Tax Collected | Pajak yang dipungut dari transaksi penjualan. |
| Tax Paid | Pajak yang dibayar/tercatat dari transaksi pembelian atau biaya. |
| Net Tax Payable | Tax collected dikurangi tax paid. |

Catatan demo:

- UI memiliki seksi per tax rate, tetapi route yang diaudit mengisi daftar detail `tax_collected` dan `tax_paid` sebagai array kosong, sementara total tetap dihitung dari service. Jangan mengklaim breakdown per tax rate sudah terisi bila data tampilan tidak menunjukkannya.
- Laporan ini membantu rekonsiliasi internal, bukan pengganti validasi formal pajak.
- Cash-basis Tax Summary belum diekspos dan tetap design-only sesuai catatan arsitektur proyek.

## 13. Profitabilitas Proyek

Profitabilitas Proyek menjawab pertanyaan: "project mana yang menghasilkan profit atau rugi?"

Fungsi bisnis:

| Fungsi | Penjelasan |
| --- | --- |
| Melihat income per project | Pendapatan dari baris GL yang memiliki `project_id`. |
| Melihat expense per project | Biaya/HPP dari baris GL yang memiliki `project_id`. |
| Mengukur profit | Income dikurangi expenses. |
| Membandingkan Akrual/Kas | UI web menyediakan pilihan basis Akrual dan Kas. |

Filter:

| Filter | Status | Cara menjelaskan |
| --- | --- | --- |
| Dari tanggal | Wajib untuk menghasilkan laporan. | Awal periode project. |
| Sampai tanggal | Wajib dan tidak boleh sebelum dari tanggal. | Akhir periode project. |
| Project | Opsional. | Kosong berarti semua project. |
| Basis | `accrual` atau `cash`. | Pilih sesuai kebutuhan demo. |

Sumber data:

- Basis Akrual membaca `account_transactions` yang memiliki `project_id`, join ke `projects`, dan hanya memakai akun income/expense.
- Basis Kas memakai ledger cash-basis internal yang memiliki `project_id`.
- Total profit dihitung dari total income dikurangi total expenses.

Guard rentang:

| Mode | Batas default |
| --- | --- |
| View | 730 hari. |
| Export/PDF | 366 hari. |

Contoh alur demo:

1. Siapkan project demo di master data.
2. Buat invoice/expense/bill yang memakai project tersebut.
3. Buka Profitabilitas Proyek.
4. Pilih periode dan project.
5. Generate basis Akrual.
6. Ubah basis ke Kas jika ada pembayaran yang relevan.
7. Export PDF untuk ringkasan manajemen project.

Catatan demo:

- Jika project kosong, cek apakah transaksi benar-benar menyimpan `project_id`.
- Jika basis Kas kosong, cek apakah sudah ada pembayaran/sumber cash-basis yang didukung.
- Jangan memakai laporan ini untuk semua biaya perusahaan bila transaksi tidak dipasang ke project.

## 14. Piutang, Utang, dan Aging

Bagian ini mencakup Umur Piutang, Umur Utang, Ringkasan Saldo Pelanggan, dan Ringkasan Saldo Vendor. Keempatnya penting untuk menjelaskan uang yang belum diterima atau belum dibayar.

### Umur Piutang

Umur Piutang menunjukkan invoice pelanggan yang sudah delivered, masih outstanding, dan dikelompokkan berdasarkan umur jatuh tempo.

| Area | Detail |
| --- | --- |
| Route | `/reports/receivables-aging` |
| Filter UI web | `date` sebagai tanggal posisi. |
| Export/PDF | CSV/XLSX/PDF tersedia. |
| Sumber data | `sale_invoices` yang sudah delivered, belum deleted, invoice date <= tanggal, dan outstanding > 0. |
| Bucket | Current, 1-30, 31-60, 61-90, 90+. |

Cara membaca: semakin besar angka di bucket 61-90 atau 90+, semakin tinggi risiko penagihan. Gunakan laporan ini untuk prioritas follow-up customer.

### Umur Utang

Umur Utang menunjukkan bill vendor yang sudah opened, masih outstanding, dan dikelompokkan berdasarkan umur jatuh tempo.

| Area | Detail |
| --- | --- |
| Route | `/reports/payables-aging` |
| Filter UI web | `date` sebagai tanggal posisi. |
| Export/PDF | CSV/XLSX/PDF tersedia. |
| Sumber data | `bills` yang sudah opened, belum deleted, bill date <= tanggal, dan outstanding > 0. |
| Bucket | Current, 1-30, 31-60, 61-90, 90+. |

Cara membaca: gunakan bucket tertua untuk menentukan prioritas pembayaran vendor dan risiko operasional.

### Ringkasan Saldo Pelanggan

Ringkasan Saldo Pelanggan menunjukkan total invoice, total paid, total credited, dan outstanding balance per pelanggan.

| Area | Detail |
| --- | --- |
| Route | `/reports/customer-balance-summary` |
| Filter UI web | `as_of_date`. |
| Export/PDF | CSV/XLSX/PDF tersedia. |
| Sumber data | `sale_invoices` delivered sampai tanggal posisi. |

Cara membaca: outstanding balance adalah nilai yang masih perlu ditagih setelah pembayaran dan credit note.

### Ringkasan Saldo Vendor

Ringkasan Saldo Vendor menunjukkan total billed, total paid, total credited, dan outstanding balance per vendor.

| Area | Detail |
| --- | --- |
| Route | `/reports/vendor-balance-summary` |
| Filter UI web | `as_of_date`. |
| Export/PDF | CSV/XLSX/PDF tersedia. |
| Sumber data | `bills` opened sampai tanggal posisi. |

Cara membaca: outstanding balance adalah nilai yang masih perlu dibayar setelah bill payment dan vendor credit.

Catatan demo:

- Service aging mendukung parameter `customer_id` dan `vendor_id`, dan API controller juga memvalidasi filter tersebut, tetapi UI web yang diaudit hanya menyediakan tanggal. Jangan klaim filter pelanggan/vendor tersedia di halaman web aging.
- Aging memakai due date. Jika bucket tampak tidak sesuai, cek due date invoice/bill.
- Jika invoice/bill masih draft, laporan aging tidak akan menampilkannya.

## 15. Laporan Persediaan/Barang

Bagian ini mencakup laporan item penjualan, item pembelian, valuasi persediaan, dan detail barang persediaan.

### Penjualan per Barang/Jasa

| Area | Detail |
| --- | --- |
| Route | `/reports/sales-by-items` |
| Filter UI web | `from_date`, `to_date`. |
| Export/PDF | CSV/XLSX/PDF tersedia. |
| Sumber data | `item_entries` yang terhubung ke `sale_invoices` delivered. |
| Angka utama | Total quantity sold dan total amount. |

Gunakan laporan ini untuk menjelaskan item mana yang paling banyak dijual atau paling besar nilai penjualannya.

### Pembelian per Barang/Jasa

| Area | Detail |
| --- | --- |
| Route | `/reports/purchases-by-items` |
| Filter UI web | `from_date`, `to_date`. |
| Export/PDF | CSV/XLSX/PDF tersedia. |
| Sumber data | `item_entries` yang terhubung ke `bills` opened. |
| Angka utama | Total quantity purchased dan total amount. |

Gunakan laporan ini untuk menjelaskan item mana yang paling besar nilai pembeliannya.

### Laporan Valuasi Persediaan

| Area | Detail |
| --- | --- |
| Route | `/reports/inventory-valuation-sheet` |
| Filter UI web | `as_of_date`. |
| Export/PDF | CSV/XLSX/PDF tersedia. |
| Sumber data | `inventory_transactions` sampai tanggal posisi, join ke `items`. |
| Angka utama | Quantity on hand, average cost, dan total value. |

Cara membaca: laporan ini menunjukkan nilai stok per item berdasarkan kuantitas on hand dan average cost dari transaksi inventaris.

### Detail Barang Persediaan

| Area | Detail |
| --- | --- |
| Route | `/reports/inventory-item-details` |
| Filter UI web | Item, dari tanggal, sampai tanggal. |
| Export/PDF | CSV/XLSX tersedia; PDF tidak terverifikasi. |
| Sumber data | `inventory_transactions` item tertentu, join ke `warehouses`. |
| Angka utama | Opening balance, in, out, rate, dan running balance. |

Cara membaca: gunakan laporan ini untuk menelusuri pergerakan stok satu item. Warehouse tampil sebagai kolom, tetapi tidak ada filter warehouse pada UI web yang diaudit.

Catatan demo:

- Sales/Purchases by Items membaca dokumen yang sudah masuk status akuntansi: invoice delivered dan bill opened.
- Valuasi persediaan hanya menampilkan item yang memiliki quantity/value relevan.
- Detail Barang Persediaan cocok untuk audit item tertentu, bukan ringkasan semua item.

## 16. Laporan Transaksi Lainnya

Bagian ini mencakup Lembar Jurnal, Transaksi per Kontak, dan Transaksi per Referensi.

### Lembar Jurnal

| Area | Detail |
| --- | --- |
| Route | `/reports/journal-sheet` |
| Filter UI web | `from_date`, `to_date`. |
| Export/PDF | CSV/XLSX/PDF tersedia. |
| Sumber data | `account_transactions`, join ke `accounts`, dikelompokkan per transaction type dan reference. |
| Guard rentang | View 730 hari; export/PDF 366 hari. |

Cara membaca: setiap transaksi ditampilkan sebagai kelompok debit-credit. Grand total debit dan credit harus sama.

### Transaksi per Kontak

| Area | Detail |
| --- | --- |
| Route | `/reports/transactions-by-contact` |
| Filter UI web | Kontak, dari tanggal, sampai tanggal. |
| Export/PDF | CSV/XLSX/PDF tersedia. |
| Sumber data | Sale invoices, payment receives, credit notes, bills, bill payments, dan vendor credits. |
| Guard rentang | View 730 hari; export/PDF 366 hari. |

Cara membaca: laporan ini menjawab semua transaksi apa saja yang terjadi dengan satu customer/vendor.

### Transaksi per Referensi

| Area | Detail |
| --- | --- |
| Route | `/reports/transactions-by-reference` |
| Filter UI web | Dari tanggal, sampai tanggal, reference opsional, akun opsional. |
| Export/PDF | CSV/XLSX tersedia; PDF tidak terverifikasi. |
| Sumber data | `account_transactions`, join ke `accounts` dan `contacts`. |
| Guard rentang | View 730 hari; export 366 hari. |

Cara membaca: laporan ini mencari baris GL berdasarkan transaction number, reference number, reference type, atau reference id. Gunakan untuk menjawab "nomor dokumen ini masuk akun apa saja?"

Catatan demo:

- Lembar Jurnal dan Buku Besar membaca GL, sehingga cocok untuk audit angka.
- Transaksi per Kontak membaca beberapa tabel dokumen, sehingga cocok untuk timeline customer/vendor.
- Transaksi per Referensi membaca GL dan cocok untuk pencarian nomor/reference.

## 17. Export, PDF, dan Batas Rentang Tanggal

Export tabular memakai route `.../export` dengan `format=csv` atau `format=xlsx`. PDF memakai route `.../pdf` dan DomPDF. Frontend mengunduh PDF/export via `window.location.href`, bukan navigasi Inertia.

Status export:

| Status | Laporan |
| --- | --- |
| CSV/XLSX tersedia | Semua 18 laporan memiliki route export. |
| PDF tersedia | Neraca, Laba Rugi, Profitabilitas Proyek, Neraca Saldo, Arus Kas, Umur Piutang, Umur Utang, Ringkasan Pajak, Buku Besar, Penjualan per Barang/Jasa, Pembelian per Barang/Jasa, Ringkasan Saldo Pelanggan, Ringkasan Saldo Vendor, Lembar Jurnal, Valuasi Persediaan, Transaksi per Kontak. |
| PDF tidak terverifikasi | Detail Barang Persediaan dan Transaksi per Referensi. |

Route PDF yang terverifikasi:

| PDF | Route |
| --- | --- |
| Neraca | `/reports/balance-sheet/pdf` |
| Laba Rugi | `/reports/income-statement/pdf` |
| Profitabilitas Proyek | `/reports/project-profitability/pdf` |
| Neraca Saldo | `/reports/trial-balance/pdf` |
| Arus Kas | `/reports/cash-flow/pdf` |
| Umur Piutang | `/reports/receivables-aging/pdf` |
| Umur Utang | `/reports/payables-aging/pdf` |
| Ringkasan Pajak | `/reports/tax-summary/pdf` |
| Buku Besar | `/reports/general-ledger/pdf` |
| Penjualan per Barang/Jasa | `/reports/sales-by-items/pdf` |
| Pembelian per Barang/Jasa | `/reports/purchases-by-items/pdf` |
| Ringkasan Saldo Pelanggan | `/reports/customer-balance-summary/pdf` |
| Ringkasan Saldo Vendor | `/reports/vendor-balance-summary/pdf` |
| Laporan Valuasi Persediaan | `/reports/inventory-valuation-sheet/pdf` |
| Transaksi per Kontak | `/reports/transactions-by-contact/pdf` |
| Lembar Jurnal | `/reports/journal-sheet/pdf` |

Batas rentang tanggal dari `ReportDateRangeGuard`:

| Mode | Default | Konfigurasi |
| --- | --- | --- |
| View | 730 hari | `REPORT_VIEW_MAX_DAYS` atau `config('reports.date_range.view_max_days')`. |
| Export/PDF | 366 hari | `REPORT_EXPORT_MAX_DAYS` atau `config('reports.date_range.export_max_days')`. |

Laporan yang memakai guard:

| Laporan | View | Export | PDF |
| --- | --- | --- | --- |
| Profitabilitas Proyek | Ya | Ya | Ya |
| Buku Besar | Ya | Ya | Ya |
| Lembar Jurnal | Ya | Ya | Ya |
| Transaksi per Kontak | Ya | Ya | Ya |
| Transaksi per Referensi | Ya | Ya | Tidak ada PDF terverifikasi |

Kalimat demo yang aman: "Untuk laporan detail yang bisa sangat besar, sistem membatasi rentang tanggal. Jika export gagal karena periode terlalu panjang, pecah laporan per bulan atau per kuartal."

## 18. Contoh Alur Demo Laporan

### Alur 1 - Laporan manajemen bulanan

1. Buka Laba Rugi.
2. Pilih periode bulan berjalan.
3. Generate basis Akrual.
4. Tunjukkan income, expenses, dan net income.
5. Export PDF.
6. Buka Neraca pada tanggal akhir bulan.
7. Tunjukkan assets, liabilities, equity.
8. Buka Arus Kas untuk menjelaskan perubahan uang pada akun Cash/Bank.

### Alur 2 - Audit satu angka pendapatan

1. Buka Laba Rugi dan catat total income.
2. Buka Buku Besar.
3. Pilih akun pendapatan terkait.
4. Pilih periode sama.
5. Tunjukkan transaksi pembentuk angka.
6. Buka Lembar Jurnal jika perlu menunjukkan pasangan debit-credit.
7. Export CSV untuk review spreadsheet.

### Alur 3 - Follow-up piutang dan pembayaran

1. Buka Umur Piutang.
2. Pilih tanggal hari ini.
3. Tunjukkan customer dengan bucket 31-60 atau 90+.
4. Buka Ringkasan Saldo Pelanggan untuk outstanding total.
5. Buka Transaksi per Kontak untuk customer yang sama.
6. Jelaskan invoice, payment receive, dan credit note yang terkait.

### Alur 4 - Review utang vendor

1. Buka Umur Utang.
2. Pilih tanggal hari ini.
3. Tunjukkan vendor prioritas bayar.
4. Buka Ringkasan Saldo Vendor.
5. Buka Transaksi per Kontak untuk vendor yang sama.
6. Jelaskan bill, bill payment, dan vendor credit.

### Alur 5 - Persediaan dan item

1. Buka Penjualan per Barang/Jasa untuk melihat item terjual.
2. Buka Pembelian per Barang/Jasa untuk melihat item dibeli.
3. Buka Laporan Valuasi Persediaan pada tanggal akhir bulan.
4. Pilih satu item di Detail Barang Persediaan.
5. Tunjukkan pergerakan in/out dan running balance.

### Alur 6 - Profitabilitas project

1. Buka Profitabilitas Proyek.
2. Pilih periode dan project demo.
3. Generate basis Akrual.
4. Tunjukkan income, expenses, dan profit.
5. Ubah basis ke Kas bila pembayaran tersedia.
6. Jelaskan mengapa hasil Akrual dan Kas bisa berbeda.

## 19. Checklist Sebelum Membuka Laporan

Gunakan checklist ini sebelum demo:

| Area | Checklist |
| --- | --- |
| Login | Gunakan user super admin/admin atau role yang punya permission report terkait. |
| Preferensi | Base currency, timezone, date format, dan fiscal year sudah benar. |
| COA | Akun Cash/Bank, AR, AP, Tax Payable, Income, Expense, Inventory, COGS, dan Equity tersedia. |
| Status dokumen | Invoice sudah delivered, bill sudah opened, expense/manual journal sudah published. |
| Payment | Payment receive dan bill payment sudah dibuat bila ingin menunjukkan kas/aging berubah. |
| Project | Project terpasang pada transaksi yang ingin dilihat di Profitabilitas Proyek atau Buku Besar. |
| Item | Item inventory/non-inventory/service sudah punya transaksi jika ingin melihat laporan item. |
| Pajak | Tax rate dan akun Tax Payable sudah dipakai di transaksi jika ingin melihat Ringkasan Pajak. |
| Rentang tanggal | Periode laporan detail tidak melebihi guard default. |
| Export/PDF | File export dan PDF yang akan ditunjukkan sudah diuji. |

## 20. Checklist Presentasi/Demo

Checklist pembawaan materi:

| Langkah | Catatan |
| --- | --- |
| Mulai dari tujuan laporan | Jelaskan pertanyaan bisnis yang dijawab, bukan langsung teknis. |
| Jelaskan filter | Tekankan apakah laporan memakai per tanggal atau periode. |
| Tunjukkan satu ringkasan | Laba Rugi, Neraca, atau Arus Kas. |
| Tunjukkan satu audit trail | Buku Besar atau Lembar Jurnal. |
| Tunjukkan AR/AP | Aging dan saldo customer/vendor. |
| Tunjukkan item/persediaan | Sales/Purchases by Items, Valuasi Persediaan, atau Detail Barang. |
| Tunjukkan export | CSV/XLSX untuk analisis, PDF untuk laporan formal. |
| Batasi klaim basis | Basis Akrual/Kas hanya didemokan pada Laba Rugi dan Profitabilitas Proyek. |
| Hindari periode besar | Untuk laporan detail, pilih periode pendek agar tidak kena guard dan demo tetap cepat. |
| Tutup dengan checklist operasional | Status transaksi, akun, tanggal, permission, dan filter adalah hal pertama yang dicek jika angka tidak muncul. |

Checklist jawaban cepat:

| Pertanyaan | Jawaban aman |
| --- | --- |
| Kenapa angka invoice belum muncul? | Cek invoice sudah delivered, tanggal masuk periode, akun benar, dan tidak deleted. |
| Kenapa bill belum muncul di utang? | Cek bill sudah opened, tanggal bill masuk posisi, dan outstanding masih ada. |
| Kenapa Laba Rugi beda dengan Arus Kas? | Laba Rugi mengukur performa, Arus Kas mengukur gerak Cash/Bank. Timing pengakuan berbeda. |
| Kenapa export gagal? | Periode melebihi batas guard atau filter wajib belum lengkap. |
| Kenapa basis Kas tidak ada di Neraca? | UI/route/export/PDF yang diaudit belum mengekspos basis Kas untuk Neraca. |

## 21. Catatan Field/Menu Yang Belum Terverifikasi

Catatan ini mencegah presenter mengklaim fitur melebihi hasil audit phase ini. Untuk area yang tidak dapat diklaim dari UI atau route yang diaudit, gunakan status: **Belum terverifikasi dari kode pada phase ini**.

| Area | Catatan belum terverifikasi |
| --- | --- |
| PDF Detail Barang Persediaan | Belum terverifikasi dari kode pada phase ini. Route export ada, tetapi route/template PDF tidak muncul pada audit route dan view PDF. |
| PDF Transaksi per Referensi | Belum terverifikasi dari kode pada phase ini. Route export ada, tetapi route/template PDF tidak muncul pada audit route dan view PDF. |
| Basis Kas Neraca di UI | Belum terverifikasi dari kode pada phase ini. Service internal menerima basis, tetapi UI web/route/export/PDF/API yang diaudit tidak mengeksposnya. |
| Basis Kas Neraca Saldo di UI | Belum terverifikasi dari kode pada phase ini. Service internal menerima basis, tetapi UI web/route/export/PDF/API yang diaudit tidak mengeksposnya. |
| Basis Kas Ringkasan Pajak | Belum terverifikasi dari kode pada phase ini. Cash-basis Tax Summary tetap design-only dan tidak boleh diekspos sebagai fitur UI/export/PDF. |
| Filter customer pada Umur Piutang web | Belum terverifikasi dari kode pada phase ini sebagai filter UI web. Service/API memiliki parameter customer, tetapi halaman React yang diaudit hanya menampilkan tanggal. |
| Filter vendor pada Umur Utang web | Belum terverifikasi dari kode pada phase ini sebagai filter UI web. Service/API memiliki parameter vendor, tetapi halaman React yang diaudit hanya menampilkan tanggal. |
| Filter warehouse pada Detail Barang Persediaan | Belum terverifikasi dari kode pada phase ini sebagai filter UI web. Warehouse tampil sebagai kolom dari transaksi, bukan filter halaman. |
| Breakdown Ringkasan Pajak per tax rate | Belum terverifikasi dari kode pada phase ini sebagai data tampilan terisi. Route web yang diaudit mengisi detail tax collected/paid sebagai array kosong dan menampilkan total dari service. |
| Saldo awal historis Arus Kas | Belum terverifikasi dari kode pada phase ini sebagai saldo awal kas historis lengkap. Route web mengisi opening balance 0 dan closing balance dari net change periode. |
| Branch filter pada laporan | Belum terverifikasi dari kode pada phase ini sebagai filter UI laporan. |
| Screenshot visual final | Belum terverifikasi dari kode pada phase ini karena phase ini hanya mengubah dokumentasi, bukan menjalankan browser screenshot. |

Sikap presentasi yang aman: bila peserta bertanya fitur di luar catatan terverifikasi, jawab bahwa laporan saat ini mendukung route/filter yang ditunjukkan di layar, sedangkan field lain perlu audit terpisah sebelum dijanjikan sebagai fitur produk.
