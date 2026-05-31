# Presentasi Admin Kontak

Dokumen ini disusun dari audit kode menu Kontak, route, Form Request, service, model, halaman Inertia, dan laporan terkait. Tujuannya adalah membantu super admin/admin menjelaskan fungsi Kontak tanpa mengklaim field atau menu yang tidak terlihat di implementasi saat ini.

## 1. Tujuan Dokumen

Dokumen ini dipakai sebagai bahan presentasi dan demo menu Kontak untuk admin. Fokusnya adalah:

- Menjelaskan peran kontak sebagai master data pelanggan dan vendor.
- Menjelaskan field input yang benar-benar ada di form Kontak.
- Menjelaskan dampak kontak ke transaksi penjualan, pembelian, biaya, proyek, jurnal, banking, dan laporan.
- Menyediakan contoh data awal dan alur demo yang aman untuk ditunjukkan.
- Menandai area yang belum terverifikasi agar tidak dijadikan janji fitur saat presentasi.

Dokumen terkait:

- [Presentasi admin Preferensi](admin-preferensi.md)
- [Presentasi admin Keuangan](admin-keuangan.md)
- [Presentasi admin Perbankan](admin-perbankan.md)
- [Presentasi admin Barang/Jasa](admin-barang-jasa.md)
- [Presentasi admin Penjualan](admin-penjualan.md)
- [Presentasi admin Pembelian](admin-pembelian.md)
- [Presentasi admin Biaya](admin-biaya.md)
- [Presentasi admin Proyek](admin-proyek.md)
- [Presentasi admin Laporan](admin-laporan.md)

## 2. Gambaran Umum Menu Kontak

Menu Kontak adalah master data untuk pihak eksternal yang dipakai dalam transaksi. Di sidebar, area `Contacts` memiliki dua entry:

| Entry sidebar | URL aktual | Route/page aktual | Keterangan |
| --- | --- | --- | --- |
| Customers | `/settings/contacts?type=customer` | `settings.contacts` - `resources/js/pages/settings/contacts.tsx` | Menampilkan daftar kontak dengan filter UI pelanggan. |
| Vendors | `/settings/contacts?type=vendor` | `settings.contacts` - `resources/js/pages/settings/contacts.tsx` | Menampilkan daftar kontak dengan filter UI vendor. |
| Detail kontak | `/settings/contacts/{id}` | `settings.contacts.show` - `resources/js/pages/settings/contacts/show.tsx` | Menampilkan detail read-only satu kontak. |

Tidak ada route web terpisah bernama `customers.*` atau `vendors.*`. Pelanggan dan vendor dikelola dari tabel dan halaman Kontak yang sama, dibedakan oleh field `contact_service`.

Hak akses utama:

| Permission | Dampak |
| --- | --- |
| `contact.view` | Melihat daftar, filter Customers/Vendors, dan detail kontak. |
| `contact.create` | Membuat kontak baru melalui form di halaman daftar. |
| `contact.edit` | Mengubah kontak dari action tabel. |
| `contact.delete` | Menghapus kontak satuan atau bulk delete. |

Endpoint web/API yang dipakai halaman:

| Operasi | Endpoint | Catatan |
| --- | --- | --- |
| Create | `POST /api/contacts` | Diproteksi `contact.create` di route web. |
| Update | `PUT /api/contacts/{id}` | Diproteksi `contact.edit` di route web. |
| Delete | `DELETE /api/contacts/{id}` | Diproteksi `contact.delete` di route web. |
| Bulk delete | `DELETE /api/contacts` | Diproteksi `contact.delete` di route web. |

Fitur daftar yang terverifikasi:

- Filter tipe: All, Customer, Vendor.
- Pagination client-side dengan rows per page `5`, `10`, `20`, `50`, `100`.
- Action View, Edit, Delete sesuai permission.
- Bulk delete dengan checkbox jika user punya `contact.delete`.
- Tidak ditemukan import/export khusus untuk master Kontak.

## 3. Perbedaan Pelanggan dan Vendor

Kontak mempunyai field `contact_service` dengan nilai enum:

| Nilai | Makna bisnis | Dipakai pada |
| --- | --- | --- |
| `customer` | Pelanggan atau pihak yang membeli barang/jasa dari perusahaan. | Estimasi, Faktur, Sale Receipt, Payment Receive, Credit Note, laporan piutang/pelanggan. |
| `vendor` | Vendor/pemasok atau pihak yang menerima pembayaran dari perusahaan. | Bill, Bill Payment, Vendor Credit, Expense payee, laporan utang/vendor. |

Audit kode saat ini hanya mendukung satu nilai `contact_service` per record: `customer` atau `vendor`. Tidak ada nilai enum `both`. Jadi satu record kontak tidak dapat secara eksplisit menjadi customer dan vendor sekaligus. Jika satu pihak bisnis harus dipakai di sisi penjualan dan pembelian, untuk demo gunakan dua record yang mudah dibedakan, misalnya `PT Nusantara Retail - Customer` dan `PT Nusantara Retail - Vendor`, atau jelaskan bahwa dukungan satu record multi-role belum tersedia.

Field `contact_type` berbeda dari `contact_service`. Di UI field ini berisi `business` atau `individual`, tetapi audit `StoreContactRequest` dan `UpdateContactRequest` belum menemukan rule validasi untuk field tersebut. Artinya nilai yang dikirim dari UI belum terverifikasi tersimpan; database punya default `business`.

## 4. Kapan Kontak Perlu Dibuat

Buat kontak sebelum transaksi jika:

- Akan membuat Estimasi, Faktur, Sale Receipt, Payment Receive, atau Credit Note untuk pelanggan.
- Akan membuat Bill, Bill Payment, Vendor Credit, atau Expense dengan payee vendor.
- Akan menghubungkan proyek ke pihak eksternal.
- Akan menjalankan laporan berbasis kontak seperti Transactions by Contact, Customer Balance Summary, Vendor Balance Summary, Receivables Aging, atau Payables Aging.

Kontak tidak perlu dibuat jika:

- Transaksi banking review hanya memakai teks `payee` tanpa relasi ke master kontak.
- Jurnal manual yang dibuat dari UI tidak membutuhkan pilihan kontak. Backend mendukung `entries.*.contact_id`, tetapi field UI jurnal belum terverifikasi tersedia.
- Data pihak eksternal hanya akan dipakai sebagai catatan bebas di deskripsi transaksi.

## 5. Urutan Penggunaan Yang Disarankan

Urutan setup yang disarankan untuk presentasi:

1. Pastikan currency dan organisasi sudah siap di Preferensi.
2. Buat minimal satu kontak `customer` untuk alur penjualan.
3. Buat minimal satu kontak `vendor` untuk alur pembelian dan biaya.
4. Isi email jika akan mendemokan pengiriman dokumen.
5. Isi alamat penagihan dan pengiriman untuk menunjukkan detail kontak di halaman show.
6. Buat transaksi contoh: Estimasi/Faktur untuk customer, Bill/Expense untuk vendor.
7. Buka laporan kontak untuk membuktikan relasi transaksi muncul di laporan.

Untuk demo yang aman, jangan mengandalkan `opening_balance` sebagai sumber saldo laporan. Laporan saldo/aging yang diaudit menghitung dari invoice dan bill, bukan dari metadata `contacts.opening_balance`.

## 6. Sub Menu/Area Kontak Dalam Sistem

Area yang terverifikasi:

| Area | Fungsi | Catatan demo |
| --- | --- | --- |
| Customers | Membuka `/settings/contacts?type=customer`. | Filter pelanggan memakai query `type=customer`, lalu UI memfilter `contact_service`. |
| Vendors | Membuka `/settings/contacts?type=vendor`. | Filter vendor memakai query `type=vendor`, lalu UI memfilter `contact_service`. |
| Form add/edit | Form inline di halaman daftar. | Tidak ada halaman `/settings/contacts/create` atau `/settings/contacts/{id}/edit`. |
| Detail kontak | Halaman read-only `/settings/contacts/{id}`. | Menampilkan basic information, balance, billing address, shipping address, dan notes jika ada. |
| Delete satuan | Action Delete pada tabel. | Diblokir jika kontak punya dokumen sales/AP tertentu. |
| Bulk delete | Checkbox tabel dan tombol Delete Selected. | Kontak yang punya transaksi dilewati, kontak aman dihapus. |

