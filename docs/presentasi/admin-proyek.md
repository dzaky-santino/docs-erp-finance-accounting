# Presentasi Admin Proyek

## 1. Tujuan Dokumen

Dokumen ini menjadi bahan presentasi admin untuk menu Proyek. Fokusnya adalah cara menyiapkan data proyek, memilih proyek pada transaksi yang sudah mendukung, dan membaca dampaknya pada laporan yang memiliki filter atau agregasi proyek.

Dokumen ini disusun dari audit route, page, request, service, permission, dan test yang ada pada kode. Jika field atau menu belum terlihat dari kode yang diaudit, statusnya ditulis eksplisit sebagai "Belum terverifikasi dari kode pada phase ini".

Rujukan silang:

- Kontak customer/vendor yang dapat ditautkan ke proyek: [admin-kontak.md](admin-kontak.md)
- Transaksi penjualan yang membawa proyek: [admin-penjualan.md](admin-penjualan.md)
- Transaksi pembelian yang membawa proyek: [admin-pembelian.md](admin-pembelian.md)
- Biaya langsung per proyek: [admin-biaya.md](admin-biaya.md)
- Project Profitability dan General Ledger: [admin-laporan.md](admin-laporan.md)

## 2. Gambaran Umum Menu Proyek

Proyek dipakai sebagai dimensi analisis. Data proyek tidak menggantikan akun, kontak, cabang, warehouse, atau dokumen bisnis. Proyek menempel pada transaksi tertentu, lalu `project_id` dibawa ke jurnal/GL jika flow modul tersebut memang mendukung posting proyek.

| Area | Route/page aktual | Request/Form | Service | Permission | Status audit |
| --- | --- | --- | --- | --- | --- |
| Daftar Proyek | `GET /projects`, page `projects/index` | Filter client-side: search, status, contact, rows per page | `ProjectService::list()` | `project.view` | Terverifikasi |
| Tambah Proyek | `GET /projects/create`, page `projects/create` | `POST /api/projects` memakai `StoreProjectRequest` | `ProjectService::create()` | `project.create` | Terverifikasi |
| Detail Proyek | `GET /projects/{id}`, page `projects/show` | Menampilkan field utama dan audit waktu | `ProjectService::find()` | `project.view` | Terverifikasi |
| Edit Proyek | `GET /projects/{id}/edit`, page `projects/edit` | `PUT /api/projects/{id}` memakai `UpdateProjectRequest` | `ProjectService::update()` | `project.edit` | Terverifikasi |
| Hapus Proyek | `DELETE /api/projects/{id}` | Delete API dengan guard referensi | `ProjectService::delete()` | `project.delete` | Terverifikasi |
| API Proyek | `/api/projects` dan `/api/projects/{id}` | List, show, create, update, delete | `ProjectController` + `ProjectService` | Sesuai aksi proyek | Terverifikasi |
| Laporan Profit Proyek | `GET /reports/project-profitability`, page `reports/project-profitability` | Filter tanggal, proyek, basis laporan, export CSV/XLSX/PDF | `ReportService::projectProfitability()` | `report-project-profitability.view` | Terverifikasi |

Sidebar menampilkan grup `Projects` dengan menu `Projects` dan `New Project`. Laporan `Project Profitability` berada di area Reports. Tidak ditemukan route atau fitur "session project" yang mengunci seluruh aplikasi ke satu proyek aktif.

Guard hapus proyek mencegah penghapusan jika proyek sudah dipakai pada invoice, estimate, sale receipt, credit note, bill, vendor credit, expense, expense category, manual journal entry, item entry, atau GL transaction.

## 3. Kapan Proyek Perlu Dipakai

Gunakan proyek saat perusahaan perlu membaca pendapatan, biaya, dan profit berdasarkan pekerjaan, kontrak, implementasi, acara, kampanye, atau aktivitas internal yang lintas dokumen.

Contoh penggunaan yang cocok:

- Implementasi ERP untuk satu klien.
- Renovasi kantor atau cabang tertentu.
- Pelatihan internal yang perlu dipantau biayanya.
- Proyek jasa yang memiliki invoice, bill, expense, dan jurnal manual terkait.

Jangan gunakan proyek sebagai pengganti kontak customer/vendor, akun COA, cabang, warehouse, atau nomor dokumen. Jika kebutuhan hanya membedakan akun akuntansi, gunakan Bagan Akun. Jika kebutuhan hanya membedakan pihak bisnis, gunakan Contact.

