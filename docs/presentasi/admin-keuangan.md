# Presentasi Admin Keuangan

Dokumen ini adalah panduan presentasi untuk super admin/admin ketika menyiapkan menu Keuangan dan Akuntansi. Fokusnya adalah penggunaan praktis di aplikasi: apa fungsi submenu, data apa yang perlu diisi, kolom mana yang perlu dijelaskan saat demo, dan apa dampaknya ke transaksi serta laporan.

Audit phase ini membaca route, controller, form request, service, model, seeder, dan halaman React yang terkait dengan Bagan Akun, Jurnal Manual, Penguncian Transaksi, dan Tarif Pajak. Jika ada field backend yang ditemukan tetapi tidak terlihat di form UI yang diaudit, field tersebut ditulis di bagian catatan belum terverifikasi.

## 1. Tujuan Dokumen

Panduan ini dipakai untuk:

| Tujuan | Penjelasan |
| --- | --- |
| Menyiapkan data awal finance | Admin dapat membuat akun GL, tarif pajak, jurnal pembuka sederhana, dan aturan lock sebelum demo transaksi. |
| Menjelaskan peran tiap submenu | Presenter dapat menjelaskan kenapa menu tersebut ada dan modul mana yang terdampak. |
| Mengurangi error saat demo | Setiap submenu memiliki checklist, validasi umum, dan contoh error yang harus dihindari. |
| Menyamakan istilah teknis dan bisnis | Istilah aplikasi tetap memakai label UI, sedangkan catatan internal menyebut route dan permission aktual yang diaudit. |
| Menandai gap audit | Field/menu yang belum terlihat di UI atau belum punya kontrak penuh tidak diklaim selesai. |

Audiens utama dokumen ini adalah:

| Audiens | Kebutuhan |
| --- | --- |
| Super admin/admin | Menyiapkan akun, pajak, lock, dan contoh jurnal sebelum demo. |
| Tim implementasi | Memastikan data demo cukup untuk menjalankan alur invoice, bill, payment, dan report dasar. |
| Presenter | Menjelaskan alur finance tanpa masuk terlalu dalam ke teknis Laravel/Inertia. |

Batasan penting:

- Dokumen ini bukan konsultasi pajak atau standar akuntansi resmi.
- Default bahasa UI adalah English, tetapi narasi panduan ini memakai bahasa Indonesia.
- Data contoh dapat disesuaikan dengan industri klien, tetapi tipe akun dan aturan validasinya harus mengikuti aplikasi.
- Jangan mendemokan pilihan basis cash untuk Trial Balance, Balance Sheet, atau Tax Summary. Kontrak cash-basis untuk laporan tersebut masih dibatasi oleh catatan arsitektur proyek.

Rujukan silang:

- Preferensi organisasi, akun default, dan penguncian: [admin-preferensi.md](admin-preferensi.md)
- Barang/Jasa, akun item, dan penyesuaian persediaan: [admin-barang-jasa.md](admin-barang-jasa.md)
- Penjualan yang memposting piutang, kas/bank, pendapatan, dan pajak: [admin-penjualan.md](admin-penjualan.md)
- Pembelian yang memposting utang, kas/bank, biaya/HPP, persediaan, dan pajak: [admin-pembelian.md](admin-pembelian.md)
- Laporan keuangan dan laporan detail GL: [admin-laporan.md](admin-laporan.md)

## 2. Gambaran Umum Menu Keuangan

Menu Keuangan berada di sidebar bagian `Financial`/`Accounting`. Empat submenu yang dipakai dalam presentasi admin adalah:

| Submenu UI | URL | Route internal yang diaudit | Permission utama | Fungsi singkat |
| --- | --- | --- | --- | --- |
| Bagan Akun | `/accounting/accounts` | `accounts.index`, `accounts.create`, `accounts.edit`, `accounts.show` | `account.view`, `account.create`, `account.edit`, `account.delete` | Master akun General Ledger. Semua transaksi akuntansi akhirnya masuk ke akun ini. |
| Jurnal Manual | `/accounting/journals` | `journals.index`, `journals.create`, `journals.show`, `journals.edit` | `manual-journal.view`, `manual-journal.create`, `manual-journal.edit`, `manual-journal.delete` | Membuat jurnal double-entry langsung ke GL untuk pembukaan, koreksi, dan penyesuaian. |
| Penguncian Transaksi | `/settings/accounting` | `settings.accounting`, `settings.accounting.lock`, `settings.accounting.cancel-lock`, `settings.accounting.unlock-partial`, `settings.accounting.cancel-unlock-partial` | `setting.edit` dan user admin-like | Mengunci transaksi sampai tanggal tertentu agar periode yang sudah ditutup tidak berubah. |
| Tarif Pajak | `/settings/tax-rates` | `settings.tax-rates` dan API `/api/tax-rates` | `tax-rate.view`, `tax-rate.create`, `tax-rate.edit`, `tax-rate.delete` | Master persentase pajak yang dipakai item dan baris transaksi. |

Catatan route penting untuk tim teknis:

- Jurnal Manual tidak memakai route name `manual-journals.*`; route halaman yang diaudit adalah `journals.*`.
- Penguncian Transaksi tidak memakai route name `transaction-locks.*`; fitur ini berada di `settings.accounting.*`.
- Tarif Pajak berada di Settings secara URL, tetapi ditampilkan di grup Financial pada sidebar.

Dampak lintas modul:

| Data finance | Dipakai oleh | Dampak jika belum siap |
| --- | --- | --- |
| Akun `accounts-receivable` | Invoice, credit note, payment receive | Posting invoice atau payment dapat gagal karena akun Piutang Usaha belum tersedia. |
| Akun `accounts-payable` | Bill, vendor credit, bill payment | Posting bill atau payment made dapat gagal karena akun Utang Usaha belum tersedia. |
| Akun `tax-payable` | Invoice, bill, sale receipt, credit note, vendor credit | Transaksi dengan pajak dapat gagal atau pajak tidak bisa diposting ke GL. |
| Akun income/expense/COGS | Item dan baris transaksi | Item tidak punya akun pendapatan/biaya yang benar; laporan laba rugi menjadi tidak bermakna. |
| Tarif pajak aktif | Item, invoice, bill, credit note, sale receipt, vendor credit, Tax Summary | Baris transaksi tidak bisa memakai tarif pajak yang benar. |
| Transaction lock | Sales, Purchases, Financial | Transaksi tanggal lama diblokir setelah periode ditutup. |
| Jurnal manual published | Account transactions, GL, report | Draft belum memengaruhi saldo; hanya published yang masuk GL. |

Scope penguncian transaksi yang diaudit:

| Scope | Nilai internal | Modul yang diblokir saat tanggal transaksi berada dalam periode terkunci |
| --- | --- | --- |
| All Transactions | `all` | Semua scope yang didukung oleh service lock. |
| Sales | `sales` | Sale invoice, sale receipt, credit note, payment receive. |
| Purchases | `purchases` | Bill, vendor credit, bill payment. |
| Financial | `financial` | Manual journal, expense, cashflow/banking review tertentu. |

## 3. Urutan Penggunaan Yang Disarankan

Urutan ini aman untuk demo awal karena mengikuti dependensi aplikasi.

| Urutan | Menu | Yang dilakukan | Alasan |
| --- | --- | --- | --- |
| 1 | Preferensi Umum | Pastikan base currency, tanggal, dan dokumen dasar sudah disiapkan. | Format uang/tanggal memengaruhi tampilan semua transaksi. Rujuk `admin-preferensi.md`. |
| 2 | Bagan Akun | Buat akun minimal untuk kas/bank, AR, AP, pajak, modal, pendapatan, HPP, dan beban. | Service transaksi mencari akun berdasarkan `AccountType`, bukan hanya nama akun. |
| 3 | Tarif Pajak | Pastikan PPN dan Non Pajak aktif. | Item dan baris transaksi membutuhkan pilihan pajak yang aktif. |
| 4 | Items | Hubungkan item ke akun penjualan/pembelian dan tarif pajak. | Invoice/bill mengambil akun dan pajak default dari item. |
| 5 | Jurnal Manual | Buat jurnal setoran modal atau saldo awal sederhana, lalu publish. | Memberi saldo awal bank/kas untuk demo pembayaran. |
| 6 | Sales/Purchases | Buat invoice atau bill kecil memakai item dan pajak. | Menguji bahwa akun AR/AP/Tax Payable sudah siap. |
| 7 | Reports | Cek laporan GL/default accrual yang relevan. | Memastikan posting jurnal sudah masuk ke account transactions. |
| 8 | Penguncian Transaksi | Setelah demo transaksi bulan lalu selesai, lock sampai akhir bulan. | Mencegah perubahan periode tertutup. |

