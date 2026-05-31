# Presentasi Admin Pembelian

Dokumen ini adalah panduan presentasi dan pengisian awal menu **Pembelian** untuk role admin/superadmin. Isinya disusun dari audit sidebar, route Laravel, halaman Inertia React, Form Request, service transaksi, model status, PDF/mail AP-side, attachment, landed cost, dan laporan yang tersedia pada kode saat ini.

Rujukan silang:

- Vendor dan data kontak: [admin-kontak.md](admin-kontak.md)
- Barang/jasa purchasable dan stok: [admin-barang-jasa.md](admin-barang-jasa.md)
- Akun utang, kas/bank, pajak, dan jurnal: [admin-keuangan.md](admin-keuangan.md)
- Akun bank dan transaksi banking: [admin-perbankan.md](admin-perbankan.md)
- Biaya langsung: [admin-biaya.md](admin-biaya.md)
- Proyek dan profitability: [admin-proyek.md](admin-proyek.md)
- Laporan: [admin-laporan.md](admin-laporan.md)

## 1. Tujuan Dokumen

Panduan ini membantu admin/superadmin:

1. memahami menu Pembelian yang benar-benar ada di sidebar dan route;
2. membedakan Bill, Bill Payment, Vendor Credit, refund vendor credit, dan Expense;
3. menyiapkan vendor, item, akun, pajak, proyek, dan data demo sebelum transaksi;
4. menjalankan alur demo pembelian dari tagihan vendor sampai pembayaran dan koreksi;
5. menjelaskan dampak pembelian ke utang usaha, kas/bank, pajak, proyek, stok, dan laporan;
6. menandai batas audit agar presenter tidak menjanjikan field/menu yang belum terverifikasi.

Dokumen ini tidak menggantikan SOP purchasing dan accounting perusahaan. Admin tetap perlu memastikan akun, periode transaksi, pajak, bukti dokumen, dan hak akses sesuai kebijakan organisasi.

## 2. Gambaran Umum Menu Pembelian

Sidebar **Purchases** menampilkan tiga sub menu utama:

| Sub menu sidebar | Istilah dokumen | Kegunaan utama | Route halaman | Permission |
| --- | --- | --- | --- | --- |
| Bills | Tagihan / Bill | Mencatat invoice dari vendor dan mengakui utang usaha saat dibuka. | `/finance/bills` | `bill.view` |
| Vendor Credits | Kredit Vendor / Vendor Credit | Mencatat kredit dari vendor untuk mengurangi bill atau dicatat sebagai refund dana masuk. | `/finance/vendor-credits` | `vendor-credit.view` |
| Payments Made | Bill Payment / Pembayaran Tagihan | Mencatat pembayaran kepada vendor untuk satu atau beberapa bill. | `/finance/bill-payments` | `bill-payment.view` |

Shortcut pembuatan dokumen:

| Shortcut sidebar | Route | Permission |
| --- | --- | --- |
| New Purchase Invoice | `/finance/bills/create` | `bill.create` |
| New Vendor Credit | `/finance/vendor-credits/create` | `vendor-credit.create` |
| New Payment Made | `/finance/bill-payments/create` | `bill-payment.create` |

Catatan route:

- Tidak ditemukan route web bernama `purchases.*`; area pembelian transaksi memakai prefix `/finance/...`.
- Tidak ditemukan route web `payments-made.*`; label sidebar **Payments Made** mengarah ke route aktual `/finance/bill-payments`.
- Tidak ditemukan route web `vendors.*`; master vendor dikelola dari Kontak dengan query `/settings/contacts?type=vendor`.

## 3. Urutan Alur Pembelian Yang Disarankan

Urutan setup:

1. Buat vendor di **Contacts > Vendors**.
2. Buat akun Accounts Payable, kas/bank, akun biaya/HPP/persediaan, dan Tax Payable jika memakai pajak.
3. Buat currency dan tax rate yang diperlukan.
4. Buat item yang dapat dibeli (`is_purchasable`) untuk Bill dan Vendor Credit.
5. Jika transaksi perlu dianalisis per pekerjaan, buat proyek terlebih dahulu.
6. Jika demo stok diperlukan, siapkan item inventory dan stok awal melalui Penyesuaian Persediaan.

Urutan transaksi AP normal:

1. Buat Bill sebagai Draft.
2. Periksa vendor, tanggal, due date, item, nilai, pajak, dan project.
3. Open Bill agar utang usaha diakui.
4. Catat Bill Payment ketika perusahaan membayar vendor.
5. Jika ada retur, potongan dari vendor, atau koreksi tagihan, buat Vendor Credit.
6. Open Vendor Credit.
7. Apply Vendor Credit ke Bill, atau catat Refund jika vendor mengembalikan uang.
8. Buka laporan AP, GL, pajak, item, dan proyek yang relevan.

Urutan transaksi pengeluaran langsung:

1. Gunakan Expense jika uang sudah keluar langsung dari kas/bank dan tidak perlu membentuk utang.
2. Gunakan Bill jika perusahaan menerima tagihan dari vendor dan pembayarannya menyusul.

## 4. Perbedaan Tagihan, Bill Payment, Vendor Credit, Refund Vendor, dan Biaya

| Dokumen/aksi | Dipakai kapan | Dampak utama | Bukan untuk |
| --- | --- | --- | --- |
| Bill | Vendor mengirim invoice/tagihan dan perusahaan belum tentu langsung membayar. | Saat Open, mengakui utang usaha (AP). | Pengeluaran tunai yang sudah langsung dibayar tanpa AP. |
| Bill Payment | Perusahaan membayar satu atau beberapa bill vendor. | Mengurangi AP dan mengurangi kas/bank. | Membuat tagihan baru atau membeli item baru. |
| Vendor Credit | Vendor memberi kredit karena retur, diskon, koreksi, atau kelebihan tagih. | Mengurangi AP dan nilai biaya/pajak terkait saat Open. | Mencatat pembayaran biasa ke vendor. |
| Apply Vendor Credit | Kredit vendor dipakai untuk mengurangi bill yang masih outstanding. | Mengurangi saldo bill dan sisa kredit vendor. | Menerima uang dari vendor. |
| Refund Vendor Credit | Vendor mengembalikan dana atas kredit yang masih tersisa. | Menambah kas/bank dan mengurangi sisa kredit vendor. | Mengurangi bill tanpa uang masuk. |
| Expense / Biaya | Pengeluaran sudah dibayar langsung dari kas/bank. | Saat Published, debit biaya dan credit akun pembayaran. Tidak melewati AP seperti Bill. | Tagihan vendor yang perlu dilacak sebagai utang. |

Prinsip praktis:

- Jika vendor menagih dan belum dibayar, gunakan **Bill**.
- Jika bill sudah dibayar, gunakan **Bill Payment**.
- Jika vendor mengurangi tagihan, gunakan **Vendor Credit** lalu **Apply To Bill**.
- Jika vendor mengembalikan uang, gunakan **Refund Vendor Credit**.
- Jika transaksi langsung dibayar tanpa utang, gunakan **Expense**.