## 4. Urutan Penggunaan Yang Disarankan

1. Pastikan role demo atau role kustom memiliki `project.view`, `project.create`, `project.edit`, `project.delete`, dan `report-project-profitability.view` sesuai kebutuhan demo.
2. Buat contact customer/vendor jika proyek perlu ditautkan ke pihak tertentu.
3. Buat proyek dari `Projects -> New Project`.
4. Isi transaksi yang mendukung proyek, terutama invoice, bill, expense, manual journal, sale receipt, credit note, vendor credit, atau estimate.
5. Post/publish/open/deliver transaksi sesuai workflow modul agar GL terbentuk.
6. Buka `Reports -> Project Profitability` untuk membaca total income, expense, dan profit per proyek.
7. Buka `Reports -> General Ledger` atau detail akun untuk menelusuri baris GL dengan filter proyek.
8. Jika angka belum muncul, cek tanggal laporan, status transaksi, basis laporan yang dipilih, dan apakah proyek diisi pada field yang benar.

## 5. Sub Menu/Area Proyek Dalam Sistem

| Area | Fungsi utama | Catatan presentasi |
| --- | --- | --- |
| Projects | Menampilkan daftar proyek, filter status/contact/search, dan pagination client-side. | Gunakan untuk menjelaskan data master proyek. |
| New Project | Membuat proyek baru dengan nama, contact opsional, deadline, cost estimate, dan status. | Cocok untuk awal demo. |
| Project Detail | Menampilkan informasi proyek, contact, deadline, cost estimate, status, created at, dan updated at. | Pakai untuk menunjukkan hasil input. |
| Project Edit | Mengubah field proyek selama masih valid. | Perubahan master tidak otomatis mengubah histori dokumen yang sudah menyimpan nama proyek di tampilan lama, tetapi `project_id` tetap referensi utama. |
| Project API | Endpoint `/api/projects` untuk integrasi internal/web. | Akses mengikuti permission proyek. |
| Project Profitability | Laporan profit proyek dengan filter tanggal, proyek, basis, export, dan PDF. | Ini layar utama untuk membaca hasil demo. |
| General Ledger | Filter `project_id` pada buku besar umum. | Pakai untuk drill-down baris GL per proyek. |
| Account Detail | Detail akun mendukung filter `project_id`. | Berguna saat ingin menelusuri akun tertentu dalam satu proyek. |

Belum ditemukan menu "Current Project", "Switch Project", atau sesi proyek global. Pilihan proyek pada transaksi berasal dari daftar proyek yang dikirim page terkait, bukan dari state sesi proyek.

## 6. Daftar Input Proyek

| Field | Tipe/aturan | Wajib/Opsional | Keterangan |
| --- | --- | --- | --- |
| `name` | String, maksimum 255 karakter | Wajib | Nama proyek yang tampil di daftar, transaksi, dan laporan. |
| `contact_id` | Contact aktif/tidak soft-deleted | Opsional | Menghubungkan proyek ke customer/vendor/contact tertentu. Kosongkan untuk proyek internal. |
| `deadline` | Tanggal | Opsional | Target tanggal selesai proyek. |
| `cost_estimate` | Numeric, minimal 0, maksimal `9999999999.99999`, disimpan decimal(15,5) | Opsional | Estimasi biaya internal. Ini bukan field `budget` terpisah. |
| `status` | Enum `active`, `completed`, `cancelled`; default `active` | Wajib pada form | Dipakai untuk filter, badge, dan urutan daftar. |
| `created_at` | Timestamp sistem | Otomatis | Tampil di detail proyek. |
| `updated_at` | Timestamp sistem | Otomatis | Tampil di detail proyek. |

Field `code`, `start_date`, `end_date`, `budget`, `description`, dan `is_active` belum ada pada model/request/page Proyek yang diaudit. Jika butuh kode proyek untuk demo, masukkan kode di awal `name`, misalnya `PRJ-ERP-UMMI - Implementasi ERP RS UMMI`.

### Kegunaan Setiap Field

- `name`: identitas utama proyek pada master data, dropdown transaksi, dan laporan.
- `contact_id`: konteks pihak terkait. Ini membantu admin menjelaskan proyek ke customer/vendor tertentu, tetapi tidak wajib.
- `deadline`: target administratif, bukan cut-off akuntansi.
- `cost_estimate`: angka estimasi biaya untuk referensi; laporan profit proyek tetap dihitung dari transaksi/GL, bukan dari field ini.
- `status`: penanda lifecycle proyek. Daftar proyek mengurutkan `active` lebih dulu, lalu `completed`, lalu status lain.

