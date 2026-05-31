# Presentasi Admin Perbankan

Dokumen ini adalah panduan presentasi untuk super admin/admin ketika menjelaskan menu Perbankan. Fokusnya adalah penggunaan praktis: apa fungsi submenu, data apa yang terlihat atau perlu diisi, bagaimana transaksi bank memengaruhi cashflow dan General Ledger, serta batas fitur yang belum boleh diklaim.

Audit phase ini membaca route, sidebar, controller, form request, service, model, migration, halaman React, locale, dan dokumen pengembangan yang terkait dengan Banking/Cashflow. Jika ada field yang umum ditemui pada aplikasi perbankan tetapi tidak ditemukan pada kode ERP ini, field tersebut ditulis di bagian catatan belum terverifikasi.

## 1. Tujuan Dokumen

Panduan ini dipakai untuk:

| Tujuan | Penjelasan |
| --- | --- |
| Menjelaskan menu Perbankan ke orang awam | Presenter dapat menjelaskan hubungan rekening bank, mutasi bank, pencocokan, dan posting akuntansi tanpa masuk ke teknis database. |
| Menyiapkan data demo | Admin tahu akun bank mana yang harus disiapkan dari Bagan Akun sebelum membuka Perbankan. |
| Menghindari klaim fitur berlebih | Dokumen membedakan fitur manual yang sudah ada dari fitur bank feed/Plaid/CSV yang tidak terverifikasi sebagai input manual ERP. |
| Menghubungkan Perbankan ke laporan | Setiap transaksi cashflow yang diposting akan membuat baris di `account_transactions`, sehingga saldo bank dan laporan ikut berubah. |
| Menyediakan checklist presentasi | Presenter punya alur demo, contoh input, error umum, dan validasi manual setelah simpan. |

Audiens utama dokumen ini adalah:

| Audiens | Kebutuhan |
| --- | --- |
| Super admin/admin | Menyiapkan akun kas/bank, rules, dan transaksi review untuk demo. |
| Finance/accounting | Memahami kapan transaksi bank dicocokkan ke dokumen lama dan kapan dibuat sebagai cashflow baru. |
| Presenter | Membawakan narasi banking dengan bahasa sederhana dan contoh nominal realistis. |
| Implementor | Mengetahui route, permission, dan batas fitur aktual dari kode. |

Batasan penting:

- Menu Perbankan tidak membuat rekening bank baru dari tabel khusus `bank_accounts`. Akun bank berasal dari Bagan Akun dengan `account_type` `cash` atau `bank`.
- Integrasi bank feed/Plaid/CSV tidak menjadi bagian input manual yang terverifikasi pada ERP phase ini.
- Bank Rules pada ERP memberi saran autofill untuk kategorisasi. Jangan klaim rule selalu melakukan auto-post tanpa review user.
- Istilah "reconcile" dalam presentasi sebaiknya dijelaskan sebagai pencocokan/matching operasional, karena status review aktual adalah `open`, `matched`, `categorized`, dan `excluded`.

Rujukan silang:

- Bagan Akun untuk membuat akun Kas/Bank: [admin-keuangan.md](admin-keuangan.md)
- Biaya langsung dari bank dan akun pembayaran: [admin-biaya.md](admin-biaya.md)
- Pembayaran customer dan dampak ke piutang: [admin-penjualan.md](admin-penjualan.md)
- Pembayaran vendor dan dampak ke utang: [admin-pembelian.md](admin-pembelian.md)
- Laporan kas, buku besar, dan transaksi detail: [admin-laporan.md](admin-laporan.md)

## 2. Gambaran Umum Menu Perbankan

Menu Perbankan berada di sidebar utama dan dapat dibuka oleh role operasional yang memiliki `cashflow.view`. Aksi tambah, ubah/match, dan hapus mengikuti `cashflow.create`, `cashflow.edit`, dan `cashflow.delete`.

| Submenu UI | URL | Route internal yang diaudit | Permission utama | Fungsi singkat |
| --- | --- | --- | --- | --- |
| Akun Bank | `/banking/accounts` | `banking.accounts.index`, `banking.accounts.show` | `cashflow.view` | Melihat daftar akun kas/bank dari Bagan Akun, saldo GL, jumlah transaksi, dan antrean review. |
| Transaksi Untuk Ditinjau | `/banking/review` | `banking.review.index`, `banking.review.create`, `banking.review.show` | `cashflow.view`, `cashflow.create`, `cashflow.edit`, `cashflow.delete` | Menyimpan mutasi bank manual yang belum diputuskan: cocokkan, kategorikan ke cashflow, exclude, restore, atau hapus bila masih aman. |
| Aturan Bank | `/banking/rules` | `banking.rules.index` | `cashflow.view`, API create/edit/delete memakai `cashflow.create/edit/delete` | Membuat rule untuk memberi saran kategorisasi otomatis pada transaksi review yang cocok. |

Menu tugas baru di grup Perbankan:

| Task | URL | Fungsi |
| --- | --- | --- |
| Add Transaction for Review | `/banking/review/create` | Input mutasi bank mentah secara manual agar masuk antrean review. |
| Add Money In | `/banking/transactions/create?direction=in` | Langsung membuat transaksi cashflow masuk dan posting GL. |
| Add Money Out | `/banking/transactions/create?direction=out` | Langsung membuat transaksi cashflow keluar dan posting GL. |

Route name yang tidak ditemukan saat audit:

| Nama yang dicari | Status |
| --- | --- |
| `banks` | Tidak ada route matching. |
| `bank-accounts` | Tidak ada route matching. Nama aktual: `banking.accounts.*`. |
| `bank-transactions` | Tidak ada route matching. Nama aktual untuk create: `banking.transactions.create`. |
| `bank-rules` | Tidak ada route matching. Nama aktual: `banking.rules.index`. |

Matrix audit ringkas:

| Submenu Perbankan | Route/page aktual | Request/Form | Service | Permission | Status audit |
| --- | --- | --- | --- | --- | --- |
| Akun Bank | `banking/accounts/index`, `banking/accounts/show` | Tidak ada form create akun bank di Banking; akun dibuat dari Account request. | `CashflowService` | `cashflow.view`; pembuatan akun dari COA memakai `account.create`. | Terverifikasi. |
| Transaksi Untuk Ditinjau | `banking/review/index`, `create`, `show` | `StoreUncategorizedBankTransactionRequest`, `CategorizeUncategorizedBankTransactionRequest`, `MatchUncategorizedBankTransactionRequest` | `BankReviewService`, `CashflowService`, `BankRuleService` | `cashflow.view/create/edit/delete`. | Terverifikasi. |
| Aturan Bank | `banking/rules/index` | `StoreBankRuleRequest`, `UpdateBankRuleRequest` | `BankRuleService` | `cashflow.view/create/edit/delete`. | Terverifikasi. |

## 3. Urutan Penggunaan Yang Disarankan

Urutan ini aman untuk demo awal karena mengikuti dependensi aplikasi.

| Urutan | Menu | Yang dilakukan | Alasan |
| --- | --- | --- | --- |
| 1 | Preferensi Umum | Pastikan base currency, format tanggal, dan timezone sudah benar. | Tampilan nominal dan tanggal dipakai di seluruh Perbankan. |
| 2 | Bagan Akun | Buat akun `cash` atau `bank`, misalnya `1002 Bank BCA Operasional`. | Perbankan hanya membaca akun COA bertipe Cash/Bank. |
| 3 | Bagan Akun | Buat akun kontra seperti Pendapatan Bunga, Beban Administrasi Bank, dan akun clearing bila perlu. | Categorize dan Add Money In/Out membutuhkan counter account. |
| 4 | Akun Bank | Buka `/banking/accounts` dan cek akun bank muncul. | Jika tidak muncul, cek apakah tipe akun bukan `cash` atau `bank`. |
| 5 | Transaksi Untuk Ditinjau | Input beberapa mutasi manual untuk review. | Membuat data antrean yang bisa dicocokkan atau dikategorikan saat demo. |
| 6 | Aturan Bank | Buat rule biaya admin atau bunga bank. | Rule akan memberi saran autofill pada transaksi review open yang cocok. |
| 7 | Review Detail | Kategorikan satu transaksi menjadi cashflow dan match satu transaksi ke dokumen existing. | Menjelaskan dua alur utama: buat posting baru atau hubungkan ke posting lama. |
| 8 | Akun Bank Detail | Buka ledger akun bank dan cek saldo berubah setelah cashflow dibuat. | Membuktikan posting masuk ke GL. |
| 9 | Reports | Buka General Ledger atau Cash Flow jika perlu. | Menunjukkan dampak ke laporan. |

Urutan demo singkat 20-30 menit:

| Menit | Aktivitas |
| --- | --- |
| 0-5 | Jelaskan bahwa Perbankan memakai akun Cash/Bank dari Bagan Akun. |
| 5-10 | Tunjukkan Akun Bank dan saldo berjalan. |
| 10-15 | Input transaksi untuk ditinjau: uang masuk, biaya admin, bunga bank. |
| 15-20 | Buat rule biaya admin dan apply suggestion pada review detail. |
| 20-25 | Cocokkan satu mutasi ke transaksi ERP existing bila kandidat match tersedia. |
| 25-30 | Tunjukkan status `categorized`/`matched` dan cek ledger akun bank. |

## 4. Sub Menu Akun Bank

### Kegunaan

Sub menu Akun Bank menampilkan akun kas/bank yang berasal dari Bagan Akun. Halaman ini bukan master rekening bank terpisah. Secara awam, anggap menu ini sebagai "kartu rekening operasional" yang memperlihatkan saldo dari transaksi akuntansi dan jumlah mutasi bank yang masih perlu direview.

Akun Bank digunakan untuk:

| Kebutuhan | Contoh |
| --- | --- |
| Melihat saldo bank/kas | Saldo `Bank BCA Operasional` dihitung dari debit dikurangi credit di `account_transactions`. |
| Melihat aktivitas terakhir | Admin dapat melihat jumlah transaksi dan tanggal transaksi terakhir. |
| Membuka ledger akun | Detail akun menampilkan 100 transaksi terbaru untuk akun tersebut. |
| Masuk ke review transaksi | Jika ada mutasi open, tombol Review Queue membawa user ke filter akun tersebut. |
| Membuat cashflow langsung | Tombol Add Money In/Out membuat transaksi arus kas dan posting GL. |

Perbedaan Akun Bank di Perbankan dan Bagan Akun:

| Area | Penjelasan awam |
| --- | --- |
| Bagan Akun | Tempat membuat nama akun GL, tipe akun, kode akun, dan status aktif. Ini sumber data master. |
| Perbankan > Akun Bank | Tempat melihat akun Cash/Bank yang sudah ada, saldo, ledger, dan antrean review. Ini bukan form pembuatan rekening bank baru. |
| Dampak GL | Semua transaksi tetap masuk ke akun COA yang sama. Jika akun salah tipe, Perbankan tidak akan menganggapnya akun bank. |

### Daftar Input

Tidak ada input create/edit Akun Bank langsung pada halaman Perbankan yang diaudit. Input untuk menyiapkan akun bank berada di Bagan Akun:

| Field | Wajib/Opsional | Validasi dari kode | Fungsi awam | Berpengaruh ke | Contoh input |
| --- | --- | --- | --- | --- | --- |
| `name` | Wajib | String maksimal 255. | Nama akun yang dibaca user. | Dropdown bank, ledger, laporan. | Bank BCA Operasional |
| `code` | Tergantung preferensi Accountant | String maksimal 30; bisa wajib dan unik jika preferensi mengharuskan. | Kode urut akun. | Sorting, identifikasi di dropdown. | 1002 |
| `account_type` | Wajib | Enum `AccountType`; untuk Perbankan harus `cash` atau `bank`. | Menentukan akun ini kas/rekening bank. | Perbankan, laporan, validasi transaksi. | bank |
| `description` | Opsional | String maksimal 1000. | Catatan penggunaan akun. | Referensi admin. | Rekening operasional utama. |
| `parent_account_id` | Opsional | ID akun valid; service menjaga root type dan kedalaman akun. | Menaruh akun di bawah parent. | Struktur COA. | Parent "Kas dan Bank". |
| `currency_code` | Opsional | String maksimal 10 dan harus ada di currencies. | Mata uang akun. | Tampilan uang dan cashflow. | IDR |
| `is_active` | Opsional | Boolean. | Menandai akun masih dipakai. | Dropdown dan transaksi baru. | true |