Urutan singkat untuk presentasi 20-30 menit:

| Menit | Aktivitas |
| --- | --- |
| 0-5 | Jelaskan gambaran menu Keuangan dan kenapa COA harus disiapkan lebih dulu. |
| 5-12 | Tunjukkan Bagan Akun, filter, tipe akun, dan akun wajib. |
| 12-17 | Tunjukkan Tarif Pajak dan kaitannya dengan item/transaksi. |
| 17-23 | Buat Jurnal Manual setoran modal, cek balanced, lalu publish. |
| 23-27 | Tunjukkan Penguncian Transaksi dan contoh partial unlock. |
| 27-30 | Tutup dengan checklist setelah setup. |

## 4. Sub Menu Bagan Akun

### Kegunaan

Bagan Akun adalah daftar akun General Ledger yang menjadi fondasi double-entry accounting. Semua invoice, bill, payment, expense, jurnal manual, dan transaksi lain akhirnya membuat baris di `account_transactions` dengan mengacu ke akun di menu ini.

Bagan Akun digunakan untuk:

| Kebutuhan | Contoh |
| --- | --- |
| Menentukan saldo normal | Kas, Bank, Piutang, Persediaan, HPP, dan Beban bersaldo normal debit. Utang, Pajak, Modal, dan Pendapatan bersaldo normal kredit. |
| Mengelompokkan laporan | Aset/liabilitas/ekuitas masuk Balance Sheet; pendapatan/HPP/beban masuk Income Statement. |
| Menyediakan akun wajib transaksi | Invoice butuh Accounts Receivable dan Tax Payable; bill butuh Accounts Payable dan Tax Payable. |
| Menjaga struktur akun | Akun dapat dibuat bertingkat sampai maksimal 5 level. |
| Menonaktifkan akun lama | Akun yang tidak dipakai lagi dapat dibuat inactive tanpa menghapus riwayat transaksi. |

### Akses dan navigasi

| Aksi | Permission | Catatan |
| --- | --- | --- |
| Lihat daftar/detail | `account.view` | Role umum dapat diberi akses lihat. |
| Buat akun | `account.create` | Diperlukan untuk setup awal. |
| Edit akun | `account.edit` | Tipe akun tidak boleh diubah setelah dibuat. |
| Hapus akun | `account.delete` | Dibatasi oleh aturan sistem, anak akun, dan transaksi. |
| Import/export/print | `account.view` atau `account.create` sesuai aksi | Route tersedia, tetapi fokus presentasi utama tetap create/edit/list. |

### Kolom daftar dan fungsinya

| Kolom | Fungsi saat presentasi |
| --- | --- |
| Checkbox | Memilih akun yang aman dihapus massal. Akun sistem, akun induk, atau akun dengan transaksi tidak bisa dipilih bebas. |
| Account Name | Nama akun. Jika ada deskripsi, deskripsi tampil di bawah nama. |
| Code | Kode akun untuk urutan dan identifikasi cepat, misalnya `1002` untuk Bank BCA. |
| Type | Backing value `AccountType` yang menentukan laporan dan saldo normal. |
| Account Normal | Debit atau credit. Ini membantu menjelaskan kenapa kas bertambah di debit dan pendapatan bertambah di kredit. |
| Currency | Mata uang akun. Jika kosong, tampilan memakai base currency organisasi. |
| Bank Balance | Saldo dari konteks bank/cashflow jika tersedia. Tidak semua akun punya nilai ini. |
| Balance | Saldo terhitung dari `account_transactions`, bukan input manual di form create/edit yang diaudit. |
| Actions | Edit, delete, activate/inactivate sesuai permission dan status akun. |

Tampilan daftar mendukung:

| Fitur | Penjelasan |
| --- | --- |
| Flat view | Daftar akun datar dengan filter/search. Cocok untuk demo cepat. |
| Tree view | Struktur akun per root type: Asset, Liability, Equity, Income, Expense. Cocok untuk menjelaskan hierarki. |
| Filter status | Active, inactive, atau all. |
| Filter type | Menampilkan satu tipe akun tertentu. |
| Search | Mencari berdasarkan nama atau kode akun. |

### Input form Bagan Akun

| Field | Wajib | Validasi yang diaudit | Fungsi |
| --- | --- | --- | --- |
| Code | Tergantung preferensi `account_code_required` | String maksimal 30; unik jika `account_code_unique` aktif; soft-deleted code boleh dipakai ulang sesuai aturan unique where null deleted. | Kode akun untuk sorting, laporan, dan identifikasi cepat. |
| Account Name | Ya | String maksimal 255; nama tidak boleh duplikat dalam parent yang sama selama akun lama belum soft-deleted. | Nama akun yang dibaca user. |
| Account Type | Ya | Harus salah satu nilai enum `AccountType`. | Menentukan saldo normal, posisi laporan, dan eligibility service transaksi. |
| Parent Account | Tidak | ID akun harus ada; root type anak harus sama dengan root type parent; maksimal 5 level. | Membuat struktur akun bertingkat. |
| Description | Tidak | String maksimal 1000. | Catatan penggunaan akun. |
| Currency | Tidak | Kode currency harus ada di master currencies; maksimal 10 karakter. | Dipakai untuk akun multi currency atau label saldo. Jika kosong pada sub-akun, service dapat mewarisi currency parent. |
| Active | Tidak | Boolean. | Akun inactive tidak ditujukan untuk transaksi baru, tetapi riwayat tetap tersimpan. |

Field yang sering ditanyakan:

| Field | Status audit |
| --- | --- |
| Opening balance | Tidak terlihat sebagai input create/edit Bagan Akun pada halaman yang diaudit. Saldo tampilan berasal dari transaksi GL terhitung. Untuk demo saldo awal, gunakan Jurnal Manual. |
| Balance | Ada di model, tetapi pada daftar COA saldo dihitung dari `account_transactions`. Jangan isi saldo langsung dari halaman COA. |
| Account type setelah dibuat | Tidak dapat diubah. Service melempar error `Account type cannot be changed after creation.` |

### Tipe akun yang paling sering dipakai

| Tipe akun | Backing value | Saldo normal | Laporan | Contoh akun |
| --- | --- | --- | --- | --- |
| Cash | `cash` | Debit | Balance Sheet | Kas kecil, kas toko. |
| Bank | `bank` | Debit | Balance Sheet | Bank BCA, Bank Mandiri. |
| Accounts Receivable | `accounts-receivable` | Debit | Balance Sheet | Piutang Usaha. |
| Inventory | `inventory` | Debit | Balance Sheet | Persediaan Barang. |
| Accounts Payable | `accounts-payable` | Credit | Balance Sheet | Utang Usaha. |
| Tax Payable | `tax-payable` | Credit | Balance Sheet | Pajak Keluaran, Pajak Terutang. |
| Equity | `equity` | Credit | Balance Sheet | Modal Pemilik. |
| Income | `income` | Credit | Income Statement | Pendapatan Penjualan, Pendapatan Jasa. |
| Cost of Goods Sold | `cost-of-goods-sold` | Debit | Income Statement | Harga Pokok Penjualan. |
| Expense | `expense` | Debit | Income Statement | Beban Operasional, Beban Administrasi Bank. |

### Contoh data awal Bagan Akun

Gunakan data ini untuk presentasi awal. Kode dapat disesuaikan, tetapi tipe akun harus mengikuti tabel.