### Wajib/Opsional

Field wajib hanya `name` dan `status` pada form. `contact_id`, `deadline`, dan `cost_estimate` boleh kosong. Form akan mengubah nilai kosong untuk field opsional menjadi `null` sebelum validasi.

### Contoh Input

| Nama Proyek | Contact | Deadline | Cost Estimate | Status | Keterangan Demo |
| --- | --- | --- | --- | --- | --- |
| `PRJ-ERP-UMMI - Implementasi ERP RS UMMI` | RS UMMI | 2026-03-31 | 150.000.000 | `active` | Demo proyek jasa dengan invoice, bill, dan expense. |
| `PRJ-RENOV-IGD - Renovasi Area IGD` | Kosong/internal | 2026-04-30 | 300.000.000 | `active` | Demo proyek internal dengan biaya dan jurnal manual. |
| `PRJ-TRAIN-FIN - Pelatihan Staff Finance` | Kosong/internal | 2026-02-15 | 25.000.000 | `active` | Demo proyek kecil untuk expense dan laporan. |

## 7. Pengaruh Proyek Ke Transaksi

| Modul | Header project | Line project | GL membawa project | UI terlihat | Report terdampak | Status audit |
| --- | --- | --- | --- | --- | --- | --- |
| Manual Journal | Tidak | Ya, per baris jurnal | Ya, mengikuti baris jurnal | Create/Edit journal | Project Profitability, General Ledger | Terverifikasi |
| Expense/Biaya | Ya | Ya, per kategori/baris biaya | Ya pada debit expense dari baris kategori; sisi payment tidak membawa proyek | Create/Edit expense | Project Profitability, General Ledger | Terverifikasi |
| Invoice/Faktur | Ya | Tidak terlihat sebagai input line | Ya, dari header invoice saat deliver | Create/Edit/Index/Show invoice | Project Profitability, General Ledger | Terverifikasi |
| Bill/Tagihan | Ya | Tidak terlihat sebagai input line | Ya, dari header bill saat open | Create/Edit/Index/Show bill | Project Profitability, General Ledger | Terverifikasi |
| Estimate | Ya | Tidak terlihat sebagai input line | Tidak membuat GL sampai dikonversi | Create/Edit/Index/Show estimate | Indirect saat convert ke invoice | Terverifikasi |
| Sale Receipt | Ya | Tidak terlihat sebagai input line | Ya saat close/posting | Create/Edit sale receipt | Project Profitability, General Ledger | Terverifikasi |
| Credit Note | Ya | Tidak terlihat sebagai input line | Ya saat open/refund/application yang didukung | Create/Edit credit note | Project Profitability, General Ledger | Terverifikasi |
| Vendor Credit | Ya | Tidak terlihat sebagai input line | Ya saat open/refund/application yang didukung | Create/Edit vendor credit | Project Profitability, General Ledger | Terverifikasi |
| Payment Receive | Tidak boleh input langsung | Tidak boleh input langsung | Payment GL tidak membawa proyek langsung | Index/Show menampilkan ringkasan proyek dari invoice alokasi | Display summary; profit cash-basis mengikuti sumber yang didukung | Terverifikasi |
| Bill Payment | Tidak boleh input langsung | Tidak boleh input langsung | Payment GL tidak membawa proyek langsung | Index/Show menampilkan ringkasan proyek dari bill alokasi | Display summary; profit cash-basis mengikuti sumber yang didukung | Terverifikasi |
| Landed Cost | Mengikuti bill target | Belum menjadi field presentasi utama | Baris landed cost memakai proyek dari target bill | Detail UI landed cost tidak diaudit penuh | General Ledger jika GL terkait membawa proyek | Sebagian terverifikasi |
| Banking/Cashflow/Inventory/Warehouse | Belum terlihat sebagai input proyek utama | Belum terlihat | Belum diverifikasi | Belum diverifikasi | Belum diverifikasi | Belum terverifikasi dari kode pada phase ini |

### Manual Journal