Kolom yang terlihat pada daftar Akun Bank:

| Kolom | Fungsi |
| --- | --- |
| Account Type | Menunjukkan `cash` atau `bank`. |
| Code dan Name | Identitas akun yang berasal dari COA. |
| Current Balance | Saldo dari `account_transactions`, bukan saldo manual yang diketik di Perbankan. |
| Last Activity | Tanggal transaksi terakhir pada akun tersebut. |
| Entries Count | Jumlah baris transaksi GL untuk akun tersebut. |
| Open Review Count | Jumlah transaksi review yang masih open untuk akun tersebut. |
| Actions | View Ledger, Review Queue, Add Money In/Out, Add Review Item. |

Field yang umum pada rekening bank tetapi tidak ditemukan sebagai input ERP:

| Field umum | Status audit |
| --- | --- |
| Nomor rekening | Belum menjadi input Akun Bank pada Perbankan; bisa ditulis di `description` COA bila perlu referensi internal. |
| Nama pemilik rekening | Belum menjadi input Akun Bank pada Perbankan. |
| Nama bank terpisah dari nama akun | Belum menjadi field khusus; gunakan `name`, misalnya `Bank BCA Operasional`. |
| Opening balance langsung di Banking | Belum menjadi input halaman Banking; saldo awal demo dibuat lewat Jurnal Manual atau kontrak cash-basis opening internal yang guarded, bukan dari form Perbankan. |
| Routing/SWIFT/IBAN/last four | Belum ditemukan sebagai input pada kode ERP. |

### Pengaruh Ke Modul Lain

| Modul/laporan | Dampak Akun Bank |
| --- | --- |
| Payment Receive | Akun kas/bank dipakai sebagai rekening deposit penerimaan customer. |
| Bill Payment | Akun kas/bank dipakai sebagai rekening pembayaran vendor. |
| Sale Receipt | Debit ke akun kas/bank saat transaksi tunai ditutup. |
| Expense | Akun kas/bank dipakai sebagai sumber pembayaran expense. |
| Manual Journal | Akun kas/bank dapat dipakai dalam jurnal setoran modal, transfer, atau koreksi. |
| Banking Review | Transaksi review selalu melekat ke satu akun bank/kas. |
| Cashflow Transaction | Add Money In/Out membuat GL ke akun bank dan counter account. |
| General Ledger | Menampilkan semua baris debit/credit pada akun tersebut. |
| Cash Flow report | Transaksi cashflow yang diposting dapat memengaruhi laporan arus kas sesuai service report. |

### Contoh Input Demo

Siapkan dari Bagan Akun:

| Code | Name | Account Type | Currency | Description |
| --- | --- | --- | --- | --- |
| `1001` | Kas Kecil | `cash` | IDR | Kas operasional harian. |
| `1002` | Bank BCA Operasional | `bank` | IDR | Rekening utama penerimaan dan pembayaran. |
| `1003` | Bank Mandiri Payroll | `bank` | IDR | Rekening pembayaran gaji dan transfer internal. |
| `6002` | Beban Administrasi Bank | `expense` | IDR | Counter account untuk biaya bank. |
| `4003` | Pendapatan Bunga Bank | `other-income` atau `income` sesuai COA | IDR | Counter account untuk bunga bank. |
| `1105` | Clearing Payment Gateway | `other-current-asset` | IDR | Akun sementara settlement QRIS/payment gateway bila digunakan. |

Contoh narasi:

> Akun Bank di sini bukan data rekening bank yang berdiri sendiri. Sistem mengambil akun dari Bagan Akun yang tipenya Cash atau Bank. Karena itu, jika `Bank BCA Operasional` tidak muncul, hal pertama yang dicek adalah tipe akun di COA.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Halaman Akun Bank kosong | Belum ada akun COA bertipe `cash` atau `bank`. | Buat akun di Bagan Akun terlebih dahulu. |
| Akun tidak muncul di Perbankan | Akun dibuat sebagai `other-current-asset` atau tipe lain. | Ubah strategi: buat akun baru bertipe `bank`/`cash`; account type tidak aman diubah sembarangan setelah dipakai transaksi. |
| Add Money In/Out diarahkan ke Bagan Akun | Tidak ada akun cash/bank atau tidak ada counter account. | Siapkan akun bank dan akun kontra. |
| Transfer antar rekening ditolak | Counter account bukan Cash/Bank atau sama dengan akun utama. | Pilih akun bank/kas lain sebagai counter account. |
| Saldo awal tidak sesuai rekening koran | Saldo ditentukan transaksi GL, bukan input manual Banking. | Buat jurnal saldo awal yang benar dan publish. |
| User tidak bisa melihat menu | Tidak punya `cashflow.view`. | Review role dan permission. |

### Checklist

- Minimal satu akun `cash` atau `bank` aktif sudah ada di Bagan Akun.
- Minimal satu counter account selain akun bank utama sudah ada untuk Add Money In/Out.
- Saldo awal demo dibuat melalui Jurnal Manual yang sudah Published.
- Akun BCA/Mandiri/Kas Kecil muncul di `/banking/accounts`.
- Open Review Count sesuai jumlah transaksi review yang belum diselesaikan.
- Ledger akun menampilkan transaksi terbaru dengan inflow/outflow yang masuk akal.
- Jika ingin demo transfer antar rekening, siapkan dua akun Cash/Bank berbeda.

## 5. Sub Menu Transaksi Untuk Ditinjau

### Kegunaan

Transaksi Untuk Ditinjau adalah tempat menampung mutasi bank mentah yang belum diputuskan perlakuannya. Dalam bahasa awam, ini seperti antrean transaksi dari rekening koran yang perlu dijawab:

1. Apakah ini sudah ada transaksi ERP-nya dan hanya perlu dicocokkan?
2. Apakah ini perlu dibuat menjadi transaksi cashflow baru?
3. Apakah ini transaksi duplikat/salah input dan perlu dikecualikan?

Perbedaan konsep:

| Konsep | Penjelasan |
| --- | --- |
| Transaksi bank mentah | Mutasi dari rekening/rekor bank yang masuk tabel `uncategorized_bank_transactions`. |
| Transaksi akuntansi | Posting yang sudah masuk `account_transactions`, misalnya Payment Receive, Bill Payment, Sale Receipt, Manual Journal, atau Cashflow. |
| Match | Review item dihubungkan ke transaksi ERP existing yang jumlahnya harus sama. Tidak membuat GL baru. |
| Categorize | Review item dibuat menjadi `CashflowTransaction` baru dan memposting GL. |
| Exclude | Review item dikeluarkan dari antrean open tanpa membuat posting. Bisa restore. |

Status review aktual:

| Status | Arti |
| --- | --- |
| `open` | Belum dicocokkan, belum dikategorikan, belum dikecualikan. |
| `matched` | Sudah dicocokkan ke satu atau lebih referensi ERP existing. |
| `categorized` | Sudah dibuat menjadi transaksi cashflow. |
| `excluded` | Dikeluarkan dari antrean review. |

### Daftar Input

Input saat membuat transaksi review manual:

| Area/Aksi | Field | Wajib/Opsional | Validasi dari kode | Fungsi awam | Contoh input |
| --- | --- | --- | --- | --- | --- |
| Import/manual transaction | `bank_account_id` | Wajib | Integer, harus ada di `accounts`; service memastikan tipe Cash/Bank. | Rekening tempat mutasi terjadi. | Bank BCA Operasional |
| Import/manual transaction | `date` | Wajib | Date. | Tanggal mutasi bank. | 2026-01-10 |
| Import/manual transaction | `direction` | Wajib | `in` atau `out`. | Uang masuk atau keluar. | in |
| Import/manual transaction | `amount` | Wajib | Numeric lebih dari 0. | Nilai mutasi absolut. Arah out disimpan negatif oleh service. | 1000000 |
| Import/manual transaction | `reference_no` | Opsional | String maksimal 100. | Nomor referensi dari bank. | BCA-TRX-0001 |
| Import/manual transaction | `payee` | Opsional | String maksimal 255. | Nama pihak terkait. | Customer A |
| Import/manual transaction | `description` | Opsional | String maksimal 2000. | Uraian rekening koran. | Transfer Customer A INV-0001 |

Input saat mengategorikan transaksi review menjadi cashflow:

| Area/Aksi | Field | Wajib/Opsional | Validasi dari kode | Fungsi awam | Contoh input |
| --- | --- | --- | --- | --- | --- |
| Categorize | `date` | Wajib | Date. | Tanggal posting cashflow. | 2026-01-10 |
| Categorize | `transaction_type` | Wajib | Enum `CashFlowType`. | Jenis arus kas. | OtherIncome atau OtherExpense |
| Categorize | `credit_account_id` | Wajib | Integer, harus ada di `accounts`; untuk transfer harus Cash/Bank. | Akun lawan posting. | Beban Administrasi Bank |
| Categorize | `reference_no` | Opsional | String maksimal 100. | Referensi bank/dokumen. | ADM-JAN-2026 |
| Categorize | `description` | Opsional | String maksimal 2000. | Catatan posting. | Biaya admin bank Januari |

Input saat matching:

| Area/Aksi | Field | Wajib/Opsional | Validasi dari kode | Fungsi awam | Contoh input |
| --- | --- | --- | --- | --- | --- |
| Match | `matched_references` | Wajib | Array minimal 1. | Daftar transaksi ERP existing yang dipilih. | Payment Receive INV-0001 |
| Match | `matched_references.*.reference_type` | Wajib | String maksimal 100. | Jenis dokumen. | PaymentReceive |
| Match | `matched_references.*.reference_id` | Wajib | Integer minimal 1. | ID dokumen. | 15 |

Kondisi matching penting:

| Aturan | Penjelasan |
| --- | --- |
| Jumlah harus balance | Total kandidat yang dipilih harus sama dengan amount review, toleransi 0.01. |
| Arah harus cocok | Mutasi masuk hanya mencari GL debit pada akun bank; mutasi keluar mencari GL credit. |
| Kandidat yang sudah matched di review lain disaring | Satu referensi ERP tidak ditawarkan lagi bila sudah dipakai review lain. |
| Kandidat cashflow yang sudah berasal dari categorize review lain disaring | Mencegah double link pada cashflow hasil categorize. |

### Pengaruh Ke Modul Lain

| Aksi | Dampak |
| --- | --- |
| Membuat review item | Menambah antrean open; belum mengubah GL. |
| Categorize ke cashflow | Membuat `CashflowTransaction`, membuat dua baris GL di `account_transactions`, mengubah status review menjadi `categorized`. |
| Match ke transaksi existing | Membuat row `matched_bank_transactions`; tidak membuat GL baru. |
| Unmatch | Menghapus snapshot matching dan membuka kembali item review. |
| Uncategorize | Menghapus cashflow dan GL hasil categorize, lalu membuka kembali item review. Ditolak jika periode financial terkunci. |
| Exclude | Mengeluarkan item dari open queue; tidak membuat GL. |
| Restore | Membuka kembali item excluded. |
| Delete | Boleh untuk item open atau excluded yang belum categorized/matched; item matched/categorized ditolak. |

### Contoh Input Demo

| Skenario | Field penting | Contoh |
| --- | --- | --- |
| Uang masuk customer | Direction `in`, amount `1000000`, payee Customer A, description transfer invoice. | `Transfer Customer A INV-0001` |
| Biaya admin bank | Direction `out`, amount `6500`, description `ADMIN BULANAN`. | Nanti dikategorikan ke Beban Administrasi Bank. |
| Transfer antar rekening | Direction `out`, amount `5000000`, description `Transfer ke Mandiri Payroll`. | Kategorikan sebagai `TransferToAccount` dengan counter account Bank Mandiri Payroll. |
| Bunga bank | Direction `in`, amount `2500`, description `BUNGA BANK`. | Kategorikan sebagai `OtherIncome` ke Pendapatan Bunga Bank. |
| Transaksi duplikat | Direction sesuai mutasi, description berisi catatan duplikat. | Exclude bila memang bukan transaksi yang perlu diposting. |

Contoh narasi:

> Mutasi bank mentah belum otomatis menjadi jurnal. Admin harus memilih: cocokkan dengan transaksi ERP yang sudah ada, buat cashflow baru, atau exclude jika bukan transaksi yang perlu dipakai.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Tidak ada transaksi untuk ditinjau | Belum ada input manual review; bank feed/import tidak terverifikasi sebagai flow UI. | Buat Add Transaction for Review. |
| Akun bank ditolak | Akun bukan `cash` atau `bank`. | Pilih akun Cash/Bank dari Bagan Akun. |
| Amount ditolak | Kosong, nol, negatif, atau bukan angka. | Isi nominal positif; arah keluar dipilih dari field direction. |
| Kategorisasi ditolak karena periode terkunci | Tanggal masuk scope Financial yang locked. | Gunakan tanggal demo yang tidak terkunci atau partial unlock sesuai SOP. |
| Transfer ditolak | Counter account transfer bukan Cash/Bank. | Pilih akun bank/kas lain. |
| Match tidak bisa disimpan | Total kandidat tidak sama dengan amount review. | Pilih kandidat yang jumlahnya tepat atau koreksi transaksi existing. |
| Delete ditolak | Item sudah matched atau categorized. | Unmatch atau uncategorize dulu bila memang perlu. |
| Cashflow hasil categorize tidak bisa dihapus dari ledger | Cashflow terhubung ke review item. | Undo categorization dari halaman review detail. |

### Checklist

- Filter status `Open`, `Resolved`, dan `Excluded` menampilkan angka yang masuk akal.
- Filter Bank Account bekerja dan URL memakai query `account_id`.
- Review item baru muncul sebagai `open`.
- Rule suggestion muncul hanya pada item `open` yang cocok.
- Categorize mengubah status menjadi `categorized` dan membuat link ke Cashflow.
- Match mengubah status menjadi `matched` dan menampilkan referensi yang dipilih.
- Exclude mengubah status menjadi `excluded`; Restore mengembalikannya ke open.
- Setelah categorize, buka Akun Bank detail dan cek ledger bertambah.

## 6. Sub Menu Aturan Bank

### Kegunaan

Aturan Bank membantu user mengisi saran kategorisasi untuk transaksi review yang berulang. Contoh paling umum adalah biaya admin bank, bunga bank, settlement QRIS, atau biaya layanan payment gateway.

Cara kerja aktual:

| Langkah | Penjelasan |
| --- | --- |
| Rule dibuat | Admin menentukan kondisi dan hasil kategorisasi yang disarankan. |
| Review detail dibuka | Service mencari rule yang sesuai dengan arah transaksi dan akun bank. |
| Saran muncul | Jika cocok, form categorize terisi transaction type, counter account, dan description. |
| User tetap review | User menekan Apply Suggestion dan submit categorize. |

Jangan klaim rule auto-post penuh. Pada kode yang diaudit, rule menghasilkan suggestion untuk transaksi `open`, bukan posting otomatis tanpa user.

### Daftar Input

| Field | Wajib/Opsional | Validasi dari kode | Fungsi awam | Berpengaruh ke | Contoh input |
| --- | --- | --- | --- | --- | --- |
| `name` | Wajib | String maksimal 255. | Nama aturan. | Daftar rule dan saran review. | Biaya Admin Bank |
| `sort_order` | Wajib | Integer minimal 0. | Prioritas/urutan rule. Angka kecil dievaluasi lebih dulu. | Konflik rule. | 10 |
| `bank_account_id` | Opsional | Nullable integer, exists accounts. | Scope rekening tertentu atau semua rekening. | Matching rule. | Bank BCA Operasional |
| `apply_if_transaction_type` | Wajib | `deposit` atau `withdrawal`. | Rule berlaku untuk uang masuk atau keluar. | Pemilihan rule. | withdrawal |
| `conditions_type` | Wajib | `and` atau `or`. | Semua kondisi harus cocok atau salah satu cukup. | Evaluasi kondisi. | and |
| `assign_transaction_type` | Wajib | Enum `CashFlowType`. | Jenis cashflow yang disarankan. | Categorize form. | OtherExpense |
| `assign_counter_account_id` | Wajib | Integer, exists accounts. | Akun lawan yang disarankan. | GL saat categorize. | Beban Administrasi Bank |
| `assign_description` | Opsional | String maksimal 2000. | Catatan default untuk posting. | Description cashflow. | Biaya admin bank bulanan. |
| `conditions` | Wajib | Array minimal 1. | Syarat rule. | Evaluasi suggestion. | Description contains ADMIN |
| `conditions.*.field` | Wajib | `description`, `payee`, `reference_no`, atau `amount`. | Bagian mutasi yang dicek. | Evaluasi suggestion. | description |
| `conditions.*.comparator` | Wajib | `equals`, `equal`, `contains`, `not_contain`, `bigger`, `bigger_or_equal`, `smaller`, `smaller_or_equal`. | Cara membandingkan. | Evaluasi suggestion. | contains |
| `conditions.*.value` | Wajib | String maksimal 255. | Nilai pembanding. | Evaluasi suggestion. | ADMIN |

Comparator praktis:

| Jenis field | Comparator UI | Contoh |
| --- | --- | --- |
| Text: description/payee/reference | contains, equals, not contain. | Description contains `ADMIN`. |
| Amount | equals, greater than, greater/equal, less than, less/equal. | Amount equals `6500`. |

### Pengaruh Ke Modul Lain

| Area | Dampak |
| --- | --- |
| Review transaksi | Rule dapat memberi suggestion saat transaksi masih open. |
| Cashflow | Jika user menerima suggestion dan submit categorize, cashflow dibuat memakai transaction type dan counter account dari rule. |
| GL/laporan | GL baru dibuat saat categorize, bukan saat rule dibuat. |
| Akun bank | Rule dapat berlaku untuk semua akun atau satu akun tertentu. |
| Audit demo | Sort order membantu menjelaskan kenapa rule spesifik sebaiknya diprioritaskan. |

### Contoh Input Demo

Rule 1: biaya admin bank