## 5. Sub Menu/Area Pembelian Dalam Sistem

| Area | Halaman aktual | Form Request / aksi | Service utama | Permission utama | Status audit |
| --- | --- | --- | --- | --- | --- |
| Bill | index, create, edit, show, print, PDF | store, update, open, duplicate, delete, bulk delete | `BillService` | `bill.view/create/edit/delete` | Terverifikasi |
| Bill Payment | index, create, edit, show, PDF route | store, update, delete, bulk delete | `BillPaymentService` | `bill-payment.view/create/edit/delete` | Terverifikasi |
| Vendor Credit | index, create/edit, show, PDF route | store, update, open, apply, refund, delete refund, delete, bulk delete | `VendorCreditService` | `vendor-credit.view/create/edit/delete/refund` | Terverifikasi |
| Landed Cost | Bill detail -> Allocate Landed Cost | allocate, delete allocation | `LandedCostService` | `bill.edit`, `bill.delete` untuk delete allocation | Terverifikasi |
| Attachment | Panel pada detail Bill, Bill Payment, Vendor Credit | upload, download, detach | `DocumentService` | view/edit sesuai owner document | Terverifikasi |
| AP API v1 | `/api/v1/bills`, `/api/v1/bill-payments`, `/api/v1/vendor-credits` | JSON/PDF/helper/mail sesuai resource | API v1 controllers/services | permission resource terkait | Terverifikasi |
| Laporan AP | Payables Aging, Purchases by Items, Vendor Balance Summary, Transactions by Contact/Reference, GL, Tax Summary | page, export, PDF untuk report yang didukung | `ReportService`, `ReportExportService` | permission report granular | Terverifikasi |

Prasyarat utama dari route create:

| Area | Prasyarat yang dicek route |
| --- | --- |
| Bill | Minimal satu vendor, currency tersedia, Accounts Payable ada, dan minimal satu akun Expense/Other Expense/COGS tersedia. |
| Bill Payment | Accounts Payable ada, minimal satu akun pembayaran aktif Cash/Bank/Other Current Asset, dan minimal satu bill terbuka/outstanding. |
| Vendor Credit | Minimal satu vendor, currency tersedia, dan Accounts Payable ada. |
| Landed Cost | Bill target harus sudah Open; source harus Bill/Expense yang memiliki baris bertanda landed cost dan sisa alokasi. |

## 6. Tagihan / Bill

### Kegunaan

Bill adalah invoice dari vendor. Dokumen ini dibuat sebagai Draft, lalu di-Open ketika tagihan sudah siap diakui. Saat Open, service membuat GL: credit Accounts Payable, debit akun biaya/HPP/persediaan per baris jika `cost_account_id` tersedia, dan debit Tax Payable jika ada pajak.

Bill juga menjadi sumber utama untuk Payables Aging, Vendor Balance Summary, Purchases by Items, dan alur Bill Payment.

### Daftar Input

| Field UI/API | Wajib/Opsional | Validasi atau perilaku aktual | Fungsi awam | Contoh |
| --- | --- | --- | --- | --- |
| Vendor | Wajib | `vendor_id` harus ada di contacts. Route create mengambil kontak bertipe vendor. | Pemasok yang mengirim tagihan. | `CV Maju Supplies` |
| Bill Number | Opsional di request, terisi default di UI | Maksimal 50 karakter dan unik untuk data aktif. Service default `BILL-000001`. | Nomor tagihan internal. | `BILL-0001` |
| Bill Date | Wajib | Tanggal valid. | Tanggal tagihan vendor. | `2026-06-01` |
| Due Date | Wajib | Tidak boleh sebelum Bill Date. | Tanggal jatuh tempo pembayaran. | `2026-06-30` |
| Reference No | Opsional | Maksimal 50 karakter. | Nomor invoice vendor/PO/referensi eksternal. | `INV-VDR-0601` |
| Currency | Opsional | Mengacu ke currency terdaftar; UI dapat mengisi dari vendor. | Mata uang tagihan. | `IDR` |
| Exchange Rate | Opsional backend | Harus lebih besar dari 0 jika dikirim. Input UI khusus belum terverifikasi. | Kurs transaksi. | `1` |
| Project | Opsional | Project aktif dapat dipilih. | Menghubungkan tagihan ke proyek. | `PRJ-RS-001` |
| Item | Opsional per baris | Dropdown memakai item purchasable. | Barang/jasa yang dibeli. | `Barang A` |
| Description | Opsional per baris | Maksimal 1.000 karakter. | Keterangan item. | `Pembelian Barang A` |
| Quantity | Wajib per baris | Lebih besar dari 0. | Jumlah dibeli. | `5` |
| Rate | Wajib per baris | Minimal 0. | Harga beli per unit. | `800000` |
| Discount | Opsional per baris/header backend | Minimal 0; tipe `percentage` atau `amount`. | Potongan tagihan. | `5%` |
| Tax | Opsional per baris | Tax rate terdaftar; nilai pajak disimpan di baris. | Pajak pembelian. | `PPN 11%` |
| Landed Cost checkbox | Opsional per baris Bill | Hanya muncul di form Bill. Untuk item inventory checkbox disabled. | Menandai baris sebagai sumber landed cost yang dapat dialokasikan nanti. | Ongkir impor |
| Note | Opsional | Maksimal 2.000 karakter. | Catatan internal. | `Pembelian stok demo.` |
| Warehouse, Branch, Inclusive Tax, Adjustment | Opsional backend | Ada di request/model, tetapi input UI utama belum terverifikasi pada phase ini. | Dimensi atau pengaturan tambahan. | - |

Form harus mengirim minimal satu baris entry. Saat item dipilih, UI mengambil deskripsi pembelian, harga beli, dan pajak pembelian default dari master item jika tersedia. Bill menampilkan stock hint untuk item inventory dengan konteks pembelian, yaitu stok saat ini dan stok setelah draft.

Catatan penting untuk demo: audit `LineItemsTable` belum menemukan input atau autofill `cost_account_id` dari master item ke line item. Service posting GL biaya/HPP hanya membuat baris debit jika `cost_account_id` terbawa. Uji jurnal Bill sebelum presentasi eksternal.

### Status dan Aksi

| Status | Makna | Aksi utama |
| --- | --- | --- |
| Draft | Bill masih disiapkan dan belum membuat GL. | Edit, Open, Duplicate, Delete. |
| Unpaid | Bill sudah Open, belum dibayar, dan belum overdue. | Record Payment, Allocate Landed Cost, PDF, Duplicate. |
| Overdue | Bill sudah Open dan melewati due date. | Record Payment, tindak lanjut vendor, laporan aging. |
| Partially Paid | Sebagian bill sudah dibayar atau dikurangi vendor credit. | Record Payment untuk sisa saldo. |
| Paid | Bill sudah lunas oleh payment dan/atau vendor credit. | Lihat histori pembayaran/kredit. |
| Opened | Backing value tersedia, tetapi service Open awal langsung memilih `unpaid` atau `overdue` berdasarkan due date. | Perlakukan sebagai bill terbuka jika muncul dari data lama/API. |