Area yang tidak ditemukan sebagai fitur master Kontak:

- Import kontak.
- Export kontak.
- Route Customers/Vendors terpisah.
- Field tax number/NPWP, payment terms, credit limit, atau dedicated contact person list.

## 7. Daftar Input Kontak

### Kegunaan Setiap Field

| Field UI/API | Kegunaan | Sumber audit |
| --- | --- | --- |
| `contact_service` | Menentukan role transaksi: `customer` atau `vendor`. | Enum `ContactType`, form Kontak, Form Request. |
| `contact_type` | UI membedakan `business` dan `individual`. | Ada di UI/model/migration; belum ada di rules Form Request. |
| `display_name` | Nama utama yang tampil di dropdown transaksi, tabel, detail, dan laporan. | Form, model, request, report join. |
| `salutation`, `first_name`, `last_name` | Identitas personal jika kontak adalah individu atau punya nama orang. | Form dan detail kontak. |
| `company_name` | Nama perusahaan/legal entity. | Form dan detail kontak. |
| `email` | Email utama kontak; juga dipakai modul mail dokumen jika tersedia. | Form, detail, service mail dokumen. |
| `work_phone`, `personal_phone` | Nomor telepon kerja dan personal. | Form dan detail. |
| `website` | Website kontak. | Form dan detail. |
| `currency_code` | Currency preferensi kontak; default UI memakai `settings.base_currency` atau currency pertama. | Form, request, model. |
| `opening_balance` | Metadata saldo awal kontak. | Form, request, model. |
| `opening_balance_at` | Tanggal saldo awal kontak. | Form, request, model. |
| `opening_balance_exchange_rate` | Kurs saldo awal, default `1`. | Form, request, model. |
| Billing address fields | Alamat, email, dan telepon penagihan. | Form dan detail. |
| Shipping address fields | Alamat, email, dan telepon pengiriman. | Form dan detail. |
| `note` | Catatan internal kontak. | Form dan detail. |
| `is_active` | Status aktif/nonaktif administratif. | Form, model, detail. |

### Wajib/Opsional

| Field | Wajib/Opsional | Validasi utama |
| --- | --- | --- |
| `contact_service` | Wajib | Enum `customer` atau `vendor`. |
| `display_name` | Wajib | String maksimal 255. |
| `contact_type` | Wajib di UI; belum terverifikasi di backend | UI opsi `business`/`individual`; tidak ada rule pada Form Request kontak saat audit. |
| `salutation` | Opsional | String maksimal 50. |
| `first_name`, `last_name`, `company_name` | Opsional | String maksimal 255. |
| `email` | Opsional | Format email, maksimal 255. |
| `work_phone`, `personal_phone` | Opsional | String maksimal 50. |
| `website` | Opsional | String maksimal 255. |
| `currency_code` | Opsional | String maksimal 4, harus ada di tabel `currencies`. |
| `opening_balance` | Opsional | Numeric, minimal 0, default 0. |
| `opening_balance_at` | Opsional | Date. |
| `opening_balance_exchange_rate` | Opsional | Numeric lebih besar dari 0, default 1. |
| Billing/shipping address line, city, state, country | Opsional | String maksimal 255. |
| Billing/shipping postcode | Opsional | String maksimal 30. |
| Billing/shipping email | Opsional | Format email, maksimal 255. |
| Billing/shipping phone | Opsional | String maksimal 50. |
| `note` | Opsional | String maksimal 2000. |
| `is_active` | Opsional | Boolean. |

Catatan audit:

- `opening_balance_branch_id` ada di model/migration, tetapi tidak ditemukan sebagai input form Kontak dan tidak ada rule Form Request Kontak.
- `balance` tampil di tabel/detail sebagai saldo saat ini, tetapi tidak diedit langsung dari form.
- `is_active` tersimpan, tetapi route dropdown transaksi yang diaudit belum memfilter `is_active`; jangan presentasikan status inactive sebagai pengunci pilihan transaksi sampai perilakunya diselaraskan.

### Contoh Input

Contoh customer:

| Field | Contoh |
| --- | --- |
| Type | `Customer` |
| Contact type | `Business` |
| Display name | `PT Sinar Retail` |
| Company | `PT Sinar Retail Indonesia` |
| Email | `billing@sinar-retail.example` |
| Work phone | `021-555-0101` |
| Currency | `IDR` |
| Opening balance | `0` |
| Billing address | `Jl. Merdeka 10, Jakarta, Indonesia` |
| Shipping address | `Gudang Sinar Retail, Bekasi, Indonesia` |
| Note | `Customer demo untuk alur estimasi, faktur, dan penerimaan pembayaran.` |

Contoh vendor:

| Field | Contoh |
| --- | --- |
| Type | `Vendor` |
| Contact type | `Business` |
| Display name | `CV Maju Supplies` |
| Company | `CV Maju Supplies` |
| Email | `ap@maju-supplies.example` |
| Work phone | `022-555-0202` |
| Currency | `IDR` |
| Opening balance | `0` |
| Billing address | `Jl. Industri 21, Bandung, Indonesia` |
| Shipping address | `Jl. Industri 21, Bandung, Indonesia` |
| Note | `Vendor demo untuk bill, bill payment, vendor credit, dan expense.` |

## 8. Pengaruh Kontak Ke Transaksi Penjualan

### Estimasi

Estimasi memakai `customer_id` yang mengarah ke kontak dengan `contact_service = customer`. Halaman create Estimasi memeriksa minimal satu customer belum terhapus; jika tidak ada, user diarahkan ke `settings.contacts` dengan pesan error.

Dampak untuk demo:

- Customer wajib dipilih.
- Nama customer muncul di daftar, detail, PDF/mail dokumen yang relevan, dan laporan Transactions by Contact.
- Estimasi yang belum berubah menjadi invoice tetap bisa ditampilkan dalam Transactions by Contact? Audit `ReportService::transactionsByContact` tidak memasukkan sale estimates, sehingga jangan klaim estimasi muncul di laporan tersebut.

### Faktur

Faktur memakai `customer_id` ke kontak customer. Create Faktur juga memeriksa minimal satu customer belum terhapus.

Dampak untuk demo:

- Customer wajib.
- Email customer penting jika ingin mendemokan mail invoice.
- Saat invoice delivered dan belum lunas, nilainya berpengaruh ke Customer Balance Summary dan Receivables Aging.
- GL posting dari invoice mencatat `contact_id` customer pada baris terkait.

### Penerimaan Pembayaran/Penerimaan Penjualan

Payment Receive memakai `customer_id` dan daftar invoice yang belum lunas difilter berdasarkan customer terpilih. Sale Receipt juga memakai customer, walaupun prompt ini berfokus pada Payment Receive.

Dampak untuk demo:

- Pilih customer lebih dulu agar daftar invoice pembayaran relevan.
- Pembayaran menurunkan outstanding invoice.
- Payment Receive muncul di Transactions by Contact.
- Email customer dibutuhkan jika ingin mengirim dokumen pembayaran lewat fitur mail.

### Nota Kredit

Credit Note memakai `customer_id`. Create Credit Note memeriksa minimal satu customer belum terhapus.

Dampak untuk demo:

- Customer wajib.
- Credit Note dapat diaplikasikan ke invoice customer yang sama.
- Credit Note mempengaruhi credited amount invoice dan outstanding customer.
- Credit Note muncul di Transactions by Contact.

## 9. Pengaruh Kontak Ke Transaksi Pembelian

### Tagihan

Bill memakai `vendor_id` yang mengarah ke kontak dengan `contact_service = vendor`. Halaman create Bill memeriksa minimal satu vendor belum terhapus; jika tidak ada, user diarahkan ke `settings.contacts`.

Dampak untuk demo:

- Vendor wajib.
- Bill yang opened dan belum lunas mempengaruhi Vendor Balance Summary dan Payables Aging.
- GL posting dari bill mencatat `contact_id` vendor pada baris terkait.

### Bill Payment

Bill Payment memakai `vendor_id`. Daftar bill yang dapat dibayar difilter berdasarkan vendor terpilih.

Dampak untuk demo:

- Pilih vendor lebih dulu agar daftar unpaid bills relevan.
- Payment menurunkan outstanding bill.
- Bill Payment muncul di Transactions by Contact.
- Email vendor dibutuhkan jika ingin mengirim dokumen pembayaran vendor lewat fitur mail.