Pada Manual Journal, proyek diisi per baris jurnal. Saat jurnal dipublish, `project_id` pada baris jurnal ikut masuk ke `account_transactions`. Ini membuat baris income/expense yang memakai akun laba rugi bisa dibaca di Project Profitability, dan semua baris terkait bisa ditelusuri di General Ledger.

Untuk demo, gunakan dua baris jurnal dengan proyek yang sama jika ingin menunjukkan debit dan credit dalam satu proyek. Jika hanya salah satu baris yang diberi proyek, hanya baris itu yang masuk filter proyek.

### Biaya

Expense mendukung proyek pada header dan pada baris kategori. Untuk dampak laporan, yang penting adalah proyek pada baris kategori biaya karena service posting memakai proyek kategori untuk baris debit expense. Sisi pembayaran/kas tidak membawa proyek.

Kesalahan demo yang sering terjadi adalah mengisi proyek di header tetapi lupa mengisi proyek pada baris kategori. Dalam kondisi itu, biaya bisa tersimpan, tetapi Project Profitability tidak menampilkan expense proyek sesuai ekspektasi.

### Faktur

Invoice memiliki `project_id` pada header. Saat invoice delivered/posting, baris GL yang terbentuk membawa proyek dari header invoice. Estimate yang dikonversi ke invoice juga mempertahankan proyek.

Untuk demo profit proyek, buat invoice dengan proyek lalu deliver. Invoice draft yang belum diposting belum menghasilkan GL dan belum masuk laporan berbasis GL.

### Tagihan

Bill memiliki `project_id` pada header. Saat bill dibuka/opened, GL yang terbentuk membawa proyek dari header bill. Bill payment tidak menerima input proyek langsung; tampilan payment bisa menampilkan ringkasan proyek dari bill yang dialokasikan.

Jika memakai landed cost, baris GL landed cost yang diaudit mengikuti proyek dari target bill. Detail UI landed cost tidak dijadikan materi utama pada dokumen ini.

### Modul Lain Yang Belum Terverifikasi

Estimate, Sale Receipt, Credit Note, dan Vendor Credit sudah terverifikasi mendukung proyek pada header dan/atau posting yang relevan. Payment Receive dan Bill Payment tidak mendukung input proyek langsung, tetapi dapat menampilkan ringkasan proyek dari dokumen yang dibayar.

Banking, Cashflow, Inventory Adjustment, Warehouse Transfer, dan sesi proyek global belum terverifikasi sebagai area input proyek dari kode pada phase ini. Jangan klaim area tersebut sudah lengkap untuk proyek saat presentasi.

## 8. Pengaruh Proyek Ke Laporan

| Laporan/area | Filter proyek | Sumber data proyek | Export/PDF | Basis | Catatan |
| --- | --- | --- | --- | --- | --- |
| Project Profitability | Ya | `account_transactions.project_id` untuk akrual; ledger kas internal untuk basis kas yang didukung | CSV, XLSX, PDF | Akrual/Kas pada laporan ini | Laporan utama untuk profit proyek. |
| General Ledger | Ya | `account_transactions.project_id` | CSV/XLSX/PDF sesuai route report | Tidak ada filter basis pada UI GL | Cocok untuk drill-down transaksi proyek. |
| Account Detail | Ya lewat query `project_id` | Transaksi akun yang membawa proyek | Mengikuti halaman detail akun | Tidak dibahas sebagai report utama | Berguna untuk cek akun spesifik. |
| Journal Sheet | Tidak terlihat sebagai filter proyek | Baris GL bisa punya proyek, tetapi UI filter proyek tidak diaudit | Export tersedia untuk report | Tidak dibahas sebagai filter proyek | Jangan pakai sebagai laporan utama proyek. |
| Transactions by Contact/Reference | Tidak terlihat sebagai filter proyek | Sumber transaksi/reference, bukan dimensi proyek utama | CSV/XLSX tersedia | Tidak dibahas sebagai filter proyek | Gunakan untuk audit kontak/reference, bukan profit proyek. |
| Income Statement | Tidak ada filter proyek | Agregasi akun laba rugi umum | CSV/XLSX tersedia | Mengikuti fitur laporan ini | Bisa memuat transaksi proyek secara total, tetapi tidak memisahkan proyek. |
| Laporan lain | Belum terlihat sebagai filter proyek | Belum diverifikasi | Mengikuti masing-masing report | Belum dibahas untuk proyek | Belum terverifikasi dari kode pada phase ini. |

### Project Profitability