Aksi yang terverifikasi:

- Save as Draft dan Save and Open dari form create.
- Open dari detail/index untuk Draft.
- Edit hanya saat Draft pada UI.
- Duplicate dari detail/index jika punya `bill.create`.
- Record Payment untuk status `opened`, `unpaid`, `overdue`, atau `partially-paid`.
- Allocate Landed Cost untuk bill non-draft.
- Download PDF dari detail Bill.
- Delete hanya ditawarkan untuk Draft; service menolak delete jika ada payment, vendor credit application, atau landed cost allocation.

### Pengaruh Ke Modul Lain

| Area | Pengaruh setelah Open |
| --- | --- |
| Accounts Payable | Credit akun Accounts Payable sebesar total bill lokal. |
| Biaya/HPP/Persediaan | Debit akun `cost_account_id` per baris jika field tersebut ada. |
| Pajak | Debit akun Tax Payable jika baris memiliki tax amount. |
| Bill Payment | Bill terbuka/outstanding muncul di form Bill Payment. |
| Vendor Credit | Bill terbuka/outstanding dapat menjadi target Apply Vendor Credit. |
| Project | `project_id` header dibawa ke GL Bill. |
| Purchases by Items | Mengambil item entry dari Bill yang sudah Open. |
| Payables Aging | Mengambil Bill terbuka yang masih outstanding. |
| Vendor Balance Summary | Menghitung total billed, paid, credited, dan outstanding dari Bill terbuka. |
| Transactions by Contact | Menampilkan Bill untuk vendor terkait. |
| Landed Cost | Bill dapat menjadi target alokasi dan juga dapat menjadi source jika punya baris landed cost. |

### Contoh Input

| Field | Nilai |
| --- | --- |
| Vendor | `CV Maju Supplies` |
| Bill Number | `BILL-0001` |
| Bill Date | `2026-06-01` |
| Due Date | `2026-06-30` |
| Reference No | `INV-VDR-0601` |
| Project | `PRJ-RS-001` |
| Item | `Barang A` |
| Quantity | `5` |
| Rate | `800000` |
| Tax | `PPN 11%` |
| Note | `Pembelian stok awal demo.` |

Nomor otomatis service memakai format `BILL-000001`. Contoh `BILL-0001` dapat dipakai sebagai nomor demo manual selama belum duplikat.

### Error Umum

| Error/gejala | Penyebab umum | Cara menghindari |
| --- | --- | --- |
| Tidak bisa buka form Bill baru | Vendor, currency, Accounts Payable, atau akun biaya/HPP belum tersedia. | Siapkan master data dan COA terlebih dahulu. |
| Due date ditolak | Due date lebih awal dari bill date. | Isi due date pada atau setelah bill date. |
| Bill number duplikat | Nomor sudah dipakai bill aktif. | Gunakan nomor lain atau biarkan default. |
| Open gagal karena akun AP/pajak tidak ada | Required account belum dibuat. | Buat Accounts Payable dan Tax Payable jika memakai pajak. |
| Open gagal karena periode terkunci | Bill date masuk Transaction Locking scope Purchases. | Gunakan tanggal periode terbuka atau buka lock sesuai otorisasi. |
| Jurnal biaya/HPP tidak lengkap | `cost_account_id` line item tidak terbawa dari UI. | Uji manual hasil GL sebelum demo dan catat batas audit. |
| Delete ditolak | Bill punya payment, vendor credit application, atau landed cost allocation. | Jangan hapus bill yang sudah terkait; buat koreksi bisnis. |
| Item tidak muncul | Item tidak purchasable. | Aktifkan pembelian pada master item. |

### Checklist

- [ ] Vendor sudah dibuat di Contacts > Vendors.
- [ ] Accounts Payable tersedia.
- [ ] Akun biaya/HPP/persediaan dan Tax Payable tersedia sesuai kebutuhan.
- [ ] Item purchasable sudah dibuat.
- [ ] Tanggal bill dan due date benar.
- [ ] Project dipilih jika transaksi perlu dianalisis per proyek.
- [ ] Pajak dan total diperiksa.
- [ ] Jurnal `cost_account_id` diuji sebelum demo.
- [ ] Bill di-Open sebelum dicatat pembayarannya.

## 7. Bill Payment / Pembayaran Tagihan

### Kegunaan

Bill Payment dipakai untuk mencatat pembayaran kepada vendor atas satu atau beberapa Bill. Dokumen ini tidak memiliki lifecycle Draft. Saat dibuat, service langsung membuat GL: debit Accounts Payable dan credit akun pembayaran Cash/Bank/Other Current Asset.

Bill Payment mengurangi `payment_amount` pada Bill. Status Bill dihitung ulang menjadi Partially Paid atau Paid sesuai total payment dan vendor credit.

### Daftar Input

| Field UI/API | Wajib/Opsional | Validasi atau perilaku aktual | Fungsi awam | Contoh |
| --- | --- | --- | --- | --- |
| Vendor | Wajib | Harus kontak vendor. Pada edit, UI mengunci vendor. | Vendor yang dibayar. | `CV Maju Supplies` |
| Payment Account | Wajib | Service menerima akun Cash, Bank, atau Other Current Asset. Route hanya menampilkan akun aktif. | Sumber dana keluar. | `Bank BCA Operasional` |
| Payment Date | Wajib | Tanggal valid dan dicek Transaction Locking Purchases. | Tanggal pembayaran. | `2026-06-10` |
| Payment Number | Opsional | Maksimal 50 karakter di request. | Nomor bukti pembayaran internal. | `BPM-0001` |
| Reference | Opsional | Maksimal 50 karakter. | Referensi transfer/memo. | `TRF-BCA-0610` |
| Statement | Opsional backend/UI detail | Maksimal 2.000 karakter; form create tidak menampilkan textarea khusus selain reference pada area utama yang diaudit. | Catatan pembayaran. | `Pembayaran termin 1.` |
| Payment Method | Opsional backend | Maksimal 50 karakter; input UI belum terverifikasi. | Cara bayar. | `Transfer` |
| Currency / Exchange Rate | Opsional backend | Currency valid dan exchange rate > 0; input UI khusus belum terverifikasi. | Mata uang/kurs pembayaran. | `IDR`, `1` |
| Bill allocation | Wajib minimal satu | Bill harus milik vendor yang dipilih, tidak Draft, currency cocok jika dikirim, dan amount tidak melebihi balance due. | Bill mana saja yang dibayar. | `BILL-0001` |
| Payment Amount per Bill | Wajib | Lebih besar dari 0. Total header harus sama dengan total alokasi. | Nilai yang dibayarkan untuk tiap bill. | `2.220.000` |
| Branch | Opsional backend | Ada di request/model, input UI belum terverifikasi. | Cabang transaksi. | - |