### Vendor Credit

Vendor Credit memakai `vendor_id`. Create Vendor Credit memeriksa minimal satu vendor belum terhapus.

Dampak untuk demo:

- Vendor wajib.
- Vendor Credit dapat diaplikasikan ke bill vendor yang sama.
- Vendor Credit mempengaruhi credited amount bill dan outstanding vendor.
- Vendor Credit muncul di Transactions by Contact.

### Biaya

Expense memakai `payee_id` opsional yang mengarah ke kontak. UI create/edit Expense mengambil payee dari kontak vendor.

Dampak untuk demo:

- Payee vendor tidak wajib; expense tetap dapat dibuat tanpa kontak.
- Jika payee dipilih, GL expense mencatat `contact_id` dari `payee_id`.
- Expense langsung berpengaruh ke laporan laba rugi/cash flow sesuai posting expense, tetapi tidak termasuk dalam `ReportService::transactionsByContact` yang diaudit.

## 10. Pengaruh Kontak Ke Laporan

### Transactions by Contact

Route:

- Page: `/reports/transactions-by-contact`
- Export: `/reports/transactions-by-contact/export`
- PDF: `/reports/transactions-by-contact/pdf`

Filter yang terverifikasi:

- `contact_id` wajib saat menjalankan report/export/PDF.
- `from_date` wajib.
- `to_date` wajib dan harus setelah atau sama dengan `from_date`.
- Export mendukung `csv` dan `xlsx`; PDF tersedia.

Sumber transaksi yang diaudit:

| Tipe kontak | Transaksi yang masuk |
| --- | --- |
| Customer | Invoice, Payment Receive, Credit Note. |
| Vendor | Bill, Bill Payment, Vendor Credit. |

Transaksi yang tidak masuk ke report ini dari audit service:

- Sale Estimate.
- Sale Receipt.
- Expense.
- Manual Journal.
- Banking review rows.

### Customer Balance

Route:

- Page: `/reports/customer-balance-summary`
- Export: `/reports/customer-balance-summary/export`
- PDF: `/reports/customer-balance-summary/pdf`

Laporan ini menghitung invoice customer yang sudah delivered sampai `as_of_date`. Kolom utama berasal dari total invoiced, paid, credited, dan outstanding balance. `contacts.opening_balance` tidak dipakai sebagai sumber saldo laporan ini.

### Vendor Balance

Route:

- Page: `/reports/vendor-balance-summary`
- Export: `/reports/vendor-balance-summary/export`
- PDF: `/reports/vendor-balance-summary/pdf`

Laporan ini menghitung bill vendor yang sudah opened sampai `as_of_date`. Kolom utama berasal dari total billed, paid, credited, dan outstanding balance. `contacts.opening_balance` tidak dipakai sebagai sumber saldo laporan ini.

### Aging Piutang/Utang

Route piutang:

- Page: `/reports/receivables-aging`
- Export: `/reports/receivables-aging/export`
- PDF: `/reports/receivables-aging/pdf`

Route utang:

- Page: `/reports/payables-aging`
- Export: `/reports/payables-aging/export`
- PDF: `/reports/payables-aging/pdf`

Receivables Aging menghitung invoice delivered yang masih outstanding berdasarkan customer. Payables Aging menghitung bill opened yang masih outstanding berdasarkan vendor. Bucket yang terverifikasi: current, 1-30, 31-60, 61-90, dan 90+ hari lewat jatuh tempo.

Catatan audit:

- Service report mendukung parameter `customerId`/`vendorId`.
- UI web aging yang diaudit hanya menyediakan filter tanggal, tidak menyediakan dropdown customer/vendor.

## 11. Contoh Data Awal Untuk Presentasi

Gunakan data sederhana berikut agar demo mudah diikuti:

| Nama | Role | Email | Tujuan demo |
| --- | --- | --- | --- |
| `PT Sinar Retail` | Customer | `billing@sinar-retail.example` | Estimasi, faktur, payment receive, credit note, Customer Balance, Receivables Aging. |
| `CV Maju Supplies` | Vendor | `ap@maju-supplies.example` | Bill, bill payment, vendor credit, expense payee, Vendor Balance, Payables Aging. |
| `PT Proyek Nusantara` | Customer | `finance@proyek-nusantara.example` | Kontak proyek dan transaksi berbasis proyek. |
| `CV Jasa Lapangan` | Vendor | `admin@jasa-lapangan.example` | Expense payee dan vendor credit sederhana. |