| Field | Nilai |
| --- | --- |
| Name | Biaya Admin Bank |
| Sort Order | 10 |
| Bank Account Scope | Bank BCA Operasional |
| Applies To | Money Out / withdrawal |
| Conditions Mode | All conditions |
| Condition | Description contains `ADMIN` |
| Assign Transaction Type | OtherExpense |
| Assign Counter Account | Beban Administrasi Bank |
| Assign Description | Biaya admin bank bulanan. |

Rule 2: bunga bank

| Field | Nilai |
| --- | --- |
| Name | Bunga Bank |
| Sort Order | 20 |
| Bank Account Scope | All Bank Accounts |
| Applies To | Money In / deposit |
| Conditions Mode | Any condition |
| Condition | Description contains `BUNGA` |
| Assign Transaction Type | OtherIncome |
| Assign Counter Account | Pendapatan Bunga Bank |
| Assign Description | Pendapatan bunga bank. |

Rule 3: settlement QRIS/payment gateway

| Field | Nilai |
| --- | --- |
| Name | Settlement QRIS |
| Sort Order | 30 |
| Bank Account Scope | Bank BCA Operasional |
| Applies To | Money In / deposit |
| Conditions Mode | All conditions |
| Condition | Description contains `QRIS` |
| Assign Transaction Type | OtherIncome atau TransferFromAccount sesuai kebijakan akun clearing |
| Assign Counter Account | Clearing Payment Gateway bila akun tersedia |
| Assign Description | Settlement QRIS/payment gateway. |

Catatan demo QRIS:

- Gunakan akun clearing hanya jika COA demo memang punya akun clearing/payment gateway.
- Jika belum ada akun clearing, jelaskan sebagai contoh konsep dan pakai rule bunga/admin yang sudah pasti tersedia.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Rule tidak memberi suggestion | Direction tidak cocok, bank account scope berbeda, atau kondisi tidak match. | Cek `apply_if_transaction_type`, bank account, dan kondisi. |
| Rule terlalu umum | Kondisi seperti `TRANSFER` menangkap banyak mutasi berbeda. | Buat rule lebih spesifik, misalnya `ADMIN`, `BUNGA`, atau kode merchant. |
| Rule yang diharapkan kalah prioritas | Sort order rule lain lebih kecil dan match lebih dulu. | Turunkan angka sort order rule yang lebih spesifik. |
| Form rule ditolak | Name kosong, sort order bukan angka, counter account kosong, atau conditions kosong. | Lengkapi field wajib. |
| Akun target salah tipe | Untuk transfer, counter account harus Cash/Bank; untuk expense/income pilih akun P&L yang benar. | Pilih account type sesuai transaksi. |
| Transaksi sudah `matched` atau `categorized` sebelum rule dibuat | Suggestion hanya muncul untuk transaksi `open`. | Buat review item baru atau reverse status jika memang perlu. |

### Checklist

- Rule tampil di tabel dengan nama dan sort order yang benar.
- Scope akun bank sudah sesuai: spesifik atau semua akun.
- Conditions terbaca jelas oleh presenter.
- Transaction type hasil sesuai direction: uang masuk memakai tipe inflow, uang keluar memakai tipe outflow.
- Counter account benar dan mudah dijelaskan.
- Buat satu review item yang cocok lalu cek suggestion muncul di detail review.
- Jangan gunakan rule terlalu luas pada demo utama.

## 7. Contoh Data Awal Untuk Presentasi

### Akun Bank

| Code | Name | Type | Kegunaan demo |
| --- | --- | --- | --- |
| `1001` | Kas Kecil | `cash` | Menjelaskan cash account jika organisasi punya petty cash. |
| `1002` | Bank BCA Operasional | `bank` | Rekening utama untuk uang masuk/keluar dan review. |
| `1003` | Bank Mandiri Payroll | `bank` | Rekening tujuan transfer internal. |

### Counter Account

| Code | Name | Type | Kegunaan demo |
| --- | --- | --- | --- |
| `3001` | Modal Pemilik | `equity` | Counter untuk setoran modal melalui Add Money In. |
| `4003` | Pendapatan Bunga Bank | `other-income` atau `income` | Counter untuk bunga bank. |
| `6002` | Beban Administrasi Bank | `expense` | Counter untuk biaya admin bank. |
| `1105` | Clearing Payment Gateway | `other-current-asset` | Counter settlement QRIS jika COA mendukung. |

### Transaksi Untuk Ditinjau

| Tanggal | Akun bank | Direction | Amount | Payee | Reference | Description | Rencana aksi |
| --- | --- | --- | ---: | --- | --- | --- | --- |
| 2026-01-10 | Bank BCA Operasional | in | 1000000 | Customer A | BCA-IN-001 | Transfer Customer A INV-0001 | Match ke Payment Receive bila kandidat ada. |
| 2026-01-11 | Bank BCA Operasional | out | 6500 | BCA | ADM-JAN | ADMIN BULANAN | Categorize ke Beban Administrasi Bank. |
| 2026-01-12 | Bank BCA Operasional | out | 5000000 | Internal | TRF-MDR | Transfer ke Mandiri Payroll | Categorize transfer ke Bank Mandiri Payroll. |
| 2026-01-31 | Bank BCA Operasional | in | 2500 | BCA | INT-JAN | BUNGA BANK | Categorize ke Pendapatan Bunga Bank. |

### Aturan Bank

| Rule | Kondisi | Hasil |
| --- | --- | --- |
| Biaya Admin Bank | Withdrawal, description contains `ADMIN`. | OtherExpense ke Beban Administrasi Bank. |
| Bunga Bank | Deposit, description contains `BUNGA`. | OtherIncome ke Pendapatan Bunga Bank. |
| Settlement QRIS | Deposit, description contains `QRIS`. | OtherIncome atau TransferFromAccount ke akun clearing jika tersedia. |

## 8. Contoh Alur Demo Perbankan

### Alur demo 1: dari COA ke Akun Bank

| Langkah | Menu | Aksi | Hasil yang dijelaskan |
| --- | --- | --- | --- |
| 1 | Bagan Akun | Tunjukkan `1002 Bank BCA Operasional` bertipe `bank`. | Akun ini menjadi sumber Akun Bank di Perbankan. |
| 2 | Perbankan > Akun Bank | Buka daftar akun bank. | Akun BCA muncul karena account type benar. |
| 3 | Detail Akun Bank | Buka ledger akun. | Saldo dan transaksi berasal dari GL. |