Form Bill Payment dapat dibuka dari menu Payments Made atau tombol Record Payment pada detail Bill. Jika dibuka dari Bill, query `bill_id` dipakai untuk preselect bill dan menurunkan vendor terkait.

### Status dan Aksi

Bill Payment tidak memakai status Draft/Open. Dokumen yang tersimpan langsung memengaruhi GL dan saldo bill.

Aksi yang terverifikasi:

- Create Bill Payment.
- Edit Bill Payment jika belum matched dengan transaksi bank dan periode tidak terkunci.
- Delete satuan dan bulk delete, dengan guard matched bank/locked period.
- Show detail dengan alokasi bill dan ringkasan project turunan dari bill.
- Attachment pada detail.
- PDF route tersedia di web `/finance/bill-payments/{id}/pdf`.
- Mail API v1 tersedia di `/api/v1/bill-payments/{id}/mail`; web detail yang diaudit belum menemukan tombol email/PDF khusus.

### Pengaruh Ke Modul Lain

| Area | Pengaruh |
| --- | --- |
| Bill | Menambah `payment_amount`, mengurangi balance due, dan menghitung ulang status. |
| Accounts Payable | Debit AP sebesar amount lokal. |
| Kas/Bank | Credit akun pembayaran yang dipilih. |
| Banking | Dokumen tidak dapat diedit/dihapus jika sudah matched bank transaction. |
| Project | Tidak ada input project langsung; detail dan index menampilkan project summary turunan dari bill yang dibayar. |
| Transactions by Contact | Menampilkan Bill Payment untuk vendor terkait. |
| General Ledger / Journal Sheet | Menampilkan jurnal BillPayment. |
| Cash Flow | `ReportService` memasukkan BillPayment sebagai sumber operasional. |

### Contoh Input

| Field | Nilai |
| --- | --- |
| Vendor | `CV Maju Supplies` |
| Payment Account | `Bank BCA Operasional` |
| Payment Date | `2026-06-10` |
| Payment Number | `BPM-0001` |
| Reference | `TRF-BCA-0610` |
| Bill | `BILL-0001` |
| Payment Amount | `2.220.000` |

### Error Umum

| Error/gejala | Penyebab umum | Cara menghindari |
| --- | --- | --- |
| Tidak bisa membuka form Payment Made | Accounts Payable, akun pembayaran aktif, atau bill outstanding belum tersedia. | Open Bill dan siapkan akun pembayaran. |
| Vendor dipilih tetapi tidak ada bill | Vendor belum punya Bill terbuka/outstanding. | Pilih vendor yang benar atau Open Bill terlebih dahulu. |
| Total pembayaran tidak valid | Total header tidak sama dengan total alokasi. | Biarkan UI menghitung total dari alokasi. |
| Pembayaran melebihi sisa bill | Amount lebih besar dari balance due. | Gunakan balance due aktual. |
| Bill bukan milik vendor | Alokasi memilih bill vendor lain. | Pilih vendor lebih dulu dan gunakan bill yang tampil dari UI. |
| Edit/delete ditolak | Payment sudah matched di banking atau periode Purchases terkunci. | Ikuti SOP banking dan Transaction Locking. |
| Email gagal | Vendor tidak punya email atau payload penerima tidak valid. | Lengkapi email vendor dan validasi To/Cc/Bcc. |

### Checklist

- [ ] Bill sudah Open dan masih memiliki balance due.
- [ ] Akun pembayaran aktif tersedia.
- [ ] Vendor sesuai dengan bill yang akan dibayar.
- [ ] Payment date masuk periode yang tidak terkunci.
- [ ] Nilai pembayaran tidak melebihi saldo bill.
- [ ] Reference transfer diisi jika perlu rekonsiliasi.
- [ ] Setelah simpan, cek status Bill dan GL AP/kas-bank.

## 8. Vendor Credit / Kredit Vendor

### Kegunaan

Vendor Credit adalah kredit dari vendor untuk mengurangi tagihan, mencatat retur pembelian, diskon setelah tagihan, koreksi harga, atau kelebihan bayar yang diakui sebagai kredit. Dokumen dibuat sebagai Draft lalu di-Open.

Saat Open, service membuat GL: debit Accounts Payable, credit akun biaya/HPP per baris jika `cost_account_id` tersedia, dan credit Tax Payable untuk pembalik pajak pembelian jika ada pajak.

### Daftar Input

| Field UI/API | Wajib/Opsional | Validasi atau perilaku aktual | Fungsi awam | Contoh |
| --- | --- | --- | --- | --- |
| Vendor | Wajib | `vendor_id` harus ada di contacts. Route create mengambil kontak vendor. | Vendor pemberi kredit. | `CV Maju Supplies` |
| Vendor Credit Number | Opsional di request, terisi default UI | Maksimal 50 karakter dan unik untuk data aktif. Default route `VC-00001`. | Nomor kredit vendor. | `VC-0001` |
| Vendor Credit Date | Wajib | Tanggal valid. | Tanggal kredit diterbitkan. | `2026-06-12` |
| Reference No | Opsional | Maksimal 50 karakter. | Nomor retur/koreksi dari vendor. | `RET-VDR-0612` |
| Project | Opsional | Project aktif dapat dipilih. | Analisis koreksi per proyek. | `PRJ-RS-001` |
| Currency | Opsional | Currency terdaftar; UI dapat mengisi dari vendor. | Mata uang kredit. | `IDR` |
| Exchange Rate | Opsional backend | Harus lebih besar dari 0 jika dikirim; input UI khusus belum terverifikasi. | Kurs kredit. | `1` |
| Item | Opsional per baris | Dropdown memakai item purchasable. | Item yang dikoreksi/dikreditkan. | `Barang A` |
| Description | Opsional per baris | Maksimal 1.000 karakter. | Alasan/rincian kredit. | `Retur 1 Barang A` |
| Quantity | Wajib per baris | Lebih besar dari 0. | Jumlah retur/koreksi. | `1` |
| Rate | Wajib per baris | Minimal 0. | Nilai per unit. | `800000` |
| Discount dan Tax | Opsional per baris | Discount minimal 0; pajak valid. | Potongan dan pajak koreksi. | `PPN 11%` |
| Note | Opsional | Maksimal 2.000 karakter. | Catatan internal. | `Retur karena barang tidak sesuai.` |
| Warehouse, Branch, Header Discount, Adjustment | Opsional backend | Ada di request/model, tetapi input UI utama belum terverifikasi pada phase ini. | Dimensi/pengaturan tambahan. | - |

Tidak ada field UI khusus bernama `Reason`. Gunakan Reference No, Description, atau Note untuk menjelaskan alasan kredit vendor.

Catatan penting untuk demo: seperti Bill, audit UI belum menemukan input atau autofill `cost_account_id` dari master item ke baris Vendor Credit. Uji jurnal pembalik biaya/pajak sebelum demo.