Minimal data transaksi untuk presentasi:

| Modul | Data contoh | Tujuan |
| --- | --- | --- |
| Estimate | Estimasi untuk `PT Sinar Retail` | Menunjukkan customer wajib di sales flow. |
| Invoice | Invoice delivered untuk `PT Sinar Retail` | Menunjukkan piutang dan aging. |
| Payment Receive | Pembayaran parsial invoice | Menunjukkan saldo outstanding berkurang. |
| Credit Note | Credit note customer yang diaplikasikan ke invoice | Menunjukkan penyesuaian piutang. |
| Bill | Bill opened dari `CV Maju Supplies` | Menunjukkan utang dan aging. |
| Bill Payment | Pembayaran parsial bill | Menunjukkan saldo vendor berkurang. |
| Vendor Credit | Vendor credit yang diaplikasikan ke bill | Menunjukkan penyesuaian utang. |
| Expense | Expense dengan payee `CV Maju Supplies` | Menunjukkan kontak vendor sebagai payee biaya. |
| Project | Proyek dengan contact `PT Proyek Nusantara` | Menunjukkan kontak sebagai relasi non-transaksi utama proyek. |

## 12. Contoh Alur Demo Kontak

Alur demo 1 - buat customer dan faktur:

1. Buka Contacts > Customers.
2. Klik Add Contact.
3. Isi `display_name`, `email`, currency, billing address, dan shipping address.
4. Simpan.
5. Buka detail kontak untuk menunjukkan data read-only.
6. Buka Finance > Invoices > New Invoice.
7. Pilih customer yang baru dibuat.
8. Buat invoice draft, lalu deliver jika akun prasyarat siap.
9. Buka Reports > Customer Balance Summary dan Receivables Aging.

Alur demo 2 - buat vendor dan bill:

1. Buka Contacts > Vendors.
2. Klik Add Contact.
3. Isi vendor demo dengan email dan alamat.
4. Simpan.
5. Buka Finance > Bills > New Bill.
6. Pilih vendor yang baru dibuat.
7. Buat bill draft, lalu open jika akun prasyarat siap.
8. Buka Reports > Vendor Balance Summary dan Payables Aging.

Alur demo 3 - Transactions by Contact:

1. Siapkan satu customer dengan invoice dan payment receive.
2. Buka Reports > Transactions by Contact.
3. Pilih kontak customer.
4. Isi from/to date yang mencakup invoice dan payment.
5. Jalankan report.
6. Tunjukkan baris Invoice, Payment Receive, dan total.
7. Ulangi dengan vendor untuk Bill, Bill Payment, dan Vendor Credit.

Alur demo 4 - delete guard:

1. Buat kontak demo yang belum dipakai transaksi.
2. Hapus kontak tersebut; penghapusan harus berhasil.
3. Pilih kontak yang sudah punya invoice/bill.
4. Coba hapus; sistem menolak dengan pesan bahwa kontak punya dokumen terkait.
5. Jelaskan bahwa kontak yang sudah dipakai transaksi sebaiknya tidak dihapus.

## 13. Error Umum dan Cara Menghindari