| Code | Account Name | Account Type | Normal | Kegunaan demo |
| --- | --- | --- | --- | --- |
| `1001` | Kas | `cash` | Debit | Kas operasional atau petty cash. |
| `1002` | Bank BCA | `bank` | Debit | Akun bank utama untuk setoran modal dan pembayaran. |
| `1101` | Piutang Usaha | `accounts-receivable` | Debit | Akun wajib invoice dan payment receive. |
| `1201` | Persediaan Barang | `inventory` | Debit | Akun aset untuk item inventory. |
| `2001` | Utang Usaha | `accounts-payable` | Credit | Akun wajib bill dan bill payment. |
| `2101` | Pajak Keluaran | `tax-payable` | Credit | Akun pajak otomatis yang wajib tersedia untuk transaksi kena pajak. |
| `2102` | Pajak Masukan | `tax-payable` | Credit | Akun pendamping untuk demo konsep PPN masukan/manual. Service pajak yang diaudit tetap mencari akun berdasarkan tipe `tax-payable`. |
| `3001` | Modal Pemilik | `equity` | Credit | Lawan akun untuk setoran modal awal. |
| `4001` | Pendapatan Penjualan | `income` | Credit | Akun pendapatan item barang. |
| `4002` | Pendapatan Jasa | `income` | Credit | Akun pendapatan item jasa. |
| `5001` | Harga Pokok Penjualan | `cost-of-goods-sold` | Debit | Akun HPP item inventory. |
| `6001` | Beban Operasional | `expense` | Debit | Beban umum untuk expense atau jurnal koreksi. |
| `6002` | Beban Administrasi Bank | `expense` | Debit | Contoh beban admin bank bulanan. |

Catatan untuk akun pajak:

- Service transaksi mencari akun pajak dengan `AccountType::TaxPayable`.
- Jika ada lebih dari satu akun `tax-payable`, pastikan akun utama yang ingin dipakai otomatis sudah jelas saat demo. Kode lebih kecil biasanya lebih mudah dijelaskan sebagai akun utama.
- Pemetaan tarif pajak ke akun pajak tertentu belum terlihat sebagai field pada audit phase ini.

### Dampak ke modul lain dan laporan

| Modul/laporan | Dampak Bagan Akun |
| --- | --- |
| Invoice | Debit ke Accounts Receivable; credit ke akun pendapatan item; credit ke Tax Payable jika ada pajak. |
| Sale Receipt | Debit ke akun kas/bank/deposit; credit ke pendapatan dan Tax Payable. |
| Bill | Credit ke Accounts Payable; debit ke akun biaya/persediaan item; debit ke Tax Payable untuk pajak pembelian sesuai service saat ini. |
| Payment Receive | Mengurangi piutang dan menambah kas/bank. Membutuhkan akun Accounts Receivable. |
| Bill Payment | Mengurangi utang dan kas/bank. Membutuhkan akun Accounts Payable. |
| Manual Journal | Langsung membuat debit/credit ke akun yang dipilih saat dipublish. |
| Income Statement | Mengambil akun income, other income, COGS, expense, dan other expense. |
| Balance Sheet | Mengambil akun asset, liability, dan equity. Jangan demokan pilihan basis cash untuk Balance Sheet. |
| General Ledger/detail akun | Menampilkan riwayat `account_transactions` per akun. |

### Validasi dan error umum

| Kondisi | Pesan/akibat | Cara menghindari |
| --- | --- | --- |
| Nama akun sama dalam parent yang sama | Service melempar duplicate account. | Pakai nama yang spesifik, misalnya `Bank BCA IDR`. |
| Kode akun sama saat unique code aktif | Service melempar duplicate account code. | Pakai kode berbeda. |
| Parent beda root type | Error `Child account type must match its parent account type.` | Sub-akun asset harus berada di parent asset, begitu juga liability, income, expense, dan equity. |
| Hierarki terlalu dalam | Error `Account hierarchy cannot exceed 5 levels.` | Batasi struktur sampai 5 level. |
| Ubah account type setelah dibuat | Error `Account type cannot be changed after creation.` | Jika tipe salah dan belum ada transaksi, buat akun baru dengan tipe benar. |
| Hapus akun sistem | Diblokir sebagai system/predefined account. | Inactive-kan akun jika diizinkan, atau biarkan aktif bila dibutuhkan sistem. |
| Hapus akun induk | Diblokir karena punya child accounts. | Pindahkan/hapus sub-akun dulu. |
| Hapus akun dengan transaksi | Diblokir karena riwayat GL tidak boleh hilang. | Nonaktifkan akun, jangan hapus. |
| Invoice/bill gagal posting karena required account missing | Service meminta akun type AR/AP/Tax Payable dibuat dulu. | Pastikan akun wajib ada sebelum demo transaksi. |

### Checklist setelah simpan Bagan Akun

- Akun tampil di daftar dengan status Active.
- Kode dan nama mudah dibaca di dropdown transaksi.
- Account Type benar, terutama AR, AP, Tax Payable, Income, COGS, Expense, Cash/Bank.
- Jika akun adalah sub-akun, parent berada di root type yang sama.
- Currency sesuai kebutuhan demo atau kosong untuk base currency.
- Akun wajib invoice/bill/payment sudah ada minimal satu per tipe.
- Jangan lanjut ke invoice/bill jika akun Tax Payable belum tersedia.
- Untuk saldo awal, siapkan Jurnal Manual, bukan input balance langsung di COA.

## 5. Sub Menu Jurnal Manual

### Kegunaan

Jurnal Manual dipakai untuk membuat jurnal double-entry langsung ke General Ledger. Fitur ini cocok untuk:

| Kebutuhan | Contoh |
| --- | --- |
| Saldo awal sederhana | Setoran modal awal ke Bank BCA. |
| Penyesuaian bulanan | Biaya administrasi bank, accrual sederhana, atau koreksi. |
| Koreksi akun | Memindahkan nominal dari akun salah ke akun benar. |
| Demo prinsip double-entry | Menunjukkan bahwa total debit harus sama dengan total credit. |

Alur status:

| Status | Arti | Dampak ke GL |
| --- | --- | --- |
| Draft | Jurnal tersimpan tetapi belum dipublish. | Belum membuat `account_transactions`. |
| Published | Jurnal sudah diterbitkan. | Membuat baris debit/credit di `account_transactions` dengan `transaction_type` Manual Journal. |

### Akses dan navigasi

| Aksi | Permission | Catatan |
| --- | --- | --- |
| Lihat daftar/detail | `manual-journal.view` | Wajib untuk membuka halaman jurnal. |
| Buat jurnal | `manual-journal.create` | Halaman create mensyaratkan minimal dua akun tersedia. |
| Edit/publish jurnal | `manual-journal.edit` | Tombol publish hanya ditampilkan untuk Draft. |
| Hapus jurnal | `manual-journal.delete` | UI menampilkan hapus untuk Draft. Bulk delete melewati published. |

Route halaman:

| Halaman | URL |
| --- | --- |
| Daftar jurnal | `/accounting/journals` |
| Buat jurnal | `/accounting/journals/create` |
| Detail jurnal | `/accounting/journals/{id}` |
| Edit jurnal | `/accounting/journals/{id}/edit` |

### Kolom daftar dan fungsinya

| Kolom | Fungsi |
| --- | --- |
| Checkbox | Memilih jurnal Draft untuk bulk delete jika punya permission delete. |
| Journal No. | Nomor jurnal. Link ke detail jurnal. |
| Date | Tanggal jurnal dan tanggal posting GL saat dipublish. |
| Description | Catatan header jurnal. |
| Projects | Ringkasan nama project dari baris jurnal. |
| Status | Draft atau Published. |
| Amount | Total nominal jurnal, dihitung dari total credit. |
| Actions | View, edit, delete sesuai status dan permission. |

Filter yang tersedia:

| Filter | Fungsi |
| --- | --- |
| Search | Mencari jurnal dari daftar. |
| Status | Draft atau Published. |
| Date range | Membatasi jurnal dari tanggal awal sampai tanggal akhir. |

### Input form Jurnal Manual

Header jurnal:

| Field | Wajib | Validasi yang diaudit | Fungsi |
| --- | --- | --- | --- |
| Journal Number | Backend nullable, UI menampilkan wajib dan mengisi nomor berikutnya | String maksimal 50; unik di jurnal yang belum soft-deleted. | Nomor dokumen jurnal, misalnya `JNL-00001`. |
| Date | Ya | Date. | Tanggal jurnal dan tanggal yang dicek oleh transaction lock saat publish. |
| Reference | Tidak | String maksimal 50. | Nomor referensi eksternal, misalnya nomor rekening koran. |
| Notes/Description | Tidak | String maksimal 2000. | Penjelasan jurnal. |
| Publish | Tidak | Boolean. | Jika true saat store, jurnal dibuat dan langsung dipublish. |

Baris jurnal:

| Field baris | Wajib | Validasi yang diaudit | Fungsi |
| --- | --- | --- | --- |
| Account | Ya | ID akun harus ada. | Akun GL yang didebit atau dikredit. |
| Description/Note | Tidak | String maksimal 1000. | Catatan per baris. |
| Project | Tidak | ID project harus ada jika diisi. | Pelacakan biaya/pendapatan per project. |
| Debit | Ya | Numeric minimal 0; kosong dinormalisasi menjadi 0. | Nilai sisi debit. |
| Credit | Ya | Numeric minimal 0; kosong dinormalisasi menjadi 0. | Nilai sisi credit. |

Aturan jumlah baris:

| Aturan | Nilai |
| --- | --- |
| Minimum baris | 2 |
| Maksimum baris | 500 |
| Total debit | Harus lebih besar dari 0 |
| Total credit | Harus lebih besar dari 0 |
| Selisih debit/credit | Harus sama setelah pembulatan 2 desimal di service |

Field backend yang tervalidasi tetapi tidak terlihat di form React yang diaudit:

| Field | Validasi | Catatan |
| --- | --- | --- |
| `currency_code` | Nullable, exists currencies, max 10 | Tidak terlihat sebagai input pada create/edit yang diaudit. |
| `exchange_rate` | Nullable numeric, greater than 0 | Tidak terlihat sebagai input pada create/edit yang diaudit. |
| `branch_id` header | Nullable exists branches | Tidak terlihat sebagai input pada create/edit yang diaudit. |
| `entries.*.contact_id` | Nullable exists contacts | Tidak terlihat sebagai input pada create/edit yang diaudit. |
| `entries.*.branch_id` | Nullable exists branches | Tidak terlihat sebagai input pada create/edit yang diaudit. |

### Contoh jurnal untuk presentasi

#### Setoran modal awal

Tujuan: memberi saldo awal Bank BCA untuk demo pembayaran.

| Field | Isi contoh |
| --- | --- |
| Journal Number | `JNL-00001` |
| Date | `2026-05-01` |
| Reference | `SETORAN-MODAL-001` |
| Notes | `Setoran modal awal pemilik ke rekening operasional.` |

| Account | Debit | Credit | Note |
| --- | ---: | ---: | --- |
| `1002 - Bank BCA` | 50,000,000 | 0 | Dana masuk ke rekening perusahaan. |
| `3001 - Modal Pemilik` | 0 | 50,000,000 | Penambahan modal pemilik. |

Dampak setelah publish:

| Akun | Dampak |
| --- | --- |
| Bank BCA | Bertambah debit 50,000,000. |
| Modal Pemilik | Bertambah credit 50,000,000. |
| Balance Sheet | Aset dan ekuitas naik seimbang. |

#### Biaya administrasi bank

Tujuan: menunjukkan beban kecil yang mengurangi bank.

| Field | Isi contoh |
| --- | --- |
| Journal Number | `JNL-00002` |
| Date | `2026-05-05` |
| Reference | `BANK-FEE-0526` |
| Notes | `Biaya administrasi bank bulan Mei 2026.` |

| Account | Debit | Credit | Note |
| --- | ---: | ---: | --- |
| `6002 - Beban Administrasi Bank` | 25,000 | 0 | Beban admin bank. |
| `1002 - Bank BCA` | 0 | 25,000 | Saldo bank berkurang. |

Dampak setelah publish:

| Akun | Dampak |
| --- | --- |
| Beban Administrasi Bank | Bertambah debit 25,000. |
| Bank BCA | Berkurang melalui credit 25,000. |
| Income Statement | Beban naik, laba turun. |

#### Koreksi salah akun

Tujuan: memindahkan beban yang sebelumnya masuk akun salah.

Skenario: biaya operasional Rp100,000 terlanjur masuk `6002 - Beban Administrasi Bank`, padahal akun yang benar adalah `6001 - Beban Operasional`.

| Field | Isi contoh |
| --- | --- |
| Journal Number | `JNL-00003` |
| Date | `2026-05-10` |
| Reference | `KOREKSI-BEBAN-001` |
| Notes | `Koreksi klasifikasi beban dari administrasi bank ke beban operasional.` |

| Account | Debit | Credit | Note |
| --- | ---: | ---: | --- |
| `6001 - Beban Operasional` | 100,000 | 0 | Akun beban yang benar. |
| `6002 - Beban Administrasi Bank` | 0 | 100,000 | Mengurangi akun beban yang salah. |

Dampak setelah publish:

| Akun | Dampak |
| --- | --- |
| Beban Operasional | Bertambah 100,000. |
| Beban Administrasi Bank | Berkurang 100,000. |
| Total beban | Tidak berubah, hanya klasifikasinya berpindah. |

### Dampak ke modul lain dan laporan

| Area | Dampak Jurnal Manual |
| --- | --- |
| General Ledger | Published journal membuat baris GL dengan reference `ManualJournal`. |
| Detail akun | Baris jurnal tampil di transaksi akun terkait. |
| Income Statement | Baris ke income/expense/COGS memengaruhi laba rugi. |
| Balance Sheet | Baris ke asset/liability/equity memengaruhi neraca. Jangan demokan pilihan basis cash. |
| Project report | Jika project diisi di baris jurnal, biaya/pendapatan bisa dilacak per project. |
| Transaction Locking | Publish jurnal dengan tanggal pada periode financial terkunci akan diblokir. |

### Validasi dan error umum

| Kondisi | Pesan/akibat | Cara menghindari |
| --- | --- | --- |
| Kurang dari dua akun di COA | Halaman create redirect ke Bagan Akun dengan error prasyarat. | Buat minimal dua akun sebelum demo jurnal. |
| Total debit tidak sama dengan total credit | Error `Journal entry is not balanced...` | Cek indikator balance sebelum save/publish. |
| Total debit/credit nol | Error `Journal entry total must be greater than zero.` | Isi nominal di salah satu sisi setiap baris yang relevan. |
| Nomor jurnal duplikat | Error duplicate journal. | Gunakan nomor otomatis atau tambah suffix unik. |
| Account kosong di baris | Validasi `entries.*.account_id` gagal. | Pilih akun pada semua baris. |
| Tanggal berada dalam periode locked financial | Error transaction locked. | Pakai tanggal setelah lock date atau lakukan partial unlock sesuai prosedur. |
| Publish jurnal yang sudah published | Error document already opened/published. | Jangan klik publish ulang. |

### Checklist setelah simpan Jurnal Manual

- Total debit sama dengan total credit.
- Nominal tidak nol.
- Tanggal sesuai periode demo.
- Jika akan publish, pastikan periode financial tidak terkunci.
- Status berubah dari Draft ke Published setelah publish.
- Detail jurnal menampilkan baris akun, debit, credit, dan total yang seimbang.
- Cek detail akun Bank BCA/Modal/Beban untuk memastikan transaksi muncul.
- Untuk demo report, gunakan jurnal sederhana yang mudah dijelaskan.

## 6. Sub Menu Penguncian Transaksi

### Kegunaan

Penguncian Transaksi dipakai setelah periode akuntansi ditutup. Tujuannya adalah mencegah invoice, bill, payment, expense, journal, dan transaksi lain mengubah data pada tanggal yang sudah disetujui.

Cara kerja yang diaudit:

| Konsep | Penjelasan |
| --- | --- |
| Lock sampai tanggal | Jika `lock_to_date` adalah `2026-04-30`, transaksi bertanggal `2026-04-30` atau sebelumnya diblokir. |
| Scope `all` | Mengunci semua scope transaksi dalam satu aksi. |
| Scope per module | Mengunci `sales`, `purchases`, atau `financial` secara terpisah. |
| Partial unlock | Membuka sementara rentang tanggal tertentu pada scope yang sedang locked. |
| Cancel lock | Menghapus lock aktif pada scope. |
| Cancel partial unlock | Menghapus pengecualian partial unlock dan mengembalikan lock normal. |

### Akses dan navigasi

| Aksi | Permission/role | Catatan |
| --- | --- | --- |
| Buka halaman | User harus admin-like dan memiliki `setting.edit`. | Sidebar memberi `requiresAdmin: true`. |
| Lock | Admin-like + `setting.edit`. | Request `LockTransactionRequest`. |
| Cancel lock | Admin-like + `setting.edit`. | Request `CancelTransactionLockRequest`. |
| Partial unlock | Admin-like + `setting.edit`. | Request `PartialUnlockTransactionRequest`. |
| Cancel partial unlock | Admin-like + `setting.edit`. | Request `CancelPartialUnlockTransactionRequest`. |

### Kartu dan kolom informasi

Halaman menampilkan empat kartu:

| Kartu | Nilai internal | Deskripsi dari service |
| --- | --- | --- |
| All Transactions | `all` | Mengunci semua transaksi akuntansi dalam satu aksi. |
| Sales | `sales` | Invoices, receipts, credit notes, dan customer payments. |
| Purchases | `purchases` | Bills, vendor credits, dan bill payments. |
| Financial | `financial` | Manual journals dan expenses. |

Informasi dalam kartu:

| Informasi | Fungsi |
| --- | --- |
| Status Open/Locked | Menunjukkan apakah scope sedang terkunci. |
| Locked through | Tanggal akhir periode terkunci. |
| Reason | Alasan lock atau unlock yang disimpan admin. |
| Partial unlock range | Rentang tanggal yang sementara boleh diubah. |
| Current mode | Menunjukkan mode `All Transactions` atau `Per Module`. |

### Input dan aksi

#### Lock transaksi

| Field | Wajib | Validasi | Fungsi |
| --- | --- | --- | --- |
| Module | Ya | Salah satu `all`, `sales`, `purchases`, `financial`. | Scope yang akan dikunci. Dipilih dari kartu, bukan input bebas. |
| Lock to date | Ya | Date. | Semua transaksi sampai tanggal ini diblokir. |
| Reason | Tidak | String maksimal 1000. | Alasan lock, misalnya closing bulan April. |

#### Cancel lock

| Field | Wajib | Validasi | Fungsi |
| --- | --- | --- | --- |
| Module | Ya | Salah satu scope valid. | Scope yang lock-nya dibatalkan. |
| Unlock reason | Tidak | String maksimal 1000. | Alasan pembatalan lock. |

#### Partial unlock

| Field | Wajib | Validasi | Fungsi |
| --- | --- | --- | --- |
| Module | Ya | Salah satu scope valid. | Scope yang dibuka sementara. |
| Unlock from | Ya | Date. | Tanggal awal pengecualian. |
| Unlock to | Ya | Date dan harus setelah/sama dengan `unlock_from_date`. | Tanggal akhir pengecualian. |
| Partial unlock reason | Tidak | String maksimal 1000. | Alasan pembukaan sementara. |

Aturan partial unlock:

| Aturan | Dampak |
| --- | --- |
| Harus ada active lock | Jika belum ada lock, partial unlock ditolak. |
| Rentang unlock tidak boleh melewati lock date | Jika lock sampai `2026-04-30`, partial unlock tidak boleh sampai `2026-05-01`. |
| Saat mode lock `all`, partial unlock kartu module dinonaktifkan | UI meminta beralih ke per-module locking dulu untuk unlock module tertentu. |
| Tanggal transaksi dalam rentang partial unlock diizinkan | Service `assertNotLocked` mengizinkan tanggal yang masuk rentang partial unlock. |

### Contoh skenario penguncian

#### Lock sampai akhir bulan

Skenario: closing April 2026 selesai. Admin ingin mencegah perubahan transaksi April.

| Field | Isi contoh |
| --- | --- |
| Card | All Transactions |
| Lock to date | `2026-04-30` |
| Reason | `Closing April 2026 sudah disetujui oleh manajemen.` |

Dampak:

| Tanggal transaksi | Hasil |
| --- | --- |
| `2026-04-29` | Diblokir. |
| `2026-04-30` | Diblokir. |
| `2026-05-01` | Diizinkan. |

#### Partial unlock

Skenario: setelah closing, ada invoice sales tanggal `2026-04-22` yang perlu dikoreksi.

Langkah yang aman untuk demo:

| Langkah | Aksi |
| --- | --- |
| 1 | Pastikan mode lock per module untuk Sales, bukan hanya All Transactions, jika ingin membuka Sales saja. |
| 2 | Buka kartu Sales. |
| 3 | Pilih Partial Unlock. |
| 4 | Isi `unlock_from_date` = `2026-04-22`. |
| 5 | Isi `unlock_to_date` = `2026-04-22`. |
| 6 | Isi reason `Koreksi invoice demo tanggal 22 April 2026.` |
| 7 | Simpan. |
| 8 | Setelah koreksi selesai, pilih Cancel Partial Unlock. |

Dampak:

| Tanggal transaksi Sales | Hasil |
| --- | --- |
| `2026-04-21` | Tetap terkunci. |
| `2026-04-22` | Diizinkan sementara. |
| `2026-04-23` | Tetap terkunci. |

#### Cancel lock

Skenario: admin salah memasang lock sampai `2026-05-31`, padahal Mei belum ditutup.

| Field | Isi contoh |
| --- | --- |
| Card | All Transactions atau module terkait |
| Action | Remove Lock |
| Unlock reason | `Tanggal lock salah, Mei 2026 belum closing.` |

Dampak:

- Lock scope tersebut menjadi inactive.
- Transaksi tanggal lama tidak lagi diblokir oleh scope tersebut.
- Reason cancel tersimpan sebagai catatan UI.

### Dampak ke modul lain

| Scope | Service yang diaudit memakai lock |
| --- | --- |
| Sales | SaleInvoiceService, SaleReceiptService, CreditNoteService, PaymentReceiveService. |
| Purchases | BillService, VendorCreditService, BillPaymentService. |
| Financial | ManualJournalService, ExpenseService, CashflowService, BankReviewService. |

Contoh pesan error:

```text
Transaction date 2026-04-30 is in a locked period (locked through 2026-04-30). Please unlock the period in Settings -> Accounting before making changes.
```

### Validasi dan error umum

| Kondisi | Pesan/akibat | Cara menghindari |
| --- | --- | --- |
| Module tidak valid | `Unknown transaction lock module [...]` | Gunakan tombol kartu, jangan kirim nilai manual. |
| Partial unlock tanpa active lock | `There is no active lock to partially unlock...` | Pasang lock dulu. |
| Unlock from setelah unlock to | `Partial unlock range must have a start date on or before the end date.` | Pastikan tanggal awal <= tanggal akhir. |
| Unlock range melewati lock date | `Partial unlock range must stay on or before the lock date...` | Batasi rentang sampai tanggal lock. |
| Partial unlock module saat mode all | `Switch to per-module locking before partially unlocking...` | Pasang lock module secara terpisah jika hanya ingin buka Sales/Purchases/Financial. |
| User bukan admin-like | 403. | Gunakan super admin/admin. |
| User tidak punya `setting.edit` | Request ditolak. | Perbarui role/permission. |

### Checklist setelah simpan Penguncian Transaksi