### Status dan Aksi

| Status | Makna | Aksi utama |
| --- | --- | --- |
| Draft | Vendor Credit belum diposting. | Edit, Open, Delete. |
| Open | Kredit sudah diposting dan masih punya remaining balance. | Apply To Bills, Refund, attachment. |
| Closed | Seluruh kredit sudah diaplikasikan atau di-refund. | Lihat histori aplikasi/refund. |

Aksi yang terverifikasi:

- Save as Draft dan Save and Open dari form create.
- Edit Draft.
- Open.
- Apply To Bills melalui service dan API; web modal ada, tetapi catatan payload mismatch ada di bagian 17.
- Refund Vendor Credit.
- Delete refund.
- Delete application.
- Attachment pada detail.
- PDF route tersedia di web `/finance/vendor-credits/{id}/pdf`.
- Mail API v1 tersedia di `/api/v1/vendor-credits/{id}/mail`; email hanya dapat dikirim untuk Vendor Credit yang sudah Open.

### Apply To Bill

Apply To Bill memakai kredit vendor yang masih tersedia untuk mengurangi Bill vendor yang sama.

Kontrak backend:

| Field apply | Wajib/Opsional | Fungsi | Contoh |
| --- | --- | --- | --- |
| `applications` | Wajib array minimal 1 | Daftar Bill yang akan dikurangi. | 1 baris |
| `applications.*.bill_id` | Wajib | Bill target. Harus vendor sama, tidak Draft, dan currency cocok jika ada. | `BILL-0001` |
| `applications.*.amount` | Wajib | Nilai kredit yang diterapkan. Tidak boleh melebihi remaining credit atau balance due Bill. | `888000` |

Dampak apply:

- Membuat record `vendor_credit_applied_bills`.
- Menambah `credited_amount` pada Bill target.
- Menambah `invoiced_amount` pada Vendor Credit.
- Menghitung ulang status Bill.
- Tidak membuat GL baru karena GL Vendor Credit sudah dibuat saat Open.

Catatan audit web UI: modal Apply pada halaman Vendor Credit mengirim key `entries`, sedangkan `ApplyVendorCreditRequest` mengharapkan `applications`. Sebelum demo Apply lewat web UI, jalankan uji manual atau perbaiki payload agar tidak gagal validasi.

### Refund Vendor Credit

Refund dipakai jika vendor mengembalikan dana atas sisa kredit.

| Field refund | Wajib/Opsional | Validasi atau perilaku aktual | Fungsi awam | Contoh |
| --- | --- | --- | --- | --- |
| Deposit Account | Wajib | Akun harus Cash, Bank, atau Other Current Asset. | Akun penerima uang refund. | `Bank BCA Operasional` |
| Amount | Wajib | Lebih besar dari 0 dan tidak boleh melebihi remaining credit. | Nilai uang yang diterima dari vendor. | `444000` |
| Date | Wajib | Tanggal valid dan dicek lock Purchases. | Tanggal refund diterima. | `2026-06-15` |
| Reference No | Opsional | Maksimal 50 karakter. | Referensi transfer refund. | `RF-VDR-0615` |
| Description | Opsional backend | Maksimal 1.000 karakter; modal web yang diaudit belum mengirim description. | Catatan refund. | `Refund sisa kredit.` |

Dampak refund:

- Menambah `refunded_amount` pada Vendor Credit.
- Membuat GL: debit akun deposit, credit Accounts Payable.
- Membawa project dari header Vendor Credit ke GL refund.
- Delete refund membalik GL dan mengurangi `refunded_amount`.

### Pengaruh Ke Modul Lain

| Area | Pengaruh |
| --- | --- |
| Accounts Payable | Open Vendor Credit mendebit AP; refund mengkredit AP. |
| Bill | Apply menambah credited amount dan mengurangi balance due. |
| Biaya/HPP/Pajak | Open Vendor Credit membalik biaya/HPP dan pajak jika `cost_account_id`/tax tersedia. |
| Kas/Bank | Refund mendebit akun deposit karena uang masuk dari vendor. |
| Project | Header project dibawa ke GL Vendor Credit dan refund. |
| Transactions by Contact | Menampilkan Vendor Credit untuk vendor terkait. |
| Tax Summary | Vendor Credit masuk sumber pajak pembelian pada service laporan. |

### Contoh Input

| Field | Nilai |
| --- | --- |
| Vendor | `CV Maju Supplies` |
| Vendor Credit Number | `VC-0001` |
| Vendor Credit Date | `2026-06-12` |
| Reference No | `RET-VDR-0612` |
| Project | `PRJ-RS-001` |
| Item | `Barang A` |
| Quantity | `1` |
| Rate | `800000` |
| Tax | `PPN 11%` |
| Note | `Retur 1 Barang A karena barang tidak sesuai.` |
| Apply target | `BILL-0001` |
| Refund account jika ada sisa | `Bank BCA Operasional` |

### Error Umum

| Error/gejala | Penyebab umum | Cara menghindari |
| --- | --- | --- |
| Tidak bisa membuat Vendor Credit | Vendor, currency, atau Accounts Payable belum tersedia. | Siapkan vendor, currency, dan AP. |
| Open gagal | Vendor Credit sudah Open, periode terkunci, atau akun AP/pajak tidak tersedia. | Pakai tanggal terbuka dan siapkan COA. |
| Apply gagal | Vendor Credit masih Draft, amount melebihi remaining credit, Bill target draft/lunas/vendor beda/currency beda. | Open Vendor Credit dan pilih Bill vendor yang sama dengan balance due. |
| Apply web UI gagal validasi | UI mengirim key `entries`, request mengharapkan `applications`. | Uji manual sebelum demo atau gunakan endpoint yang sesuai setelah payload diperbaiki. |
| Refund gagal | Amount melebihi remaining credit atau akun deposit bukan Cash/Bank/Other Current Asset. | Pilih akun deposit aktif yang sesuai dan amount tersedia. |
| Delete ditolak | Vendor Credit punya refund atau application. | Hapus application/refund dulu jika memang perlu koreksi. |
| Email gagal | Vendor tidak punya email atau Vendor Credit masih Draft. | Isi email vendor dan Open dokumen. |

### Checklist

- [ ] Vendor Credit dibuat untuk vendor yang sama dengan Bill target.
- [ ] Accounts Payable dan Tax Payable tersedia.
- [ ] Item purchasable dan pajak diperiksa.
- [ ] Note/reference menjelaskan alasan kredit.
- [ ] Vendor Credit di-Open sebelum Apply atau Refund.
- [ ] Remaining credit dicek sebelum Apply/Refund.
- [ ] Apply tidak melebihi balance due Bill.
- [ ] Refund memakai akun deposit yang benar.
- [ ] Payload Apply web UI diuji sebelum presentasi.

## 9. PDF, Email, Attachment, dan Dokumen Pembelian