| Error/gejala | Penyebab yang terverifikasi | Cara menghindari |
| --- | --- | --- |
| Tidak bisa membuat Estimasi/Faktur/Credit Note/Sale Receipt | Belum ada kontak `customer`. | Buat minimal satu customer di Contacts > Customers. |
| Tidak bisa membuat Bill/Vendor Credit | Belum ada kontak `vendor`. | Buat minimal satu vendor di Contacts > Vendors. |
| Tidak bisa kirim dokumen lewat mail | Email customer/vendor kosong. | Isi field `email` pada kontak sebelum demo mail. |
| Delete kontak gagal | Kontak punya invoice, bill, estimate, credit note, vendor credit, payment receive, atau bill payment. | Jangan hapus kontak yang sudah punya transaksi; gunakan data demo baru untuk delete. |
| Bulk delete tidak menghapus semua pilihan | Service melewati kontak yang punya dokumen terkait. | Jelaskan hasil bulk delete: deleted dan skipped. |
| Field `contact_type` terlihat berubah di UI tetapi hasilnya tidak jelas | Form Request belum memiliki rule `contact_type`. | Jangan jadikan field ini poin utama demo sampai backend diselaraskan. |
| Aging tidak bisa difilter per kontak di UI | UI aging hanya memiliki filter tanggal. | Gunakan Transactions by Contact untuk filter satu kontak; gunakan aging untuk ringkasan semua kontak. |
| Saldo awal kontak tidak muncul di Customer/Vendor Balance Summary | Report menghitung invoice/bill, bukan `contacts.opening_balance`. | Gunakan transaksi invoice/bill pembuka bila ingin saldo tampil di report tersebut. |

## 14. Checklist Setelah Setup Kontak

- Minimal satu customer aktif dibuat.
- Minimal satu vendor aktif dibuat.
- Display name mudah dibedakan di dropdown.
- Email customer/vendor diisi jika demo mail dokumen akan dilakukan.
- Currency kontak sesuai base currency atau skenario multi-currency.
- Billing address dan shipping address diisi jika detail kontak akan ditunjukkan.
- Tidak ada data asli, secret, atau alamat pribadi yang dipakai dalam demo.
- Customer sudah dipakai pada satu invoice delivered jika ingin demo Customer Balance/Aging.
- Vendor sudah dipakai pada satu bill opened jika ingin demo Vendor Balance/Aging.
- Kontak demo tanpa transaksi tersedia jika ingin demo delete berhasil.

## 15. Checklist Presentasi/Demo

- Buka sidebar Contacts dan tunjukkan Customers serta Vendors.
- Jelaskan bahwa keduanya memakai halaman yang sama dengan filter role.
- Tunjukkan Add Contact dan field wajib `Type` serta `Display Name`.
- Jelaskan perbedaan `customer` dan `vendor` dari dampaknya ke transaksi.
- Tunjukkan detail kontak read-only.
- Tunjukkan transaksi sales yang memakai customer.
- Tunjukkan transaksi purchase/expense yang memakai vendor.
- Tunjukkan Transactions by Contact untuk customer dan vendor.
- Tunjukkan Customer Balance Summary dan Vendor Balance Summary.
- Tunjukkan Receivables Aging dan Payables Aging sebagai ringkasan outstanding per kontak.
- Tunjukkan delete guard dengan kontak yang sudah punya transaksi.
- Tutup dengan catatan area yang belum terverifikasi agar ekspektasi user tetap akurat.

## 16. Catatan Field/Menu Yang Belum Terverifikasi

Catatan berikut harus disampaikan sebagai batasan audit, bukan sebagai fitur yang sudah siap:

- Tidak ada route web `customers.*` atau `vendors.*`; sidebar Customers/Vendors mengarah ke `/settings/contacts` dengan query type.
- Tidak ditemukan import/export khusus untuk master Kontak.
- Tidak ditemukan field tax number/NPWP, payment terms, credit limit, atau daftar contact person pada form Kontak.
- Tidak ditemukan nilai `contact_service = both`; satu record kontak hanya `customer` atau `vendor`.
- Field `contact_type` ada di UI/model/migration, tetapi belum ada di `StoreContactRequest` dan `UpdateContactRequest`.
- Field `opening_balance_branch_id` ada di model/migration, tetapi tidak ditemukan di form Kontak dan tidak ada di Form Request Kontak.
- `is_active` ada di form/model, tetapi route dropdown transaksi yang diaudit belum memfilter kontak aktif.
- Banking review memakai teks `payee`, bukan relasi `contact_id` ke master Kontak.
- Backend jurnal manual mendukung `entries.*.contact_id`, tetapi UI jurnal yang diaudit belum menampilkan field kontak.
- UI Receivables Aging dan Payables Aging belum menyediakan filter customer/vendor walaupun service backend mendukung parameter tersebut.
- `contacts.opening_balance` tidak dipakai oleh Customer Balance Summary, Vendor Balance Summary, Receivables Aging, atau Payables Aging yang diaudit.