Narasi singkat:

> Perbankan tidak menyimpan rekening bank terpisah. Ia membaca akun Cash/Bank dari Bagan Akun. Jadi fondasinya tetap double-entry accounting.

### Alur demo 2: biaya admin bank dengan rule

| Langkah | Menu | Aksi | Hasil yang dijelaskan |
| --- | --- | --- | --- |
| 1 | Aturan Bank | Buat rule `Biaya Admin Bank`. | Rule siap memberi suggestion. |
| 2 | Transaksi Untuk Ditinjau | Input mutasi out Rp6.500 dengan description `ADMIN BULANAN`. | Item masuk status open. |
| 3 | Review Detail | Lihat suggestion rule, apply, lalu categorize. | Cashflow dibuat otomatis dari form yang sudah terisi. |
| 4 | Akun Bank Detail | Cek ledger Bank BCA. | Outflow Rp6.500 muncul dan saldo berkurang. |

Narasi singkat:

> Rule mempercepat kerja finance untuk transaksi berulang. Tetapi user tetap memeriksa dan menekan categorize supaya posting tidak terjadi diam-diam.

### Alur demo 3: uang masuk customer dicocokkan

| Langkah | Menu | Aksi | Hasil yang dijelaskan |
| --- | --- | --- | --- |
| 1 | Sales/Finance | Pastikan Payment Receive atau Sale Receipt sudah membuat GL ke Bank BCA. | Ada transaksi ERP existing sebagai kandidat. |
| 2 | Transaksi Untuk Ditinjau | Input mutasi in Rp1.000.000 dari Customer A. | Review item open. |
| 3 | Review Detail | Pilih kandidat match yang amount-nya sama. | Status menjadi matched. |
| 4 | Akun Bank Detail | Cek ledger tidak dobel. | Matching tidak membuat GL baru. |

Narasi singkat:

> Jika uang masuk sudah dicatat di ERP lewat Payment Receive, mutasi bank cukup dicocokkan. Jangan dikategorikan lagi, karena itu akan menggandakan posting.

### Alur demo 4: transaksi duplikat dikecualikan

| Langkah | Menu | Aksi | Hasil yang dijelaskan |
| --- | --- | --- | --- |
| 1 | Transaksi Untuk Ditinjau | Buka item yang ternyata duplikat. | Status masih open. |
| 2 | Review Detail | Klik Exclude. | Item keluar dari antrean open. |
| 3 | Filter Excluded | Tampilkan item excluded. | Item masih bisa direstore. |

Narasi singkat:

> Exclude bukan hapus permanen. Ini dipakai untuk menyembunyikan mutasi yang tidak perlu diproses, tetapi masih bisa dikembalikan jika keputusan berubah.

## 9. Checklist Setelah Setup Menu Perbankan

| Area | Checklist | Status |
| --- | --- | --- |
| COA | Minimal ada satu akun `cash` atau `bank`. |  |
| COA | Minimal ada counter account untuk biaya bank, bunga bank, modal, dan transfer. |  |
| Permission | User demo punya `cashflow.view/create/edit/delete` sesuai aksi. |  |
| Akun Bank | Akun bank muncul di `/banking/accounts`. |  |
| Akun Bank | Saldo awal berasal dari transaksi GL yang benar. |  |
| Review | Minimal ada satu item open untuk uang masuk dan satu item open untuk uang keluar. |  |
| Review | Filter status dan akun bank bekerja. |  |
| Rules | Rule biaya admin dan bunga bank sudah dibuat. |  |
| Rules | Rule dengan kondisi spesifik punya sort order lebih kecil dari rule umum. |  |
| Categorize | Satu transaksi berhasil dikategorikan dan membuat cashflow. |  |
| Match | Satu transaksi berhasil matched ke referensi ERP existing bila kandidat ada. |  |
| Exclude | Satu transaksi excluded bisa direstore. |  |
| Reports | General Ledger atau Cash Flow siap ditampilkan setelah posting. |  |

Validasi cepat sebelum presentasi:

| Tes | Ekspektasi |
| --- | --- |
| Buka Akun Bank tanpa akun Cash/Bank | Sistem mengarahkan setup ke Bagan Akun pada flow create/review. |
| Input review amount 0 | Validasi menolak. |
| Categorize transaksi pada periode locked | Service menolak. |
| Match kandidat dengan jumlah berbeda | Service menolak karena amount tidak balance. |
| Delete review yang sudah matched | Service menolak; unmatch dulu jika perlu. |
| Delete cashflow hasil categorize dari ledger | Service menolak; undo categorization dari review detail. |

## 10. Checklist Presentasi/Demo

### Persiapan user

| Checklist | Catatan |
| --- | --- |
| Login sebagai user role Banking/Cashier atau admin. | Sidebar Perbankan mengikuti permission `cashflow.*`. |
| Role punya `cashflow.view`. | Untuk melihat Akun Bank, Review, dan Rules. |
| Role punya `cashflow.create`. | Untuk Add Money In/Out, Add Transaction for Review, dan create bank rule. |
| Role punya `cashflow.edit`. | Untuk match, unmatch, exclude, restore, update rule. |
| Role punya `cashflow.delete`. | Untuk hapus transaksi cashflow/review/rule yang aman. |

### Persiapan data

| Checklist | Catatan |
| --- | --- |
| Base currency IDR sudah siap. | Rujuk panduan Preferensi. |
| Akun `1002 Bank BCA Operasional` dan `1003 Bank Mandiri Payroll` tersedia. | Tipe harus `bank`. |
| Akun `6002 Beban Administrasi Bank` tersedia. | Tipe expense. |
| Akun `4003 Pendapatan Bunga Bank` tersedia. | Tipe income/other-income sesuai COA. |
| Ada transaksi ERP existing untuk skenario match. | Contoh Payment Receive ke Bank BCA. |
| Tidak ada transaction lock yang menghalangi tanggal demo. | Gunakan tanggal yang aman. |

### Narasi demo