| Dokumen | PDF | Email | Attachment | Catatan |
| --- | --- | --- | --- | --- |
| Bill | Web PDF tersedia: `/finance/bills/{id}/pdf`; print route tersedia. | Tidak ditemukan Bill mail route/service/API. Statusnya different-by-design / not production-critical sesuai audit development. | Tersedia dengan owner type `bill`. | Jangan menjanjikan kirim email Bill dari sistem. Bill adalah dokumen inbound dari vendor. |
| Bill Payment | Web PDF route tersedia: `/finance/bill-payments/{id}/pdf`; API v1 PDF juga tersedia. | API v1 mail state/send tersedia di `/api/v1/bill-payments/{id}/mail`, permission `bill-payment.edit`; web detail yang diaudit belum menemukan tombol email. | Tersedia dengan owner type `bill_payment`. | Email dapat attach PDF dan tidak memutasi GL/status/balance. |
| Vendor Credit | Web PDF route tersedia: `/finance/vendor-credits/{id}/pdf`; API v1 PDF juga tersedia. | API v1 mail state/send tersedia di `/api/v1/vendor-credits/{id}/mail`, permission `vendor-credit.edit`; hanya Vendor Credit Open yang sendable; web detail yang diaudit belum menemukan tombol email. | Tersedia dengan owner type `vendor_credit`. | Email dapat attach PDF dan tidak memutasi GL/status/balance. |

Policy attachment yang diaudit menerima ekstensi `pdf`, `jpg`, `jpeg`, `png`, `webp`, `doc`, `docx`, `xls`, `xlsx`, `csv`, dan `txt` sampai 10 MB.

Alamat email vendor diambil dari `contacts.email`, fallback ke `billing_address_email` untuk mail AP-side. Jika vendor tidak memiliki email, mail API mengembalikan domain error 422.

## 10. Landed Cost Jika Tersedia

Landed Cost tersedia dan terhubung ke Bill.

Alur awam:

1. Ada biaya tambahan pembelian, misalnya ongkir impor, asuransi, atau bea masuk.
2. Biaya tambahan itu dicatat sebagai baris landed cost pada Bill lain, atau sebagai Expense category bertanda landed cost.
3. Bill target yang menerima landed cost harus sudah Open.
4. Admin membuka detail Bill target lalu klik **Allocate Landed Cost**.
5. Pilih source type `Bill` atau `Expense`.
6. Pilih source transaction dan source entry yang masih punya remaining amount.
7. Pilih metode alokasi `value` atau `quantity`.
8. Sistem menyarankan pembagian biaya ke baris item target.
9. Simpan alokasi.

Kontrak validasi:

| Field | Wajib/Opsional | Fungsi |
| --- | --- | --- |
| `transaction_type` | Wajib | Source `Bill` atau `Expense`. |
| `transaction_id` | Wajib | ID source transaction. |
| `transaction_entry_id` | Wajib | Baris source yang bertanda landed cost. |
| `allocation_method` | Wajib | `value` atau `quantity`. |
| `description` | Opsional | Catatan alokasi. |
| `items.*.entry_id` | Wajib | Baris item Bill target. |
| `items.*.cost` | Wajib | Nilai landed cost untuk baris target. |

Dampak akuntansi dan stok:

- Membuat record `landed_costs` dan `landed_cost_entries`.
- Menambah `allocated_cost_amount` pada source transaction, source entry, dan target item entry.
- Membuat GL LandedCost: debit akun target item, credit akun biaya source.
- Untuk target item inventory, membuat `inventory_transactions` direction `IN` dengan quantity `0` dan rate sebesar landed cost untuk menaikkan nilai persediaan.
- Delete allocation membalik GL dan inventory transaction landed cost.

Batas audit:

- Landed cost biasa dari Bill/Expense tersedia dari kode phase ini.
- Mutasi stok pembelian biasa dari Bill tanpa Landed Cost belum terverifikasi dari kode pada phase ini.
- Source Bill tidak boleh sama dengan Bill target.

## 11. Pengaruh Pembelian Ke Stok, Pajak, Proyek, dan Laporan

| Area | Pengaruh Pembelian | Status audit |
| --- | --- | --- |
| Vendor/Kontak | Bill, Bill Payment, dan Vendor Credit memakai kontak vendor. | Terverifikasi |
| Barang/Jasa | Bill dan Vendor Credit memakai item purchasable. UI mengisi deskripsi, cost price, dan purchase tax default jika ada. | Terverifikasi |
| Stok | Bill menampilkan stock hint pembelian untuk inventory; Vendor Credit menampilkan stock preview netral. Mutasi stok aktual dari Bill/Vendor Credit biasa belum terverifikasi. | Sebagian terverifikasi |
| Landed Cost | Alokasi landed cost membuat inventory transaction quantity 0 untuk item inventory target. | Terverifikasi |
| Pajak | Bill mendebit Tax Payable; Vendor Credit mengkredit Tax Payable; Tax Summary menghitung pajak pembelian dari Bill, Vendor Credit, dan Expense. | Terverifikasi |
| Proyek | Bill dan Vendor Credit punya project header; GL membawa project. Bill Payment menampilkan project summary dari bill. | Terverifikasi |
| AP | Bill Open menambah AP; Bill Payment dan Vendor Credit mengurangi AP; refund Vendor Credit membuat dana masuk dan credit AP. | Terverifikasi |
| Kas/Bank | Bill Payment mengurangi kas/bank; refund Vendor Credit menambah kas/bank. | Terverifikasi |
| Expense/COGS/Inventory | Posting biaya/HPP Bill/Vendor Credit bergantung pada `cost_account_id` line item. Landed Cost dapat mendebit akun inventory/cost target. | Terverifikasi dengan catatan |
| Payables Aging | Mengambil Bill terbuka yang masih outstanding berdasarkan due date. | Terverifikasi |
| Vendor Balance Summary | Mengambil Bill terbuka untuk total billed, paid, credited, outstanding. | Terverifikasi |
| Purchases by Items | Mengambil item entry dari Bill yang sudah Open. | Terverifikasi |
| Transactions by Contact | Memuat Bill, Bill Payment, dan Vendor Credit untuk vendor. | Terverifikasi |
| Transactions by Reference | Membaca GL rows, termasuk reference type Bill, BillPayment, VendorCredit, RefundVendorCredit, dan LandedCost jika terposting. | Terverifikasi |
| General Ledger / Journal Sheet | Membaca posting GL dari service transaksi pembelian. | Terverifikasi |
| Income Statement / Project Profitability | Terpengaruh oleh GL biaya/HPP dan basis report yang didukung. | Terverifikasi dengan catatan `cost_account_id` |

Untuk presentasi stok, jelaskan dengan hati-hati: stock hint pada form pembelian terverifikasi, landed cost inventory transaction terverifikasi, tetapi mutasi stok aktual dari Bill/Vendor Credit biasa belum dapat diklaim dari audit phase ini.