- Current mode sesuai ekspektasi: All Transactions atau Per Module.
- Kartu target berubah menjadi Locked.
- `Locked through` menampilkan tanggal yang benar.
- Reason jelas dan mudah diaudit.
- Coba transaksi tanggal setelah lock date untuk memastikan masih bisa berjalan.
- Jika perlu koreksi tanggal lama, gunakan partial unlock yang sangat sempit.
- Setelah koreksi selesai, cancel partial unlock.
- Jangan membiarkan lock salah bulan aktif sebelum demo transaksi berikutnya.

## 7. Sub Menu Tarif Pajak

### Kegunaan

Tarif Pajak adalah master persentase pajak yang dapat dipilih pada item dan baris transaksi. Model `TaxRate` menyimpan nama, kode, rate, deskripsi, status aktif, compound tax, dan non-recoverable flag.

Tarif pajak dipakai oleh:

| Area | Penggunaan |
| --- | --- |
| Item master | Field default `sell_tax_rate_id` dan `purchase_tax_rate_id`. |
| Invoice dan sale receipt | Baris item mengambil tarif pajak penjualan. |
| Bill dan vendor credit | Baris item mengambil tarif pajak pembelian. |
| Estimate dan credit note | Baris item menyimpan `tax_rate_id` dan nilai rate saat transaksi. |
| Account transactions | Baris pajak dapat menyimpan `tax_rate_id` dan `tax_rate`. |
| Tax Summary | Laporan membaca data pajak dari transaksi. Jangan demokan basis cash untuk Tax Summary. |

### Akses dan navigasi

| Aksi | Permission | Catatan |
| --- | --- | --- |
| Lihat daftar | `tax-rate.view` | Route halaman memakai `/settings/tax-rates`. |
| Buat tarif pajak | `tax-rate.create` | POST `/api/tax-rates`. |
| Edit/toggle active | `tax-rate.edit` | PUT `/api/tax-rates/{id}` atau POST toggle active. |
| Hapus | `tax-rate.delete` | Delete diblokir jika tarif sudah dipakai transaksi. |

### Kolom daftar dan fungsinya

| Kolom | Fungsi |
| --- | --- |
| Name | Nama pajak yang dipilih user, misalnya `PPN 11% Penjualan`. |
| Code | Kode singkat, misalnya `PPN-11-SALES`. |
| Rate | Persentase pajak, ditampilkan dengan `%`. |
| Description | Penjelasan pajak. |
| Status | Active atau inactive. Inactive tidak ditujukan untuk pilihan transaksi baru. |
| Compound | Menandai pajak majemuk. |
| Non Recoverable | Menandai pajak yang tidak dapat diklaim kembali. |
| Actions | Edit, activate/deactivate, delete sesuai permission. |

Halaman sudah memakai pagination client-side:

| Kontrol | Nilai |
| --- | --- |
| Rows per page | 5, 10, 20, 50, 100 |
| Default per page | 10 |

### Input form Tarif Pajak

| Field | Wajib | Validasi yang diaudit | Fungsi |
| --- | --- | --- | --- |
| Name | Ya | String maksimal 255. | Nama pajak yang dibaca user. |
| Code | Tidak | String maksimal 30. | Kode pajak. Tidak ada unique rule yang terlihat di Form Request. |
| Rate Percent | Ya | Numeric minimal 0 maksimal 100. | Persentase pajak. |
| Description | Tidak | String maksimal 255. | Keterangan pajak. UI memiliki counter 255 karakter. |
| Compound Tax | Tidak | Boolean. | Untuk pajak majemuk. |
| Non Recoverable | Tidak | Boolean. | Untuk pajak yang tidak recoverable. |
| Is Active | Tidak di form create yang diaudit | Boolean di backend; default model/migration true. | Status aktif/inactive dikontrol lewat toggle. |

Catatan penting:

- Field arah pajak `sales` atau `purchase` tidak terlihat di tabel `tax_rates`.
- Jika ingin memisahkan PPN penjualan dan pembelian, gunakan nama/kode berbeda lalu pilih sebagai default sell/purchase tax di item.
- Seeder sudah menyediakan tarif standar Indonesia seperti `PPN 11%`, `PPN 12%`, PPh, `Tax Exempt`, dan `Non-Taxable`.

### Contoh data awal Tarif Pajak

| Name | Code | Rate | Compound | Non Recoverable | Description |
| --- | --- | ---: | --- | --- | --- |
| PPN 11% Penjualan | `PPN-11-SALES` | 11 | No | No | PPN keluaran untuk transaksi penjualan. |
| PPN 11% Pembelian | `PPN-11-PURCH` | 11 | No | No | PPN masukan untuk transaksi pembelian. |
| Non Pajak 0% | `NON-TAX` | 0 | No | No | Untuk item atau transaksi yang tidak dikenakan pajak. |

Jika database demo sudah memakai seeder:

| Kondisi | Saran demo |
| --- | --- |
| `PPN 11%` sudah ada | Gunakan tarif existing untuk demo singkat. |
| Perlu membedakan sales/purchase di slide | Buat dua tarif baru dengan suffix Penjualan/Pembelian, lalu hubungkan di item. |
| Klien bertanya kenapa ada PPN 12% | Jelaskan bahwa seeder menyediakan pilihan standar, tetapi admin memilih tarif yang aktif/relevan untuk transaksi. |

### Dampak ke modul lain dan laporan

| Modul | Dampak Tarif Pajak |
| --- | --- |
| Items | Default pajak penjualan/pembelian disimpan sebagai `sell_tax_rate_id` dan `purchase_tax_rate_id`. |
| Invoice | Baris item menyimpan tax rate dan menghitung pajak penjualan. |
| Bill | Baris item menyimpan tax rate dan menghitung pajak pembelian. |
| Credit Note/Vendor Credit | Pajak dibalik sesuai arah dokumen koreksi. |
| Sale Receipt | Pajak langsung diposting bersama penerimaan. |
| Account Transactions | Baris pajak menyimpan `tax_rate_id` dan nilai `tax_rate`. |
| Tax Summary | Data pajak dapat diringkas dari transaksi. Basis cash tidak diekspos untuk demo. |

### Validasi dan error umum

| Kondisi | Pesan/akibat | Cara menghindari |
| --- | --- | --- |
| Name kosong | Validasi required gagal. | Isi nama pajak. |
| Rate kosong | Validasi required gagal. | Isi rate angka. |
| Rate negatif atau lebih dari 100 | Validasi min/max gagal. | Gunakan 0 sampai 100. |
| Description lebih dari 255 karakter | UI membatasi dan backend menolak jika lebih. | Gunakan deskripsi singkat. |
| Hapus pajak yang sudah dipakai transaksi | Error tax rate referenced in existing transactions. | Nonaktifkan saja jika tidak dipakai untuk transaksi baru. |
| Tarif inactive tidak muncul di pilihan transaksi baru | User mengira pajak hilang. | Aktifkan lagi via action activate. |

### Checklist setelah simpan Tarif Pajak

- Tarif baru tampil di daftar.
- Rate tampil sebagai persentase yang benar.
- Status Active jika akan dipakai transaksi.
- Nama/kode mudah dibedakan antara penjualan dan pembelian jika keduanya dibuat.
- Pilih tarif tersebut pada item master sebelum membuat invoice/bill.
- Jangan hapus tarif yang sudah pernah dipakai; gunakan inactive.
- Untuk demo pajak, siapkan satu item taxable dan satu item non-taxable.

## 8. Contoh Data Awal Untuk Presentasi

Bagian ini merangkum data minimal agar presenter tidak perlu menyusun dari nol.

### Bagan Akun minimal

| Code | Name | Type | Catatan setup |
| --- | --- | --- | --- |
| `1001` | Kas | `cash` | Aktif. |
| `1002` | Bank BCA | `bank` | Aktif; dipakai setoran modal dan pembayaran. |
| `1101` | Piutang Usaha | `accounts-receivable` | Wajib untuk invoice/payment receive. |
| `1201` | Persediaan Barang | `inventory` | Dipakai item inventory. |
| `2001` | Utang Usaha | `accounts-payable` | Wajib untuk bill/bill payment. |
| `2101` | Pajak Keluaran | `tax-payable` | Akun pajak utama otomatis. |
| `2102` | Pajak Masukan | `tax-payable` | Akun pendamping/manual untuk demo PPN masukan. |
| `3001` | Modal Pemilik | `equity` | Lawan akun setoran modal. |
| `4001` | Pendapatan Penjualan | `income` | Default sell account barang. |
| `4002` | Pendapatan Jasa | `income` | Default sell account jasa. |
| `5001` | Harga Pokok Penjualan | `cost-of-goods-sold` | Default COGS inventory. |
| `6001` | Beban Operasional | `expense` | Expense umum. |
| `6002` | Beban Administrasi Bank | `expense` | Contoh jurnal biaya bank. |