Project Profitability menghitung income, expense, dan profit per proyek. Pada basis akrual, sumber utamanya adalah baris `account_transactions` yang memiliki `project_id`, akun laba rugi, dan proyek yang belum soft-deleted. Pada basis kas, laporan memakai ledger kas internal yang sudah mendukung sumber tertentu dan tetap memfilter proyek.

Filter utama untuk presentasi:

- `from_date` dan `to_date` untuk rentang laporan.
- `project_id` untuk memilih satu proyek atau melihat semua proyek.
- `basis` untuk memilih cara baca laporan pada Project Profitability.
- Export CSV/XLSX dan download PDF.

### General Ledger

General Ledger mendukung filter `project_id`. Filter ini membatasi opening dan period rows ke transaksi dengan proyek tersebut. Gunakan GL setelah Project Profitability jika admin perlu menjelaskan baris sumber dari angka income/expense/profit.

### Laporan Lain

Laporan lain dapat tetap terdampak transaksi yang sama secara total, tetapi tidak semua menyediakan filter proyek. Untuk presentasi proyek, fokuskan narasi pada Project Profitability, General Ledger, dan detail akun. Jangan menjanjikan filter proyek pada laporan yang belum terlihat dari kode.

## 9. Contoh Data Awal Untuk Presentasi

Gunakan data yang sederhana dan konsisten agar demo mudah diikuti.

| Master data | Contoh | Catatan |
| --- | --- | --- |
| Contact customer | RS UMMI | Dipakai pada proyek implementasi dan invoice. |
| Contact vendor | PT Konsultan Implementasi | Dipakai pada bill vendor. |
| Proyek utama | `PRJ-ERP-UMMI - Implementasi ERP RS UMMI` | Nama memuat kode karena field `code` khusus tidak tersedia. |
| Estimasi biaya | 150.000.000 | Isi ke `cost_estimate`. |
| Deadline | 2026-03-31 | Isi ke `deadline`. |
| Status awal | `active` | Status default dan paling cocok untuk demo. |
| Invoice jasa | 250.000.000 | Assign ke proyek, lalu deliver. |
| Expense operasional | 10.000.000 | Assign proyek pada baris kategori. |
| Bill vendor | 80.000.000 | Assign ke proyek, lalu open. |
| Manual Journal | Koreksi income/expense proyek | Isi proyek pada baris yang relevan. |

Pastikan akun yang dibutuhkan modul sudah tersedia sebelum demo: Accounts Receivable untuk invoice/payment receive, Accounts Payable untuk bill/bill payment, Tax Payable jika memakai pajak, akun kas/bank, akun income, dan akun expense.

## 10. Contoh Alur Demo Proyek

1. Buka `Projects -> New Project`.
2. Buat proyek `PRJ-ERP-UMMI - Implementasi ERP RS UMMI` dengan contact RS UMMI, deadline 2026-03-31, cost estimate 150.000.000, dan status `active`.
3. Buka detail proyek dan tunjukkan field utama.
4. Buat invoice jasa untuk RS UMMI, pilih proyek tersebut, lalu deliver.
5. Buat expense operasional, pilih proyek pada baris kategori biaya, lalu publish.
6. Buat bill vendor, pilih proyek pada header bill, lalu open.
7. Opsional: buat payment receive atau bill payment untuk menunjukkan ringkasan proyek dari dokumen alokasi, sambil menjelaskan bahwa payment tidak menerima proyek langsung.
8. Buka `Reports -> Project Profitability`, pilih rentang tanggal yang memuat transaksi, lalu filter proyek.
9. Export CSV/PDF jika ingin menunjukkan output presentasi.
10. Buka `Reports -> General Ledger`, pilih proyek yang sama, lalu tunjukkan baris GL sumber.
11. Ubah filter tanggal atau proyek untuk menunjukkan angka berubah sesuai transaksi yang membawa `project_id`.

## 11. Error Umum dan Cara Menghindari