## 12. Contoh Data Awal Untuk Presentasi

### Vendor

| Nama | Email | Tujuan |
| --- | --- | --- |
| `CV Maju Supplies` | `ap@maju-supplies.example` | Bill, Bill Payment, Vendor Credit, mail AP-side. |
| `PT Logistik Nusantara` | `billing@logistik-nusantara.example` | Source landed cost dari Bill/Expense. |

### Akun

| Akun | Tipe/fungsi | Dipakai untuk |
| --- | --- | --- |
| `Hutang Usaha` | Accounts Payable | Bill Open, Bill Payment, Vendor Credit, refund. |
| `Bank BCA Operasional` | Bank | Sumber Bill Payment dan deposit refund. |
| `Kas Kecil` | Cash | Contoh akun pembayaran lain. |
| `HPP` | Cost Of Goods Sold | Biaya pembelian item. |
| `Beban Pengiriman Import` | Expense | Source landed cost. |
| `Persediaan Barang` | Inventory | Item inventory dan target landed cost. |
| `Pajak Terutang` | Tax Payable | Pajak pembelian dan pembalik pajak. |

### Pajak

| Nama | Rate | Dipakai untuk |
| --- | --- | --- |
| `PPN 11%` | `11` | Bill dan Vendor Credit berpajak. |

### Barang dan Jasa

| Item | Tipe | Purchasable | Cost price | Pajak beli | Catatan |
| --- | --- | ---: | ---: | --- | --- |
| `Barang A` | Inventory | Ya | `800000` | `PPN 11%` | Siapkan stok awal jika demo inventory. |
| `Jasa Instalasi Vendor` | Service | Ya | `500000` | Sesuai kebutuhan | Tidak dilacak stok. |
| `Ongkir Import` | Non-Inventory/Service | Ya | `1500000` | Sesuai kebutuhan | Dapat dicatat sebagai source landed cost jika baris ditandai. |

### Proyek Opsional

| Kode | Nama |
| --- | --- |
| `PRJ-RS-001` | `Implementasi ERP RS Demo` |

### Nomor Dokumen Demo

| Dokumen | Nomor demo manual | Contoh nomor otomatis |
| --- | --- | --- |
| Bill | `BILL-0001` | `BILL-000001` |
| Bill Payment | `BPM-0001` | Jika kosong, tampil fallback ID; tidak ada generator web yang diaudit. |
| Vendor Credit | `VC-0001` | `VC-00001` |
| Refund Reference | `RF-VDR-0615` | Diisi manual. |

## 13. Contoh Alur Demo Pembelian End-to-End

### Skenario A - Bill Dibuka dan Dibayar

1. Buka **Purchases > Bills**.
2. Klik **New Bill**.
3. Pilih vendor `CV Maju Supplies`.
4. Isi `BILL-0001`, Bill Date `2026-06-01`, Due Date `2026-06-30`, Reference `INV-VDR-0601`.
5. Pilih project `PRJ-RS-001` jika demo proyek diperlukan.
6. Tambahkan item `Barang A`, quantity `5`, rate `800000`, tax `PPN 11%`.
7. Simpan sebagai Draft.
8. Buka detail dan klik Open.
9. Periksa status Bill, balance due, dan GL. Verifikasi manual baris biaya/HPP karena `cost_account_id` UI belum terverifikasi.
10. Klik Record Payment.
11. Pilih akun `Bank BCA Operasional`, payment date `2026-06-10`, payment number `BPM-0001`.
12. Alokasikan pembayaran ke `BILL-0001`.
13. Simpan dan buka kembali Bill untuk menunjukkan status/remaining balance.

### Skenario B - Vendor Credit Mengurangi Bill

1. Buka **Purchases > Vendor Credits**.
2. Buat `VC-0001` untuk `CV Maju Supplies`.
3. Isi tanggal `2026-06-12`, reference `RET-VDR-0612`, dan note alasan retur.
4. Tambahkan `Barang A`, quantity `1`, rate `800000`, tax `PPN 11%`.
5. Simpan lalu Open.
6. Apply ke `BILL-0001` jika masih outstanding.
7. Jika memakai web UI current code, uji payload Apply lebih dulu karena audit menemukan mismatch `entries` vs `applications`.
8. Tunjukkan remaining credit dan credited amount pada Bill.

### Skenario C - Refund Vendor Credit

1. Buka Vendor Credit yang sudah Open dan masih punya remaining balance.
2. Klik Refund.
3. Pilih deposit account `Bank BCA Operasional`.
4. Isi amount yang tidak melebihi remaining balance.
5. Isi date `2026-06-15` dan reference `RF-VDR-0615`.
6. Simpan refund.
7. Tunjukkan riwayat refund, saldo remaining credit, dan GL kas/bank.

### Skenario D - Landed Cost

1. Buat atau siapkan source Bill/Expense yang punya baris bertanda landed cost, misalnya `Ongkir Import`.
2. Pastikan source sudah Open/Published dan masih punya remaining landed cost.
3. Buka Bill target `BILL-0001` yang sudah Open.
4. Klik **Allocate Landed Cost**.
5. Pilih source type Bill atau Expense.
6. Pilih source entry `Ongkir Import`.
7. Pilih metode alokasi by value atau by quantity.
8. Simpan alokasi.
9. Tunjukkan tabel existing allocations, GL LandedCost, dan inventory transaction quantity 0 untuk item inventory jika relevan.

### Skenario E - Tinjau Laporan

Setelah transaksi, buka:

1. **Payables Aging** untuk outstanding Bill.
2. **Vendor Balance Summary** untuk saldo vendor.
3. **Purchases by Items** untuk pembelian per item dari Bill Open.
4. **Transactions by Contact** untuk Bill, Bill Payment, dan Vendor Credit vendor.
5. **General Ledger** untuk AP, kas/bank, biaya/HPP, pajak, dan landed cost.
6. **Tax Summary** untuk pajak pembelian.
7. **Project Profitability** jika memakai project.

## 14. Error Umum dan Cara Menghindari