| Urutan narasi | Poin yang harus disebut |
| --- | --- |
| Akun Bank | "Ini daftar akun kas/bank dari Bagan Akun, bukan rekening bank terpisah." |
| Saldo | "Saldo dihitung dari GL, bukan saldo yang diketik manual." |
| Review | "Mutasi bank mentah perlu diputuskan: match, categorize, atau exclude." |
| Match | "Matching tidak membuat jurnal baru; ia menghubungkan mutasi bank ke transaksi yang sudah ada." |
| Categorize | "Categorize membuat cashflow baru dan posting GL." |
| Rules | "Rules memberi saran pengisian, user tetap review sebelum posting." |
| Exclude | "Exclude menyembunyikan item dari antrean open dan bisa restore." |

### Hal yang sebaiknya tidak dilakukan saat demo

| Hindari | Alasan |
| --- | --- |
| Menyebut ada Plaid/bank feed aktif di ERP | Tidak terverifikasi sebagai flow input manual pada kode ERP ini. |
| Mengklaim bank rules auto-post tanpa review | Kode memberi suggestion, bukan posting otomatis penuh. |
| Menggunakan akun non-Cash/Bank sebagai bank account | Service menolak. |
| Mengategorikan uang masuk yang sebenarnya sudah ada Payment Receive | Dapat menggandakan GL. Gunakan match. |
| Menggunakan rule terlalu umum seperti `TRANSFER` | Bisa memberi suggestion salah. |
| Menjelaskan status `reconciled` sebagai status review ERP | Status review aktual adalah `open`, `matched`, `categorized`, `excluded`. |

### Script singkat presenter

1. Buka Bagan Akun dan tunjukkan `1002 Bank BCA Operasional` bertipe `bank`.
2. Buka Perbankan > Akun Bank, jelaskan saldo dan open review count.
3. Buka Aturan Bank, buat atau tunjukkan rule `Biaya Admin Bank`.
4. Buka Transaksi Untuk Ditinjau, tambah mutasi out Rp6.500 dengan description `ADMIN BULANAN`.
5. Buka detail review, apply suggestion, lalu categorize.
6. Buka Akun Bank detail untuk menunjukkan ledger cashflow yang dibuat.
7. Buka satu review uang masuk customer dan match ke Payment Receive jika kandidat tersedia.
8. Tutup dengan perbedaan match dan categorize agar user tidak double posting.

## 11. Catatan Field/Menu Yang Belum Terverifikasi

Catatan ini mencegah presenter mengklaim fitur melebihi hasil audit phase ini. Untuk field/menu yang tidak dapat diklaim dari UI atau kontrak kode yang diaudit, gunakan status: **Belum terverifikasi dari kode pada phase ini**.

| Area | Catatan belum terverifikasi |
| --- | --- |
| Model `BankAccount` khusus | Belum terverifikasi dari kode pada phase ini. Akun Bank memakai model `Account` bertipe `cash` atau `bank`. |
| Input nomor rekening | Belum terverifikasi dari kode pada phase ini sebagai field Perbankan. Tidak ada `account_number` khusus untuk Akun Bank. |
| Input pemilik rekening | Belum terverifikasi dari kode pada phase ini sebagai field Perbankan. |
| `bank_name`, `routing`, `swift`, `iban`, `last_four` | Belum terverifikasi dari kode pada phase ini sebagai input ERP. |
| Opening balance langsung dari halaman Akun Bank | Belum terverifikasi dari kode pada phase ini. Gunakan Jurnal Manual untuk saldo awal demo. |
| Upload bank statement/CSV | Belum terverifikasi dari kode pada phase ini sebagai flow UI Perbankan. |
| Plaid/bank feed aktif di ERP | Belum terverifikasi dari kode pada phase ini. Audit Bigcapital menemukan modul Plaid di referensi Bigcapital, tetapi ERP menandainya out-of-scope/manual workflow. |
| Pending/recognized transactions seperti Bigcapital | Belum terverifikasi dari kode pada phase ini sebagai submenu ERP terpisah. ERP memakai review queue dengan status `open`, `matched`, `categorized`, dan `excluded`. |
| Reason saat exclude/skip | Belum terverifikasi dari kode pada phase ini. Aksi exclude tidak menerima field reason. |
| `is_active` pada Bank Rule | Belum terverifikasi dari kode pada phase ini. Rule dapat dibuat, diedit, dan dihapus, tetapi tidak ada field aktif/nonaktif yang diaudit. |
| Auto-post Bank Rule tanpa user | Belum terverifikasi dari kode pada phase ini. Rule memberi suggestion untuk form categorize. |
| Status review `reconciled` | Belum terverifikasi dari kode pada phase ini sebagai status tabel review. Status aktual: `open`, `matched`, `categorized`, `excluded`. |
| Pagination server-side review/bank accounts | Belum terverifikasi dari kode pada phase ini. Query review dibatasi `limit(100)`, akun bank list penuh dari akun Cash/Bank. |

Rujukan internal yang dipakai saat audit:

| Area | File utama |
| --- | --- |
| Sidebar Perbankan | `resources/js/layouts/AppLayout.tsx` |
| Route Perbankan | `routes/web.php` |
| Akun Bank/Cashflow | `app/Services/CashflowService.php`, `app/Http/Controllers/Banking/CashflowController.php`, `resources/js/pages/banking/accounts/*`, `resources/js/pages/banking/transactions/create.tsx` |
| Review transaksi bank | `app/Services/BankReviewService.php`, `app/Http/Controllers/Banking/BankReviewController.php`, `app/Http/Requests/Banking/*Uncategorized*`, `resources/js/pages/banking/review/*` |
| Aturan Bank | `app/Services/BankRuleService.php`, `app/Http/Controllers/Banking/BankRuleController.php`, `app/Http/Requests/Banking/*BankRule*`, `resources/js/pages/banking/rules/index.tsx` |
| Model dan migration | `app/Models/UncategorizedBankTransaction.php`, `app/Models/MatchedBankTransaction.php`, `app/Models/BankRule.php`, `app/Models/BankRuleCondition.php`, `database/migrations/2026_04_21_000001_create_banking_review_tables.php` |
| Permission | `database/seeders/RolePermissionSeeder.php` |
| Referensi Bigcapital | `bigcapital/packages/server/src/modules/Banking*`, `bigcapital/shared/sdk-ts/openapi.json` |