### Tarif pajak minimal

| Name | Code | Rate | Status |
| --- | --- | ---: | --- |
| PPN 11% Penjualan | `PPN-11-SALES` | 11 | Active |
| PPN 11% Pembelian | `PPN-11-PURCH` | 11 | Active |
| Non Pajak 0% | `NON-TAX` | 0 | Active |

### Jurnal manual minimal

| Jurnal | Debit | Credit | Tujuan |
| --- | --- | --- | --- |
| Setoran modal awal | `1002 - Bank BCA` 50,000,000 | `3001 - Modal Pemilik` 50,000,000 | Membuat saldo awal bank. |
| Biaya admin bank | `6002 - Beban Administrasi Bank` 25,000 | `1002 - Bank BCA` 25,000 | Menunjukkan beban dan pengurangan bank. |
| Koreksi salah akun | `6001 - Beban Operasional` 100,000 | `6002 - Beban Administrasi Bank` 100,000 | Memindahkan klasifikasi beban. |

### Transaction lock minimal

| Skenario | Scope | Tanggal | Reason |
| --- | --- | --- | --- |
| Lock sampai akhir bulan | All Transactions | `2026-04-30` | `Closing April 2026 sudah selesai.` |
| Partial unlock | Sales | `2026-04-22` sampai `2026-04-22` | `Koreksi invoice demo tanggal 22 April 2026.` |
| Cancel partial unlock | Sales | Hapus rentang partial unlock | `Koreksi selesai.` |
| Cancel lock | All atau module terkait | Hapus lock aktif | `Tanggal lock salah atau periode dibuka kembali.` |

### Item singkat untuk menghubungkan finance

Walaupun dokumen ini fokus menu Keuangan, item perlu disiapkan agar invoice/bill demo berjalan.

| Item | Type | Sell account | Purchase/expense account | Tax |
| --- | --- | --- | --- | --- |
| Jasa Konsultasi | `service` | `4002 - Pendapatan Jasa` | `6001 - Beban Operasional` jika dibeli | PPN 11% Penjualan atau Non Pajak |
| Produk Demo | `inventory` | `4001 - Pendapatan Penjualan` | `1201 - Persediaan Barang` dan `5001 - HPP` | PPN 11% Penjualan/Pembelian |

## 9. Contoh Alur Demo Keuangan

### Alur demo 1: setup sampai saldo awal

| Langkah | Menu | Aksi | Hasil yang dijelaskan |
| --- | --- | --- | --- |
| 1 | Bagan Akun | Buat atau tunjukkan akun `1002 Bank BCA`, `3001 Modal Pemilik`, `1101 Piutang Usaha`, `2001 Utang Usaha`, `2101 Pajak Keluaran`. | Akun wajib sudah tersedia. |
| 2 | Tarif Pajak | Buat atau tunjukkan PPN 11% dan Non Pajak 0%. | Pajak siap dipilih di item/transaksi. |
| 3 | Jurnal Manual | Buat `JNL-00001` setoran modal awal. | Jurnal balanced. |
| 4 | Jurnal Manual | Publish jurnal. | Saldo Bank BCA dan Modal Pemilik bertambah. |
| 5 | Detail akun | Buka Bank BCA. | Tampilkan transaksi jurnal manual di riwayat akun. |

Narasi singkat:

> Kita mulai dari Bagan Akun karena semua transaksi harus punya tujuan posting. Setelah akun bank dan modal siap, jurnal setoran modal dipublish agar saldo awal muncul di GL.

### Alur demo 2: pajak dan transaksi sales sederhana

| Langkah | Menu | Aksi | Hasil yang dijelaskan |
| --- | --- | --- | --- |
| 1 | Tarif Pajak | Pastikan PPN 11% Penjualan active. | Tarif dapat dipilih. |
| 2 | Items | Pilih sell tax dan akun pendapatan pada item. | Item membawa default akun dan pajak. |
| 3 | Invoice | Buat invoice taxable. | Sistem membuat AR, revenue, dan tax payable saat deliver/posting sesuai flow. |
| 4 | Bagan Akun/detail akun | Buka Piutang Usaha atau Pajak Keluaran. | Tampilkan dampak GL. |

Narasi singkat:

> Pajak bukan hanya label di invoice. Tarif pajak disimpan pada baris item dan ikut dibawa ke account transactions, sehingga laporan pajak dapat menelusuri asal nilai pajak.

### Alur demo 3: closing dan lock periode

| Langkah | Menu | Aksi | Hasil yang dijelaskan |
| --- | --- | --- | --- |
| 1 | Penguncian Transaksi | Lock All Transactions sampai `2026-04-30`. | Periode April tertutup. |
| 2 | Jurnal Manual/Invoice | Coba transaksi tanggal `2026-04-30`. | Sistem menolak karena locked period. |
| 3 | Penguncian Transaksi | Partial unlock Sales tanggal `2026-04-22`. | Hanya tanggal dan scope tertentu dibuka. |
| 4 | Sales | Koreksi transaksi yang diperlukan. | Koreksi terbatas. |
| 5 | Penguncian Transaksi | Cancel partial unlock. | Periode kembali terkunci penuh. |

Narasi singkat:

> Lock menjaga angka periode closing tidak berubah diam-diam. Partial unlock dipakai sebagai pengecualian yang sempit dan harus ditutup lagi setelah koreksi selesai.

### Alur demo 4: koreksi salah akun

| Langkah | Menu | Aksi | Hasil yang dijelaskan |
| --- | --- | --- | --- |
| 1 | Jurnal Manual | Buat jurnal koreksi dari `6002` ke `6001`. | Total debit dan credit tetap sama. |
| 2 | Publish | Terbitkan jurnal. | GL membuat dua baris koreksi. |
| 3 | Income Statement/detail akun | Cek beban terkait. | Total beban sama, klasifikasi akun berubah. |

Narasi singkat:

> Koreksi tidak menghapus transaksi lama. Sistem membuat jurnal baru yang membalik atau memindahkan klasifikasi agar audit trail tetap jelas.

## 10. Checklist Setelah Setup Menu Keuangan

Gunakan checklist ini sebelum pindah ke demo transaksi customer/vendor.

| Area | Checklist | Status |
| --- | --- | --- |
| Bagan Akun | Minimal ada Cash/Bank, Accounts Receivable, Inventory, Accounts Payable, Tax Payable, Equity, Income, COGS, Expense. |  |
| Bagan Akun | Kode akun tidak duplikat jika unique code aktif. |  |
| Bagan Akun | Akun wajib AR/AP/Tax Payable active. |  |
| Bagan Akun | Akun pendapatan dan beban sudah bisa dipilih di item. |  |
| Bagan Akun | Tidak ada account type yang salah untuk akun utama. |  |
| Tarif Pajak | PPN 11% dan Non Pajak 0% active. |  |
| Tarif Pajak | Tarif penjualan/pembelian mudah dibedakan jika dibuat terpisah. |  |
| Items | Item demo punya sell account, purchase/expense account, dan tax default. |  |
| Jurnal Manual | Setoran modal awal sudah Published. |  |
| Jurnal Manual | Detail akun bank menampilkan saldo dari jurnal. |  |
| Locking | Tidak ada lock yang menghalangi tanggal demo utama. |  |
| Locking | Jika demo closing, lock date dan partial unlock sudah disiapkan. |  |
| Reports | Cek laporan/default accrual yang ingin ditampilkan; jangan tampilkan basis cash untuk report yang belum final kontraknya. |  |

Validasi cepat sebelum presentasi:

| Tes | Ekspektasi |
| --- | --- |
| Buat jurnal draft dengan debit/credit beda | Tombol publish disabled atau backend menolak saat disubmit. |
| Publish jurnal balanced | Status menjadi Published dan detail akun berubah. |
| Buat tarif pajak rate 101 | Validasi menolak. |
| Delete tax rate yang sudah dipakai | Ditolak, gunakan inactive. |
| Lock sampai akhir bulan lalu buat transaksi tanggal locked | Ditolak dengan pesan locked period. |
| Partial unlock tanggal tertentu | Tanggal tersebut bisa dipakai, tanggal lain tetap locked. |

## 11. Checklist Presentasi/Demo

### Persiapan akun user

| Checklist | Catatan |
| --- | --- |
| Login sebagai role Tax/Accounting atau admin. | Tax Rate mengikuti `tax-rate.*`; Transaction Locking tetap butuh admin-level. |
| Role punya permission `account.*`, `manual-journal.*`, dan `tax-rate.*`; `setting.edit` hanya untuk admin-level yang mengelola lock/preferensi. | Sesuaikan dari menu Users/Roles jika perlu. |
| Jangan memakai user dengan akses terbatas untuk setup awal. | Aksi dapat hilang dari UI. |

### Persiapan data

| Checklist | Catatan |
| --- | --- |
| Base currency sesuai demo, misalnya IDR. | Rujuk panduan Preferensi. |
| COA minimal sudah dibuat. | Gunakan tabel data awal di atas. |
| Tarif pajak active. | PPN 11% dan Non Pajak 0%. |
| Item demo sudah punya akun dan pajak. | Agar invoice/bill tidak berhenti di setup item. |
| Jurnal setoran modal sudah published. | Agar bank punya saldo awal yang masuk akal. |
| Lock demo tidak mengganggu transaksi utama. | Cancel lock jika bukan bagian dari demo closing. |

### Narasi demo

| Urutan narasi | Poin yang harus disebut |
| --- | --- |
| Bagan Akun | "Akun adalah tujuan posting semua transaksi." |
| Account Type | "Tipe akun menentukan debit/credit normal dan laporan." |
| Tarif Pajak | "Pajak disiapkan sekali, lalu dipakai item dan transaksi." |
| Jurnal Manual | "Draft belum masuk GL; Published masuk GL." |
| Locking | "Lock melindungi periode yang sudah closing." |
| Partial Unlock | "Pengecualian harus sempit dan ditutup lagi." |

### Hal yang sebaiknya tidak dilakukan saat demo

| Hindari | Alasan |
| --- | --- |
| Mengubah account type akun yang sudah dibuat. | Service memang memblokir perubahan tipe. |
| Menghapus akun dengan transaksi. | Akan ditolak dan mengganggu alur demo. |
| Menghapus tax rate yang sudah dipakai. | Akan ditolak; gunakan inactive. |
| Membuat jurnal tidak seimbang. | Bagus untuk contoh validasi, tetapi jangan jadikan data final. |
| Membuka basis cash untuk Trial Balance/Balance Sheet/Tax Summary. | Kontrak cash-basis untuk area tersebut belum dibuka untuk UI/demo. |
| Membiarkan partial unlock aktif setelah koreksi. | Membuat periode tertutup bisa berubah lagi. |

### Script singkat presenter

1. Buka Bagan Akun dan tunjukkan akun `1002 Bank BCA`, `1101 Piutang Usaha`, `2001 Utang Usaha`, dan `2101 Pajak Keluaran`.
2. Jelaskan bahwa invoice mencari Accounts Receivable dan Tax Payable, sedangkan bill mencari Accounts Payable dan Tax Payable.
3. Buka Tarif Pajak dan tunjukkan PPN 11% serta Non Pajak 0%.
4. Buka Jurnal Manual, buat setoran modal awal, tunjukkan indikator balanced, lalu publish.
5. Buka detail akun Bank BCA untuk menunjukkan transaksi jurnal.
6. Buka Penguncian Transaksi, jelaskan lock sampai akhir bulan dan partial unlock untuk koreksi terbatas.
7. Tutup dengan checklist bahwa setup finance siap dipakai oleh Sales/Purchases/Reports.

## 12. Catatan Field/Menu Yang Belum Terverifikasi

Catatan ini mencegah presenter mengklaim fitur melebihi hasil audit phase ini. Untuk field/menu yang tidak dapat diklaim dari UI atau kontrak kode yang diaudit, gunakan status: **Belum terverifikasi dari kode pada phase ini**.

| Area | Catatan belum terverifikasi |
| --- | --- |
| Opening balance COA | Belum terverifikasi dari kode pada phase ini sebagai input halaman create/edit Bagan Akun. Gunakan Jurnal Manual untuk saldo awal demo. |
| `accounts.balance` sebagai input manual | Belum terverifikasi dari kode pada phase ini sebagai flow UI. Model menyimpan field `balance`, tetapi daftar COA menampilkan saldo terhitung dari transaksi. |
| Pemetaan pajak ke akun pajak tertentu | Belum terverifikasi dari kode pada phase ini. Service transaksi mencari akun berdasarkan `AccountType::TaxPayable`. |
| PPN Masukan terpisah otomatis | Belum terverifikasi dari kode pada phase ini. Akun `2102 Pajak Masukan` dapat disiapkan untuk penjelasan/manual, tetapi posting pajak bill yang diaudit tetap memakai akun Tax Payable yang ditemukan service. |
| Arah pajak sales/purchase di master Tax Rate | Belum terverifikasi dari kode pada phase ini sebagai field `tax_rates`. Pembedaan penjualan/pembelian dilakukan lewat nama/kode dan pemilihan default di item. |
| Currency/exchange rate Jurnal Manual di UI | Belum terverifikasi dari kode pada phase ini sebagai input form React. Backend memvalidasi `currency_code` dan `exchange_rate`. |
| Branch/contact di Jurnal Manual di UI | Belum terverifikasi dari kode pada phase ini sebagai input form React. Backend memvalidasi `branch_id`, `entries.*.branch_id`, dan `entries.*.contact_id`. |
| Detail UX import/export/print COA | Belum terverifikasi dari kode pada phase ini. Route tersedia, tetapi dokumen presentasi ini tidak mengaudit detail UX mapping import/export. |
| Transaction lock audit trail lengkap | Belum terverifikasi dari kode pada phase ini sebagai workflow approval/audit trail khusus. Reason disimpan di settings. |
| Cash-basis Tax Summary | Belum terverifikasi dari kode pada phase ini sebagai fitur expose route/UI/export/PDF. Tetap design-only sesuai catatan proyek. |
| Cash-basis Balance Sheet/Trial Balance | Belum terverifikasi dari kode pada phase ini sebagai fitur expose route/UI/export/PDF. Jangan expose sampai kontrak opening/period/closing balance selesai dan teruji. |
| Inventory/COGS cash-basis | Belum terverifikasi dari kode pada phase ini sebagai kontrak cost-layer/payment source. Jangan infer COGS cash-basis dari inventory/bill/payment. |

Rujukan internal yang dipakai saat audit:

| Area | File utama |
| --- | --- |
| Sidebar menu | `resources/js/layouts/AppLayout.tsx` |
| Bagan Akun | `app/Models/Account.php`, `app/Services/AccountService.php`, `app/Http/Requests/Account/*`, `resources/js/pages/accounting/accounts/*` |
| Jurnal Manual | `app/Models/ManualJournal.php`, `app/Services/ManualJournalService.php`, `app/Http/Requests/Journal/*`, `resources/js/pages/accounting/journals/*` |
| Penguncian Transaksi | `app/Services/TransactionLockService.php`, `app/Http/Controllers/Settings/TransactionLockController.php`, `app/Http/Requests/TransactionLock/*`, `resources/js/pages/settings/accounting.tsx` |
| Tarif Pajak | `app/Models/TaxRate.php`, `app/Services/TaxRateService.php`, `app/Http/Requests/TaxRate/*`, `resources/js/pages/settings/tax-rates.tsx`, `database/seeders/TaxRateSeeder.php` |