| Masalah | Penyebab umum | Cara menghindari |
| --- | --- | --- |
| Menu Proyek tidak muncul | Role tidak punya `project.view` atau `project.create` | Cek role dan permission dari Settings/Users/Roles. |
| Laporan Profit Proyek tidak bisa dibuka | User tidak punya `report-project-profitability.view` | Berikan permission report yang sesuai. |
| Proyek tidak muncul pada dropdown transaksi | Data proyek tidak tersedia di daftar page, proyek soft-deleted, atau filter/search membatasi tampilan | Pastikan proyek tersimpan dan belum dihapus. |
| Hapus proyek gagal | Proyek sudah dipakai transaksi, item entry, expense category, journal entry, atau GL | Jangan hapus proyek yang sudah dipakai; ubah status menjadi `completed` atau `cancelled`. |
| Project Profitability kosong | Transaksi belum diposting, tanggal report salah, atau `project_id` tidak sampai ke GL | Post transaksi dan cek General Ledger dengan filter proyek. |
| Expense proyek tidak muncul sebagai expense proyek | Project hanya diisi di header expense, bukan pada baris kategori | Isi proyek pada baris kategori biaya. |
| Payment Receive/Bill Payment menolak `project_id` | Request payment memang melarang input proyek langsung | Isi proyek di invoice/bill sumber, bukan di payment. |
| Validasi project gagal | `project_id` tidak valid atau proyek sudah soft-deleted | Pilih proyek dari dropdown resmi. |
| Cost estimate ditolak | Nilai negatif atau melebihi batas validasi | Gunakan angka 0 sampai `9999999999.99999`. |
| Tanggal ditolak | Format tanggal tidak valid | Pakai DatePicker pada form. |

## 12. Checklist Setelah Setup Proyek

- Role admin memiliki permission proyek dan laporan yang dibutuhkan.
- Contact customer/vendor yang dipakai demo sudah tersedia.
- Proyek utama sudah dibuat dengan status `active`.
- Proyek internal yang tidak punya contact sengaja dibiarkan kosong pada `contact_id`.
- Required account untuk invoice, bill, payment, tax, income, expense, cash/bank sudah tersedia.
- Invoice demo sudah delivered jika ingin muncul di GL/laporan.
- Bill demo sudah opened jika ingin muncul di GL/laporan.
- Expense demo sudah published, dan proyek diisi pada baris kategori.
- Manual Journal demo sudah published, dan proyek diisi pada baris yang relevan.
- Rentang tanggal laporan mencakup tanggal transaksi.
- Project Profitability dan General Ledger sudah dicek sebelum presentasi.

## 13. Checklist Presentasi/Demo

- Mulai dari tujuan proyek sebagai dimensi analisis, bukan modul akuntansi terpisah.
- Tunjukkan route/menu `Projects` dan `New Project`.
- Buat satu proyek dengan data yang mudah dikenali.
- Tunjukkan field `name`, `contact`, `deadline`, `cost_estimate`, dan `status`.
- Tunjukkan delete guard secara naratif, tidak perlu menghapus data live.
- Buat atau buka invoice yang membawa proyek dan sudah delivered.
- Buat atau buka expense yang membawa proyek pada baris kategori dan sudah published.
- Buat atau buka bill yang membawa proyek dan sudah opened.
- Jelaskan bahwa payment receive/bill payment tidak menerima proyek langsung.
- Buka Project Profitability dengan filter proyek dan tanggal yang benar.
- Buka General Ledger dengan filter proyek untuk drill-down.
- Tutup demo dengan daftar area yang belum terverifikasi agar ekspektasi pengguna jelas.

## 14. Catatan Field/Menu Yang Belum Terverifikasi

- Field `code`, `start_date`, `end_date`, `budget`, `description`, dan `is_active` belum menjadi field Proyek pada kode yang diaudit.
- Tidak ditemukan fitur session/current project switch yang memfilter seluruh aplikasi berdasarkan satu proyek aktif.
- Dropdown proyek pada transaksi berasal dari daftar proyek page; tidak terverifikasi bahwa status non-active otomatis disembunyikan dari semua dropdown.
- Payment Receive dan Bill Payment tidak memiliki input proyek langsung dan request API melarang `project_id` langsung.
- Banking, Cashflow, Inventory Adjustment, Warehouse Transfer, dan fitur warehouse lain belum terverifikasi sebagai area input proyek.
- Line-level project pada item invoice/bill tidak terlihat sebagai input UI; dukungan yang diaudit adalah project pada header invoice/bill.
- Detail UI landed cost tidak diaudit sebagai materi presentasi utama; yang terverifikasi adalah posting service dapat memakai proyek dari target bill.
- Screenshot untuk materi presentasi belum dibuat pada phase ini.
- Belum terverifikasi dari kode pada phase ini apakah semua report selain Project Profitability, General Ledger, dan detail akun memiliki kebutuhan filter proyek yang setara.