| Kondisi | Dampak | Cara menghindari |
| --- | --- | --- |
| Vendor belum dibuat | Form Bill/Vendor Credit tidak dapat dipakai. | Buat kontak vendor lebih dulu. |
| Accounts Payable belum ada | Bill Open, Bill Payment, dan Vendor Credit dapat gagal. | Buat akun AP di Bagan Akun. |
| Tax Payable belum ada | Transaksi berpajak dapat gagal saat posting. | Siapkan akun Tax Payable sebelum demo PPN. |
| Akun pembayaran tidak aktif/tidak sesuai tipe | Bill Payment atau refund ditolak. | Gunakan akun aktif Cash, Bank, atau Other Current Asset. |
| Item tidak purchasable | Item tidak muncul di Bill/Vendor Credit. | Aktifkan purchasable pada master item. |
| `cost_account_id` tidak terbawa | Jurnal biaya/HPP tidak lengkap. | Uji GL manual sebelum demo dan perbaiki data/payload bila perlu. |
| Bill belum Open | Tidak muncul untuk Bill Payment atau Apply Vendor Credit. | Open Bill sebelum settlement. |
| Pembayaran/kredit melebihi saldo | Service menolak amount. | Gunakan balance due dan remaining credit aktual. |
| Vendor berbeda | Payment/apply ditolak. | Pilih Bill vendor yang sama. |
| Currency berbeda | Payment/apply dapat ditolak. | Pakai currency yang sama untuk dokumen terkait. |
| Periode Purchases terkunci | Open, payment, refund, edit, atau delete ditolak. | Gunakan tanggal periode terbuka atau ubah lock sesuai otorisasi. |
| Payment sudah matched bank | Edit/delete Bill Payment ditolak. | Ikuti SOP banking. |
| Attachment gagal upload | File terlalu besar atau ekstensi tidak didukung. | Gunakan file maksimum 10 MB dengan ekstensi yang didukung. |
| Bill mail dicari di demo | Fitur tidak ada pada current code. | Jelaskan Bill mail different-by-design; gunakan PDF/attachment. |
| Apply Vendor Credit web UI gagal | Payload modal web belum selaras dengan Form Request. | Uji/fix sebelum demo Apply dari web UI. |

## 15. Checklist Sebelum Input Pembelian

- [ ] Role admin/superadmin memiliki permission `bill.*`, `bill-payment.*`, dan `vendor-credit.*` yang diperlukan.
- [ ] Vendor demo tersedia dan email vendor diisi jika akan mendemokan mail AP-side.
- [ ] Accounts Payable tersedia.
- [ ] Akun Bank/Cash aktif tersedia untuk payment dan refund.
- [ ] Akun biaya/HPP/persediaan tersedia.
- [ ] Tax Payable dan tax rate tersedia jika memakai pajak.
- [ ] Item purchasable sudah tersedia.
- [ ] Project tersedia jika ingin analisis proyek.
- [ ] Transaction Locking tidak memblokir tanggal demo.
- [ ] Jurnal Bill/Vendor Credit diuji karena `cost_account_id` line item belum terverifikasi otomatis dari UI.
- [ ] Jika demo Apply Vendor Credit dari web UI, payload sudah diuji manual.
- [ ] File attachment demo berukuran kecil dan tidak berisi data rahasia.

## 16. Checklist Presentasi/Demo

- [ ] Login sebagai admin/superadmin.
- [ ] Tunjukkan sidebar Purchases: Bills, Vendor Credits, Payments Made.
- [ ] Jelaskan bahwa Payments Made route aktualnya Bill Payment.
- [ ] Jelaskan vendor master ada di Contacts > Vendors.
- [ ] Jelaskan perbedaan Bill, Bill Payment, Vendor Credit, Apply, Refund, dan Expense.
- [ ] Buat Bill Draft lalu Open.
- [ ] Tunjukkan AP/balance due dan PDF Bill.
- [ ] Catat Bill Payment dari Bank BCA.
- [ ] Buat Vendor Credit, Open, lalu jelaskan Apply To Bill dan Refund.
- [ ] Tunjukkan attachment untuk dokumen pembelian.
- [ ] Tunjukkan landed cost jika source dan target sudah siap.
- [ ] Buka Payables Aging, Vendor Balance Summary, Purchases by Items, Transactions by Contact, GL, Tax Summary, dan Project Profitability.
- [ ] Sampaikan batas audit yang belum aman dijanjikan.

## 17. Catatan Field/Menu Yang Belum Terverifikasi

Area berikut adalah batas audit, bukan janji fitur:

1. **Bill mail:** tidak ditemukan route/service/API/client untuk email Bill. Statusnya different-by-design / not production-critical pada dokumen development; jangan menjanjikan email Bill.
2. **Tombol email/PDF AP-side di web detail:** PDF route dan API mail untuk Bill Payment/Vendor Credit tersedia, tetapi halaman web detail yang diaudit belum menampilkan tombol email dan belum menampilkan tombol PDF khusus untuk kedua dokumen tersebut.
3. **Apply Vendor Credit web UI:** modal halaman `vendor-credits/show.tsx` mengirim payload `{ entries }`, sedangkan `ApplyVendorCreditRequest` mengharapkan `applications`. Uji/fix sebelum demo Apply lewat web UI.
4. **`cost_account_id` line item:** UI pembelian belum terverifikasi mengisi otomatis atau menampilkan input akun biaya dari master item. Service Bill/Vendor Credit hanya membuat baris biaya/HPP jika field ini ada.
5. **Mutasi stok biasa dari Bill/Vendor Credit:** stock hint UI terverifikasi dan landed cost inventory transaction terverifikasi, tetapi mutasi stok aktual dari dokumen pembelian biasa belum terverifikasi dari kode pada phase ini.
6. **Input exchange rate, warehouse, branch, inclusive tax, header discount, dan adjustment pada Bill UI:** field ada di request/model/state, tetapi kontrol UI utama belum terverifikasi.
7. **Input exchange rate, warehouse, branch, header discount, dan adjustment pada Vendor Credit UI:** field ada di request/model/state, tetapi kontrol UI utama belum terverifikasi.
8. **Bill Payment `payment_method`, `currency_code`, `exchange_rate`, `statement`, dan `branch_id`:** request/model mendukung sebagian field, tetapi kontrol UI utama yang diaudit belum menampilkan semuanya.
9. **Refund Vendor Credit `description`:** request/service mendukung description, tetapi modal web yang diaudit belum mengirim field description.
10. **Filter kontak aktif:** route helper vendor mengambil kontak bertipe vendor, tetapi filter `is_active` belum terverifikasi.
11. **Route `purchases.*`, `payments-made.*`, dan `vendors.*`:** tidak ditemukan dari route audit. Gunakan route aktual `/finance/bills`, `/finance/bill-payments`, `/finance/vendor-credits`, dan `/settings/contacts?type=vendor`.
12. **Permissions Apply/Delete Application web:** route web apply berada dalam middleware `vendor-credit.create`, sedangkan Form Request apply mengizinkan `vendor-credit.edit`; uji role kustom sebelum demo.
13. **Bill Payment number generator web:** tidak ditemukan generator nomor otomatis khusus pada form web; jika kosong, tampilan dapat memakai fallback ID.
14. **Purchases by Items:** report mengambil item entry dari Bill yang sudah Open; tidak mencakup Vendor Credit atau Expense dari audit service.
15. **Transactions by Contact:** report vendor mencakup Bill, Bill Payment, dan Vendor Credit; tidak mencakup Expense payee.
16. **Screenshot/deck visual:** belum dibuat pada phase ini. Dokumen ini adalah panduan Markdown hasil audit kode.

Sebelum demo eksternal, jalankan satu transaksi contoh di environment demo dan cocokkan saldo AP, kas/bank, pajak, GL, project attribution, landed cost, dan laporan yang akan ditampilkan.
