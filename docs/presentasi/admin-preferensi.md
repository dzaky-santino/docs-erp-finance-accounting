# Presentasi Admin Preferensi

## 1. Tujuan Dokumen

Dokumen ini dipakai sebagai bahan presentasi super admin/admin saat menyiapkan menu Preferensi sebelum transaksi pertama dibuat. Isinya bersifat praktis: apa yang diisi, contoh nilai yang aman untuk demo RS UMMI, dampak ke modul lain, error umum, dan checklist validasi.

Semua field di bawah diaudit dari route, Form Request, React page, seeder, dan service terkait. Field yang hanya ditemukan di seeder atau service tetapi tidak muncul sebagai input halaman Preferensi dipisahkan pada bagian catatan agar tidak dipakai sebagai instruksi input demo.

Rujukan silang:

- Setup akun, pajak, dan penguncian transaksi: [admin-keuangan.md](admin-keuangan.md)
- Barang/Jasa, kategori item, gudang, dan stok awal: [admin-barang-jasa.md](admin-barang-jasa.md)
- Customer/vendor dan email dokumen: [admin-kontak.md](admin-kontak.md)
- Laporan yang terdampak preferensi: [admin-laporan.md](admin-laporan.md)

## 2. Gambaran Umum Menu Preferensi

Menu Preferensi pada sidebar utama berisi:

| Sub menu sidebar | Route | Kegunaan utama | Permission/gate |
| --- | --- | --- | --- |
| General | `/settings/organization` | Identitas organisasi, mata uang dasar, tahun fiskal, lokalisasi, logo, dan branding PDF. | Admin-level dan `setting.edit`. |
| Users | `/settings/users` | Invitation user, edit user, activate/deactivate, role list. | Admin-level dan `setting.edit`. |
| Estimates | `/settings/estimates` | Default catatan customer dan syarat estimasi. | Admin-level dan `setting.edit`. |
| Invoices | `/settings/invoices` | Default pesan invoice dan syarat pembayaran. | Admin-level dan `setting.edit`. |
| Receipts | `/settings/receipts` | Default pesan receipt dan statement sale receipt. | Admin-level dan `setting.edit`. |
| Credit Notes | `/settings/credit-notes` | Default catatan customer dan syarat nota kredit. | Admin-level dan `setting.edit`. |
| Currencies | `/settings/currencies` | Master mata uang dan kurs. | Admin-level dan `setting.edit`. |
| Branches | `/settings/branches` | Master cabang operasional. | Admin-level dan `setting.edit`. |
| Warehouses | `/settings/warehouses` | Master gudang inventory. | Admin-level dan `setting.edit`. |
| Accountant | `/settings/accountant` | Basis akuntansi, aturan kode akun, dan akun default payment. | Admin-level dan `setting.edit`. |
| Items | `/settings/item-preferences` | Akun default untuk barang/jasa. | Admin-level dan `setting.edit`. |

Halaman pendukung yang masih berada di bawah `/settings`, tetapi bukan item utama grup Preferensi sidebar:

| Halaman pendukung | Route | Kegunaan |
| --- | --- | --- |
| Tax Rates | `/settings/tax-rates` | Master tarif pajak untuk item dan transaksi. |
| Items master | `/settings/items` | Master barang/jasa yang dijual/dibeli. |
| Item Categories | `/settings/item-categories` | Kategori barang/jasa dan default akun per kategori. |
| Contacts | `/settings/contacts` | Customer dan vendor. |
| Transaction Locking | `/settings/accounting` | Kunci periode transaksi. |

## 3. Urutan Pengisian Yang Disarankan

1. Isi General: nama organisasi, alamat, base currency, tahun fiskal, bahasa, format tanggal, timezone, logo, warna aksen, dan footer dokumen.
2. Review Currencies: pastikan `IDR` tersedia sebagai base currency; tambah kurs hanya bila transaksi multi-currency dipakai.
3. Buat COA minimal: kas/bank, piutang, utang, pajak, pendapatan, beban, HPP, dan persediaan bila inventory dipakai.
4. Review Tax Rates untuk PPN/PPh dan opsi non-pajak.
5. Isi Branches dan Warehouses bila operasional memakai cabang, gudang, stok, inventory adjustment, atau warehouse transfer.
6. Isi Accountant: basis akuntansi, aturan kode akun, default akun deposit customer, withdrawal vendor, dan advance deposit.
7. Isi Items preference, lalu siapkan Item Categories dan Items master.
8. Isi default teks dokumen untuk Estimates, Invoices, Receipts, dan Credit Notes.
9. Buat role kustom, undang user, lalu cek sidebar sesuai permission.
10. Gunakan Transaction Locking hanya setelah periode transaksi sudah direview.

## 4. Sub Menu Umum

### Kegunaan

Sub menu Umum mencakup halaman General dan Currencies. General mengatur identitas organisasi dan preferensi global yang dibagikan ke frontend, laporan, PDF, dan form transaksi. Currencies mengatur master mata uang dan kurs untuk transaksi selain base currency.

### Daftar Input

| Area | Field | Wajib | Validasi utama | Contoh input demo |
| --- | --- | ---: | --- | --- |
| Identitas | `organization_name` | Ya | String, maksimal 100 karakter. | RS UMMI |
| Identitas | `tax_id` | Tidak | String, maksimal 30 karakter. | 01.234.567.8-999.000 |
| Identitas | `organization_industry` | Tidak | String, maksimal 80 karakter. | Kesehatan/Rumah Sakit |
| Identitas | `organization_country` | Tidak | String, maksimal 80 karakter. | Indonesia |
| Alamat | `organization_address` | Tidak | String, maksimal 255 karakter. | Jl. Empang II No. 2 |
| Alamat | `organization_address_2` | Tidak | String, maksimal 255 karakter. | Bogor Selatan |
| Alamat | `organization_city` | Tidak | String, maksimal 80 karakter. | Bogor |
| Alamat | `organization_postal_code` | Tidak | String, maksimal 20 karakter. | 16132 |
| Alamat | `organization_state_province` | Tidak | String, maksimal 80 karakter. | Jawa Barat |
| Kontak | `organization_phone` | Tidak | String, maksimal 30 karakter. | 0251-000000 |
| Keuangan | `base_currency` | Ya | Harus ada di tabel currencies. | IDR |
| Keuangan | `fiscal_year` | Ya | Salah satu bulan Januari sampai Desember dalam backing value bahasa Inggris. | january |
| Lokalisasi | `language` | Ya | `en` atau `id`. | id |
| Lokalisasi | `date_format` | Ya | Format yang didukung UI, misalnya `DD/MM/yyyy`. | DD/MM/yyyy |
| Lokalisasi | `timezone` | Ya | Harus ada di timezone PHP. | Asia/Jakarta |
| Branding | `organization_logo` | Tidak | Gambar `png`, `jpg`, `jpeg`, atau `gif`, maksimal 2 MB dan 1200 x 1200 px. | logo-rs-ummi.png |
| Branding | `show_logo_on_pdf` | Tidak | Boolean `0` atau `1`. | 1 |
| Branding | `document_accent_color` | Tidak | Hex color `#RRGGBB`, maksimal 7 karakter. | #0F766E |
| Branding | `document_footer` | Tidak | String, maksimal 500 karakter. | Terima kasih atas kepercayaan Anda. |
| Mata uang | `code` | Ya | String, maksimal 10 karakter, unik pada row aktif. | IDR |
| Mata uang | `name` | Ya | String, maksimal 255 karakter. | Indonesian Rupiah |
| Mata uang | `symbol` | Tidak | String, maksimal 10 karakter. | Rp |
| Kurs | `currency_code` | Ya | Harus ada di currencies. | USD |
| Kurs | `exchange_rate` | Ya | Numeric lebih besar dari 0. | 16500 |
| Kurs | `date` | Ya | Tanggal valid. | 2026-01-01 |

### Pengaruh Ke Modul Lain

Nama, alamat, logo, warna aksen, dan footer memengaruhi header laporan dan dokumen PDF. `base_currency` menjadi fallback mata uang transaksi dan format nominal. `date_format` dan `timezone` dipakai helper tampilan tanggal di halaman dokumen, laporan, dashboard, dan komponen DatePicker. `fiscal_year` dipakai untuk resolver fiskal internal cash-basis dan quick range "Tahun Fiskal Ini" pada DatePicker. Kurs dipakai saat transaksi menggunakan mata uang selain base currency.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Base currency ditolak | Kode mata uang tidak ada atau sudah soft-deleted. | Tambahkan atau restore currency, lalu pilih ulang di General. |
| Logo ditolak | File bukan gambar yang diizinkan, terlalu besar, atau dimensi terlalu besar. | Gunakan gambar di bawah 2 MB dan maksimal 1200 x 1200 px. |
| Warna aksen ditolak | Format bukan `#RRGGBB`. | Pakai contoh seperti `#0F766E`. |
| Timezone ditolak | Nilai tidak ada di daftar timezone PHP. | Pakai `Asia/Jakarta`, `Asia/Makassar`, atau timezone valid lain. |
| Transaksi multi-currency gagal | Kurs belum dibuat untuk currency dan tanggal transaksi. | Tambahkan exchange rate sebelum transaksi. |
| Validasi form kembali ke halaman yang salah | Browser memakai referer `/` karena URL masking. | Backend Preferences sekarang memakai redirect eksplisit ke submenu asal; ulangi input di submenu yang sama. |

### Checklist

- Nama organisasi, `IDR`, `Asia/Jakarta`, dan `DD/MM/yyyy` tersimpan setelah reload.
- Logo tampil pada preview dan PDF bila `show_logo_on_pdf` aktif.
- Currency selain base currency memiliki exchange rate jika akan dipakai transaksi.
- Materi demo menegaskan prefix contoh `EST`, `INV`, `PAY`, dan `CN` sebagai konvensi nomor, bukan field edit General.

## 5. Sub Menu Estimasi

### Kegunaan

Sub menu Estimasi mengatur teks bawaan saat membuat sales estimate baru. Teks ini membantu tim sales memakai catatan dan syarat yang konsisten.

### Daftar Input

| Field | Wajib | Validasi utama | Contoh input demo |
| --- | ---: | --- | --- |
| `note` | Tidak | String nullable, maksimal 2.000 karakter. | Estimasi ini berlaku 14 hari sejak tanggal penerbitan. |
| `terms_conditions` | Tidak | String nullable, maksimal 2.000 karakter. | Harga belum termasuk perubahan layanan di luar ruang lingkup. |

### Pengaruh Ke Modul Lain

Nilai ini menjadi default teks pada estimate baru dan ikut terbawa ke dokumen customer-facing bila form dokumen menggunakan default tersebut. Untuk demo nomor, gunakan pola `EST-00001` sebagai contoh konvensi dokumen.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Preferensi tidak tersimpan | User bukan admin-level atau tidak punya `setting.edit`. | Login sebagai super admin/admin atau sesuaikan role. |
| Validasi gagal | Teks lebih dari 2.000 karakter. | Ringkas catatan dan terms. |
| Teks tidak muncul pada dokumen lama | Default hanya membantu dokumen baru. | Buat estimate baru setelah preferensi disimpan. |

### Checklist

- Isi customer notes dan terms.
- Simpan, reload, dan pastikan teks tetap ada.
- Buat estimate baru dan cek default teks muncul.

## 6. Sub Menu Faktur

### Kegunaan

Sub menu Faktur mengatur pesan dan syarat default untuk invoice penjualan. Tujuannya membuat komunikasi tagihan konsisten di invoice, PDF, dan mail dokumen.

### Daftar Input

| Field | Wajib | Validasi utama | Contoh input demo |
| --- | ---: | --- | --- |
| `invoice_message` | Tidak | String nullable, maksimal 2.000 karakter. | Terima kasih atas kepercayaan Anda kepada RS UMMI. |
| `terms_conditions` | Tidak | String nullable, maksimal 2.000 karakter. | Pembayaran jatuh tempo sesuai tanggal invoice. |

### Pengaruh Ke Modul Lain

Default ini membantu invoice baru. Setelah invoice dibuat dan delivered, transaksi akan memakai workflow invoice dan GL sesuai service transaksi. Untuk demo nomor, gunakan pola `INV-00001`.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Default invoice tidak berubah | Invoice lama sudah punya teks sendiri. | Uji dengan invoice baru. |
| Teks terlalu panjang | Maksimal 2.000 karakter per field. | Ringkas pesan invoice. |
| User tidak bisa membuka halaman | Tidak punya `setting.edit`. | Update role atau gunakan admin-level. |

### Checklist

- Isi invoice message dan terms.
- Buat invoice baru setelah save.
- Cek PDF/mail invoice memakai pesan yang diharapkan.

## 7. Sub Menu Penerimaan Penjualan

### Kegunaan

Sub menu Penerimaan Penjualan pada route `/settings/receipts` mengatur default teks sale receipt. Ini dipakai untuk penjualan tunai atau penerimaan langsung yang memakai sale receipt.

### Daftar Input

| Field | Wajib | Validasi utama | Contoh input demo |
| --- | ---: | --- | --- |
| `receipt_message` | Tidak | String nullable, maksimal 2.000 karakter. | Pembayaran telah kami terima. |
| `statement` | Tidak | String nullable, maksimal 2.000 karakter. | Bukti penerimaan ini sah setelah dana efektif diterima. |

### Pengaruh Ke Modul Lain

Default ini membantu sale receipt baru. Payment Receive memiliki group setting dan prefix seed `PAY-`, tetapi halaman default teks yang diaudit untuk sub menu ini adalah `sales_receipts`. Untuk demo, gunakan `PAY-00001` hanya saat menjelaskan konvensi nomor Payment Receive.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Default tidak tampil pada receipt lama | Dokumen lama sudah dibuat sebelum setting diubah. | Buat sale receipt baru. |
| Teks ditolak | Melebihi 2.000 karakter. | Ringkas message atau statement. |
| Salah memahami Payment Receive | Receipt default bukan form Payment Receive. | Jelaskan perbedaan sale receipt dan payment receive saat demo. |

### Checklist

- Isi receipt message dan statement.
- Buat sale receipt baru.
- Cek teks pada halaman detail dan PDF receipt.

## 8. Sub Menu Nota Kredit

### Kegunaan

Sub menu Nota Kredit mengatur default catatan dan syarat credit note. Ini membantu tim sales/accounting memberi komunikasi standar saat mengoreksi invoice customer.

### Daftar Input

| Field | Wajib | Validasi utama | Contoh input demo |
| --- | ---: | --- | --- |
| `note` | Tidak | String nullable, maksimal 2.000 karakter. | Nota kredit diterbitkan untuk koreksi layanan. |
| `terms_conditions` | Tidak | String nullable, maksimal 2.000 karakter. | Nilai dapat diterapkan ke invoice terbuka atau direfund sesuai persetujuan. |

### Pengaruh Ke Modul Lain

Default ini membantu credit note baru. Credit note yang dibuka/refund tetap membutuhkan akun piutang yang benar di COA. Untuk demo nomor, gunakan pola `CN-00001`.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Open/refund credit note gagal | Required account `AccountsReceivable` belum ada. | Buat akun Piutang Usaha dengan tipe `accounts-receivable`. |
| Default tidak muncul | Dokumen dibuat sebelum setting disimpan. | Buat credit note baru setelah save. |
| Teks ditolak | Melebihi 2.000 karakter. | Ringkas catatan. |

### Checklist

- Isi note dan terms.
- Pastikan akun Piutang Usaha tersedia.
- Buat credit note baru dan cek default teks.

## 9. Sub Menu Cabang

### Kegunaan

Sub menu Cabang menyimpan lokasi operasional seperti kantor pusat, klinik, atau unit bisnis. Cabang membantu pemisahan data operasional bila transaksi memakai branch.

### Daftar Input

| Field | Wajib | Validasi utama | Contoh input demo |
| --- | ---: | --- | --- |
| `name` | Ya | String, maksimal 255 karakter. | Cabang Utama |
| `code` | Tidak | String, maksimal 50 karakter, unik pada row aktif. | HO |
| `address` | Tidak | String, maksimal 1.000 karakter. | Jl. Empang II No. 2 |
| `city` | Tidak | String, maksimal 255 karakter. | Bogor |
| `country` | Tidak | String, maksimal 255 karakter. | Indonesia |
| `phone_number` | Tidak | String, maksimal 50 karakter. | 0251-000000 |
| `email` | Tidak | Email, maksimal 255 karakter. | cabang.utama@rs-ummi.test |
| `website` | Tidak | String, maksimal 255 karakter. | rs-ummi.test |
| `is_primary` | Tidak | Boolean. | Aktif |

### Pengaruh Ke Modul Lain

Branch dapat menjadi referensi operasional untuk transaksi dan laporan yang memakai dimensi cabang. Cabang primary membantu admin mengenali cabang utama pada tabel.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Code ditolak | Kode sama dengan cabang aktif lain. | Gunakan kode unik seperti `HO` atau `BR-01`. |
| Email ditolak | Format email tidak valid. | Gunakan email valid. |
| Tidak bisa edit/delete | User bukan admin-level atau tidak punya `setting.edit`. | Sesuaikan role. |

### Checklist

- Buat `Cabang Utama` dengan kode `HO`.
- Tandai primary bila ini lokasi default.
- Edit sekali dan pastikan perubahan tersimpan.

## 10. Sub Menu Gudang

### Kegunaan

Sub menu Gudang menyimpan lokasi penyimpanan stok. Gudang penting untuk inventory item, inventory adjustment, dan warehouse transfer.

### Daftar Input

| Field | Wajib | Validasi utama | Contoh input demo |
| --- | ---: | --- | --- |
| `name` | Ya | String, maksimal 255 karakter. | Gudang Utama |
| `code` | Tidak | String, maksimal 50 karakter, unik pada row aktif. | WH-01 |
| `address` | Tidak | String, maksimal 1.000 karakter. | Area logistik RS UMMI |
| `city` | Tidak | String, maksimal 255 karakter. | Bogor |
| `country` | Tidak | String, maksimal 255 karakter. | Indonesia |
| `phone_number` | Tidak | String, maksimal 50 karakter. | 0251-000001 |
| `email` | Tidak | Email, maksimal 255 karakter. | gudang@rs-ummi.test |
| `website` | Tidak | String, maksimal 255 karakter. | rs-ummi.test |
| `is_primary` | Tidak | Boolean. | Aktif |

### Pengaruh Ke Modul Lain

Inventory adjustment dan warehouse transfer membutuhkan gudang yang benar. Item inventory juga akan terlihat lebih masuk akal dalam demo bila minimal satu gudang tersedia.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Tidak bisa membuat warehouse transfer | Gudang asal/tujuan belum ada atau sama. | Buat minimal dua gudang bila ingin demo transfer. |
| Code ditolak | Kode sama dengan gudang aktif lain. | Gunakan kode unik seperti `WH-01`. |
| Stok tidak cukup | Inventory item belum punya stok di gudang terkait. | Jalankan inventory adjustment sesuai prosedur. |

### Checklist

- Buat `Gudang Utama` dengan kode `WH-01`.
- Tandai primary bila menjadi gudang default operasional.
- Siapkan gudang kedua bila demo warehouse transfer.

## 11. Sub Menu Akuntansi

### Kegunaan

Sub menu Akuntansi pada presentasi ini merujuk ke Preferences > Accountant dan halaman pendukung Transaction Locking. Accountant mengatur basis akuntansi dan akun default pembayaran. Transaction Locking mengunci periode agar data yang sudah direview tidak berubah sembarangan.

### Daftar Input Accountant

| Field | Wajib | Validasi utama | Contoh input demo |
| --- | ---: | --- | --- |
| `accounting_basis` | Ya | Enum `accrual` atau `cash`. | accrual |
| `account_code_required` | Ya | Boolean. | true |
| `account_code_unique` | Ya | Boolean. | true |
| `preferred_deposit_account` | Tidak | Account type `cash`, `bank`, atau `other-current-asset`. | Bank BCA |
| `withdrawal_account` | Tidak | Account type `cash`, `bank`, atau `other-current-asset`. | Bank BCA |
| `preferred_advance_deposit` | Tidak | Account type `cash`, `bank`, `accounts-receivable`, `inventory`, atau `other-current-asset`. | Uang Muka Pelanggan |

### Daftar Input Transaction Locking

| Aksi | Field | Wajib | Validasi utama | Contoh input demo |
| --- | --- | ---: | --- | --- |
| Lock | `module` | Ya | `all`, `sales`, `purchases`, atau `financial`. | all |
| Lock | `lock_to_date` | Ya | Tanggal valid. | 2026-03-31 |
| Lock | `reason` | Tidak | String, maksimal 1.000 karakter. | Closing Maret selesai. |
| Partial unlock | `unlock_from_date` | Ya | Tanggal valid. | 2026-03-15 |
| Partial unlock | `unlock_to_date` | Ya | Tanggal valid dan setelah/sama dengan from date. | 2026-03-20 |
| Cancel lock | `reason` | Tidak | String, maksimal 1.000 karakter. | Koreksi audit. |

### Pengaruh Ke Modul Lain

Basis `accrual` adalah contoh presentasi yang paling aman untuk finance/accounting umum. Account code required/unique memengaruhi COA. Akun deposit dan withdrawal mempercepat Payment Receive dan Bill Payment. Transaction Locking menolak perubahan transaksi pada periode yang terkunci.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Akun default tidak muncul | AccountType tidak sesuai rule field. | Buat akun Cash/Bank/Other Current Asset sesuai kebutuhan. |
| Basis cash membingungkan pada laporan | Cash basis belum diekspos untuk Balance Sheet, Trial Balance, dan Tax Summary. | Jelaskan bahwa pilihan ini preferensi umum, bukan toggle laporan tersebut. |
| Transaksi ditolak karena tanggal terkunci | Lock aktif sampai tanggal transaksi. | Gunakan partial unlock atau cancel lock sesuai SOP. |
| Partial unlock ditolak | Tanggal akhir sebelum tanggal awal. | Pastikan range tanggal valid. |

### Checklist

- Pilih basis Akrual (`accrual`) untuk demo utama.
- Aktifkan account code unique.
- Pilih Bank BCA sebagai akun deposit dan withdrawal bila COA sudah tersedia.
- Uji satu transaksi pada periode terkunci hanya bila bagian demo membutuhkan kontrol close period.

## 12. Sub Menu Barang/Jasa

### Kegunaan

Sub menu Barang/Jasa mencakup Preferences > Items untuk akun default, serta halaman pendukung Items master dan Item Categories. Bagian ini menentukan item apa yang bisa dijual, dibeli, menjadi inventory, dan akun mana yang dipakai saat item masuk transaksi.

### Daftar Input Item Preferences

| Field | Wajib | Validasi utama | Contoh input demo |
| --- | ---: | --- | --- |
| `preferred_sell_account` | Tidak | Account type `income` atau `other-income`. | Pendapatan Jasa |
| `preferred_cost_account` | Tidak | Account type `expense`, `other-expense`, atau `cost-of-goods-sold`. | Beban Operasional |
| `preferred_inventory_account` | Tidak | Account type `inventory`. | Persediaan |

### Daftar Input Item Categories

| Field | Wajib | Validasi utama | Contoh input demo |
| --- | ---: | --- | --- |
| `name` | Ya | String, maksimal 255 karakter. | Layanan Medis |
| `description` | Tidak | String, maksimal 255 karakter. | Kategori jasa layanan kesehatan |
| `sell_account_id` | Tidak | Account ID valid. | Pendapatan Jasa |
| `cost_account_id` | Tidak | Account ID valid. | Beban Operasional |
| `inventory_account_id` | Tidak | Account ID valid. | Persediaan |

### Daftar Input Items Master

| Field | Wajib | Validasi utama | Contoh input demo |
| --- | ---: | --- | --- |
| `name` | Ya | String, maksimal 50 karakter. | Jasa Konsultasi |
| `type` | Ya | Enum backing value `service`, `inventory`, atau `non-inventory`. | service |
| `code` | Tidak | String, maksimal 50 karakter, unik pada row aktif. | JASA-KONSULTASI |
| `category_id` | Tidak | ID kategori item valid. | Layanan Medis |
| `is_sellable` | Tidak | Boolean; bila aktif, field penjualan wajib. | Aktif |
| `sell_price` | Ya bila sellable | Numeric minimal 0. | 1500000 |
| `sell_account_id` | Ya bila sellable | Account ID valid, opsi UI difilter ke Income/Other Income. | Pendapatan Jasa |
| `sell_tax_rate_id` | Tidak | Tax rate ID valid. | PPN 11% |
| `sell_description` | Tidak | String, maksimal 100 karakter. | Layanan konsultasi dokter |
| `is_purchasable` | Tidak | Boolean; bila aktif, field pembelian wajib. | Nonaktif untuk jasa demo |
| `cost_price` | Ya bila purchasable | Numeric minimal 0. | 750000 |
| `cost_account_id` | Ya bila purchasable | Account ID valid, opsi UI difilter ke Expense/Other Expense/COGS. | Beban Operasional |
| `purchase_tax_rate_id` | Tidak | Tax rate ID valid. | PPN 11% |
| `purchase_description` | Tidak | String, maksimal 100 karakter. | Biaya layanan vendor |
| `inventory_account_id` | Ya bila type inventory | Account ID valid, opsi UI difilter ke Inventory. | Persediaan Obat |

### Pengaruh Ke Modul Lain

Item sellable muncul pada estimate, invoice, dan sale receipt. Item purchasable muncul pada bill dan vendor credit. Item inventory membutuhkan akun inventory dan memengaruhi stok, inventory adjustment, warehouse transfer, serta report inventory. Kategori item mempercepat pemilihan akun default pada item.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Item sellable ditolak | Harga jual atau akun pendapatan kosong. | Isi `sell_price` dan pilih akun Income/Other Income. |
| Item purchasable ditolak | Harga beli atau akun biaya kosong. | Isi `cost_price` dan pilih Expense/Other Expense/COGS. |
| Item inventory ditolak | Akun inventory kosong. | Pilih akun bertipe `inventory`. |
| Item tidak muncul di transaksi sales | `is_sellable` tidak aktif. | Aktifkan sellable dan lengkapi field penjualan. |
| Item tidak muncul di transaksi purchases | `is_purchasable` tidak aktif. | Aktifkan purchasable dan lengkapi field pembelian. |
| Stok tidak terlihat cukup | Belum ada stok di warehouse terkait. | Siapkan inventory adjustment sebelum demo penjualan inventory. |

### Checklist

- Buat kategori `Layanan Medis`.
- Buat item service `JASA-KONSULTASI` dengan sellable aktif.
- Buat item inventory hanya setelah akun Persediaan dan Gudang Utama tersedia.
- Cek dropdown item pada invoice dan bill sesuai flag sellable/purchasable.

## 13. Sub Menu Pengguna, Role, dan Akses

### Kegunaan

Users mengatur akun login dan status user. Role mengelompokkan akses berdasarkan pekerjaan, sedangkan permission menentukan tindakan kecil yang boleh dilakukan. Setup role perlu diselesaikan sebelum production agar user harian hanya melihat menu dan tombol yang relevan dengan tanggung jawabnya.

User management memakai admin-level access; tidak ada permission `user.*` pada seeder. Pengelolaan user dan role dilakukan dari Preferences > Users oleh Super Admin/Admin, bukan oleh role operasional biasa.

### Konsep User, Role, dan Permission

| Konsep | Penjelasan awam | Contoh |
| --- | --- | --- |
| User | Akun login milik satu orang. Jangan dipakai bersama oleh beberapa orang. | `finance@rs-ummi.test` |
| Role | Jabatan atau kelompok akses yang ditugaskan ke user. | `Sales Staff`, `Purchasing/AP Supervisor` |
| Permission | Izin detail berbentuk `{module}.{action}`. Permission menentukan menu, halaman, atau aksi yang boleh dipakai role. | `sale-invoice.view`, `bill.create` |

Arti aksi permission:

| Aksi | Arti praktis | Catatan |
| --- | --- | --- |
| `.view` | Boleh melihat menu, halaman, dan data modul terkait. | Biasanya menjadi akses minimal sebelum aksi lain diberikan. |
| `.create` | Boleh membuat record baru. | Beberapa transaksi langsung memengaruhi akuntansi saat disimpan, misalnya Bill Payment. |
| `.edit` | Boleh mengubah record. | Pada beberapa modul, permission ini juga menjaga aksi status seperti deliver, open, publish, close, initiate, atau apply. |
| `.delete` | Boleh menghapus record jika service masih mengizinkan. | Berikan terbatas karena delete guard tiap modul berbeda. |
| `.refund` | Boleh melakukan refund pada modul yang menyediakan permission khusus. | Tersedia pada Credit Note dan Vendor Credit, tetapi route Credit Note web juga menerima permission create/edit; baca strategi approval pada section 15. |
| `.writeoff` | Boleh melakukan write-off invoice. | Tersedia sebagai aksi khusus invoice, tetapi route web juga menerima `sale-invoice.edit`; baca strategi approval pada section 15. |
| `report-*.view` | Boleh membuka laporan tertentu. | Pilih laporan granular sesuai kebutuhan user; jangan memberi semua laporan tanpa review. |

### Perbedaan Super Admin, Admin, dan Role Kustom

| Jenis role | Kegunaan | Cocok untuk | Catatan keamanan |
| --- | --- | --- | --- |
| Super Admin | Kontrol penuh sistem dan bootstrap awal. | Owner sistem atau IT utama. | Jangan dipakai untuk transaksi harian. Role ini tidak assignable dari UI biasa. |
| Admin | Pengelola operasional dan Preferensi dengan akses sangat luas. | Kepala admin atau finance lead yang benar-benar mengelola setting global. | Role predefined ini hanya dapat ditugaskan oleh Super Admin. |
| Custom Role | Akses sesuai pekerjaan harian. | Staff sales, purchases, inventory, banking, accounting, project, management, atau auditor. | Gunakan sebagai default untuk user operasional. Pilih checkbox permission secukupnya. |

Super Admin dan Admin adalah role predefined dari seeder dan memperoleh seluruh permission. Role predefined tidak dapat diedit atau dihapus lewat UI. Permission `setting.view` dan `setting.edit` tetap sensitif, tampil sebagai admin-only notice, dan tidak menjadi checkbox role kustom. Jangan memberi akses Admin hanya karena user perlu menginput transaksi.

### Daftar Input Users

| Flow | Field | Wajib | Validasi utama | Contoh input demo |
| --- | --- | ---: | --- | --- |
| Invite | `email` | Ya | Email, maksimal 255 karakter. | finance@rs-ummi.test |
| Invite | `role` | Ya | Nama role valid dan assignable. | Finance |
| Edit user | `name` | Ya pada dialog edit | String, maksimal 255 karakter. | Finance RS UMMI |
| Edit user | `email` | Ya pada dialog edit | Email, maksimal 255 karakter, unik pada row aktif. | finance@rs-ummi.test |
| Edit user | `role` | Ya pada dialog edit | Nama role valid dan assignable. | Finance |
| Direct-create/API internal | `password` | Ya | String, minimal 8 dan maksimal 255 karakter. | Gunakan sesuai kebijakan internal. |

### Daftar Input Roles

| Field | Wajib | Validasi utama | Contoh input demo |
| --- | ---: | --- | --- |
| `name` | Ya | String, maksimal 255 karakter, unik untuk guard `web`, bukan `super-admin` atau `admin`. | Finance |
| `description` | Tidak | String, maksimal 1.000 karakter. | Akses pembayaran, jurnal, dan laporan finance. |
| `permissions` | Ya | Array minimal satu permission valid. | `payment-receive.view`, `bill-payment.create` |

### Grup Permission di Role UI

| Grup Role UI | Menu yang dicakup | Contoh permission | Kapan diberikan |
| --- | --- | --- | --- |
| Items & Inventory | Barang/Jasa, Penyesuaian Persediaan, Transfer Gudang | `item.view`, `inventory-adjustment.create`, `warehouse-transfer.create` | Staff master item, gudang, atau inventory. |
| Contacts | Customer dan vendor | `contact.view`, `contact.create` | Staff master data, sales, atau purchasing. |
| Sales | Estimasi, Faktur, Sale Receipt, Payment Receive, Nota Kredit | `sale-invoice.create`, `payment-receive.create`, `credit-note.refund` | Sales, AR, atau supervisor sales. |
| Purchases | Bill, Bill Payment, Vendor Credit | `bill.create`, `bill-payment.create`, `vendor-credit.refund` | Purchasing, AP, atau supervisor purchasing. |
| Expenses | Biaya langsung | `expense.create`, `expense.edit` | Staff finance yang mencatat pengeluaran langsung. |
| Financial Accounting | Bagan Akun, Jurnal Manual, Tarif Pajak | `account.view`, `manual-journal.create`, `tax-rate.edit` | Accounting atau controller. |
| Banking | Banking/Cashflow | `cashflow.view`, `cashflow.create` | Cashier atau staff banking. |
| Projects | Proyek | `project.view`, `project.create` | Project admin atau staff yang mengelola master project. |
| Financial Reports | Laporan granular | `report-income-statement.view`, `report-general-ledger.view` | Management, accounting, atau auditor. |
| Preferences/Admin | Setting global, user, dan role | `setting.view`, `setting.edit` | Admin-only. Permission ini tampil sebagai notice dan tidak dipilih dari role kustom biasa. |

Notasi `module.*` pada dokumen ini adalah singkatan untuk kelompok permission aktual. Di Role UI, pilih checkbox per aksi yang diperlukan. Role form juga menerapkan dependensi bertingkat: memilih edit atau delete dapat ikut membutuhkan permission dasar modul terkait.

### Cara Membuat Role Baru

1. Buka Preferences > Users.
2. Masuk ke tab Roles.
3. Klik Create Role.
4. Isi nama role yang mudah dikenali.
5. Isi deskripsi singkat tanggung jawab role.
6. Centang permission sesuai pekerjaan user.
7. Simpan role.
8. Undang user dari tab Users dan pilih role tersebut.
9. Login sebagai user itu untuk memeriksa sidebar, halaman, tombol aksi, dan laporan.

### Cara Menguji Role

- Menu yang seharusnya tampil sudah muncul di sidebar.
- Menu yang tidak boleh diakses tidak muncul.
- Akses URL langsung ke halaman yang tidak boleh dibuka menghasilkan 403.
- Tombol create, edit, delete, refund, atau write-off tidak muncul jika permission terkait tidak ada.
- User view-only tidak dapat menyimpan atau menghapus data.
- Daftar laporan hanya memuat laporan sesuai permission granular.
- Permission sensitif diuji memakai data demo, bukan transaksi production.

### Pengaruh Ke Modul Lain

Permission menentukan sidebar, route, dan tombol aksi. Perubahan role juga memengaruhi akses user yang memakai role tersebut. Gunakan role kustom untuk pekerjaan harian, lalu simpan keputusan pemberian akses sensitif dalam SOP internal.

### Error Umum

| Gejala | Penyebab umum | Tindakan |
| --- | --- | --- |
| Menu tidak muncul | Role tidak punya permission `.view` atau route memang admin-level seperti Preferences atau Transaction Locking. | Tambahkan permission view yang sesuai atau gunakan user Admin untuk area admin. |
| Bisa melihat tetapi tidak bisa menyimpan | Role tidak punya permission `.create` atau `.edit`. | Tambahkan aksi sesuai kebutuhan kerja. |
| Tombol refund tidak muncul | Role tidak punya permission `.refund` pada modul yang menjaganya secara khusus. | Berikan hanya ke supervisor setelah review SOP. |
| 403 saat membuka URL langsung | Route dilindungi permission yang belum dimiliki. | Review role pada tab Roles. |
| User bisa terlalu banyak akses | Role terlalu luas atau user diberi Admin. | Buat role kustom lebih spesifik dan uji ulang login. |
| Preferensi tidak bisa dibuka | `setting.edit` admin-only dan route juga memakai guard admin-level. | Gunakan user Admin untuk setting global. |
| Invite gagal | Email tidak valid, role tidak assignable, atau email sudah dipakai. | Pakai email valid dan role non-predefined yang assignable. |
| Role gagal disimpan | Nama role reserved atau permissions kosong. | Gunakan nama role lain dan pilih minimal satu permission. |
| Role gagal dihapus | Role masih dipakai user atau predefined. | Pindahkan user ke role lain terlebih dahulu. |

### Checklist

- Buat role kustom sebelum invite user.
- Invite user dengan role yang tepat.
- Login sebagai user demo dan cek sidebar sesuai role.
- Pastikan user tanpa `setting.edit` tidak bisa mengubah Preferensi.
- Uji URL langsung dan tombol aksi sensitif memakai user non-admin.
- Berikan refund, write-off, dan delete hanya kepada supervisor yang memang membutuhkan.
- Jangan gunakan akun Super Admin untuk operasional harian demo.

## 14. Contoh Role Untuk Finance & Accounting

Template berikut adalah titik awal. Sesuaikan dengan pemisahan tugas organisasi, lalu manual test setiap role. Permission berakhiran `.*` adalah singkatan dokumentasi; pada Role UI pilih checkbox aksi aktual satu per satu.

| Role | Cocok untuk | Permission minimal | Permission opsional | Permission sensitif yang jangan sembarang diberikan | Menu yang biasanya terlihat | Manual test singkat |
| --- | --- | --- | --- | --- | --- | --- |
| Super Admin | Owner sistem atau IT utama. | Semua permission dari seeder. | Tidak ada. | Seluruh akses sudah aktif. Jangan pakai harian. | Semua menu. | Cek bootstrap, Admin cadangan, dan akses Preferences. |
| Admin Operasional | Kepala admin atau finance lead pengelola setting global. | Gunakan role predefined `admin`; seluruh permission aktif. | Tidak ada. | `setting.edit`, user/role management, lock periode, refund, write-off, delete. | Semua menu operasional dan Preferences. | Cek Preferences, invite user, role list, dan Transaction Locking. |
| Master Data Officer | Staff yang memelihara customer, vendor, dan master data dasar. | `contact.view`, `contact.create`, `contact.edit`, `item.view`. | `item.create`, `item.edit`, `project.view`, `project.create`, `project.edit`. | `contact.delete`, `item.delete`, `project.delete`. | Contacts, Items, dan Projects bila diberikan. | Buat customer demo; pastikan Settings global tidak terbuka. |
| Items & Inventory Officer | Staff gudang atau inventory. | `item.view`, `inventory-adjustment.view`, `inventory-adjustment.create`, `warehouse-transfer.view`, `warehouse-transfer.create`. | `item.create`, `item.edit`, `inventory-adjustment.edit`, `warehouse-transfer.edit`, report inventory terkait. | Semua `.delete`; `.edit` inventory adjustment dapat publish dan `.edit` transfer dapat initiate/deliver. | Items, Inventory Adjustment, Warehouse Transfer, report inventory terpilih. | Buat draft adjustment dan transfer; cek aksi status sesuai SOP. |
| Sales Staff | Staff yang menyiapkan draft sales. | `contact.view`, `item.view`, `sale-estimate.view`, `sale-estimate.create`, `sale-invoice.view`, `sale-invoice.create`, `sale-receipt.view`, `sale-receipt.create`, `credit-note.view`, `credit-note.create`. | Permission `.edit` sales bila staff boleh menjalankan aksi status sesuai SOP. | `sale-invoice.writeoff`, `credit-note.refund`, semua `.delete`; `.edit` juga dapat membuka aksi status tertentu. | Contacts, Items, Estimates, Invoices, Sale Receipts, Credit Notes. | Buat draft dokumen; cek aksi posting yang tidak disetujui tidak dipakai. |
| Sales Supervisor / AR | Supervisor sales atau pengelola piutang. | `sale-estimate.view`, `sale-estimate.create`, `sale-estimate.edit`, `sale-invoice.view`, `sale-invoice.create`, `sale-invoice.edit`, `sale-receipt.view`, `sale-receipt.create`, `sale-receipt.edit`, `payment-receive.view`, `payment-receive.create`, `payment-receive.edit`, `credit-note.view`, `credit-note.create`, `credit-note.edit`, report AR terkait. | `.delete` terpilih, `sale-invoice.writeoff`, `credit-note.refund`. | Write-off, refund, dan delete. | Seluruh menu Sales, Payment Receive, report AR terpilih. | Deliver invoice, record payment, lalu uji write-off/refund hanya pada data demo. |
| Purchasing/AP Staff | Staff yang menyiapkan transaksi pembelian. | `contact.view`, `item.view`, `bill.view`, `bill.create`, `vendor-credit.view`, `vendor-credit.create`, `bill-payment.view`. | `bill.edit`, `vendor-credit.edit`, `bill-payment.create`, `bill-payment.edit`. | Refund, delete; `bill.edit` dapat open bill, `vendor-credit.edit` dapat open/apply, dan `bill-payment.create` langsung memposting pembayaran. | Contacts, Items, Bills, Vendor Credits, Payments Made sesuai permission. | Buat draft bill; cek open dan pembayaran mengikuti SOP. |
| Purchasing/AP Supervisor | Supervisor purchasing atau AP lead. | `bill.view`, `bill.create`, `bill.edit`, `vendor-credit.view`, `vendor-credit.create`, `vendor-credit.edit`, `bill-payment.view`, `bill-payment.create`, `bill-payment.edit`, report AP terkait. | `.delete` terpilih, `vendor-credit.refund`. | Refund vendor credit, delete, dan koreksi payment. | Seluruh menu Purchases dan report AP terpilih. | Open bill, buat payment demo, apply/refund vendor credit sesuai hak akses. |
| Cashier / Banking | Kasir atau staff bank. | `cashflow.view`, `cashflow.create`. | `cashflow.edit`, `cashflow.delete`, `payment-receive.view`, `payment-receive.create`, `bill-payment.view`, `bill-payment.create`, report cash flow terkait. | Delete cashflow dan transaksi pembayaran yang langsung memengaruhi kas/bank. | Banking, Payment Receive, Payments Made, dan report kas terpilih. | Buat transaksi bank demo dan cek dampak ledger kas/bank. |
| Expense Staff | Staff pencatat biaya langsung bayar. | `expense.view`, `expense.create`. | `expense.edit`, report expense/P&L terkait. | `expense.delete`; `expense.edit` juga dapat publish. | Expenses dan report terpilih. | Buat draft expense; cek publish mengikuti SOP. |
| Accounting Staff | Staff accounting harian. | `account.view`, `manual-journal.view`, `manual-journal.create`, `tax-rate.view`, report GL dan Journal Sheet terkait. | `manual-journal.edit`, `expense.view`, report keuangan lain. | `manual-journal.delete`; `.edit` jurnal juga dapat publish. | Chart of Accounts, Manual Journals, Tax Rates read-only, report terpilih. | Buat draft jurnal balanced dan cek publish sesuai SOP. |
| Accounting Supervisor / Controller | Controller atau accounting lead. | `account.*`, `manual-journal.*`, `tax-rate.*`, `expense.view`, `expense.edit`, report financial terkait. | `expense.delete`, report lain sesuai kebutuhan. Jika perlu setting global atau lock periode, gunakan role predefined `admin` setelah keputusan akses khusus. | Delete, publish jurnal/expense, perubahan COA/tax rate, dan akses Admin bila diberikan. | Financial Accounting, Expenses, Reports; Preferences hanya bila diprovision sebagai Admin. | Publish jurnal demo, review COA/tax, dan cek akses lock periode sesuai keputusan. |
| Project Admin | Staff pengelola master proyek. | `project.view`, `project.create`, `project.edit`. | `project.delete`, `report-project-profitability.view`, `report-general-ledger.view`. | Delete proyek. | Projects dan report proyek terpilih. | Buat proyek demo dan pastikan report proyek hanya terbuka bila diberikan. |
| Report Viewer / Management | Manajemen yang membaca laporan tertentu. | Pilih `report-*.view` granular sesuai kebutuhan. | `account.view`, `contact.view`, `item.view` untuk drill-down terbatas. | Hindari create, edit, delete, refund, write-off, dan `setting.edit`. | Hanya laporan terpilih dan menu referensi read-only bila diberikan. | Buka laporan yang diizinkan dan pastikan transaksi tidak dapat diubah. |
| Auditor | Auditor internal/eksternal read-only. | Pilih `report-*.view` granular, `account.view`. | `contact.view`, `item.view`, report tambahan sesuai ruang lingkup audit. | Hindari semua permission mutasi dan `setting.edit`. | Reports, Chart of Accounts, dan referensi read-only terpilih. | Uji URL create/edit langsung menghasilkan 403 dan laporan audit dapat dibuka. |

Catatan keamanan template:

- Role Staff sebaiknya dimulai dari permission minimum, lalu ditambah berdasarkan hasil manual test.
- Pada banyak modul, permission `.edit` mencakup perubahan draft sekaligus aksi status. Jangan menganggap `.edit` selalu berarti edit draft saja.
- Permission refund, write-off, dan delete harus dibatasi ke supervisor. Credit Note web memiliki caveat khusus pada section 15.
- Pilih `report-*.view` satu per satu sesuai kebutuhan. Hindari memberi seluruh laporan hanya karena user memerlukan satu report.
- Accounting Supervisor yang perlu Preferences atau Transaction Locking harus diprovision sebagai Admin berdasarkan keputusan akses khusus. Checkbox `setting.edit` tidak tersedia pada role kustom biasa.

## 15. Strategi Approval Sederhana Dengan Role Saat Ini

Sistem saat ini lebih kuat pada status dokumen dan permission, bukan workflow approval formal multi-user. Organisasi dapat memakai pola sederhana: staff membuat draft atau menyiapkan transaksi, lalu supervisor menjalankan aksi status sensitif sesuai SOP dan permission yang tersedia.

Pola ini membantu operasional sebelum ada workflow approval formal, tetapi pemisahannya belum selalu dapat ditegakkan per aksi. Pada beberapa route, permission `.edit` menjaga perubahan draft sekaligus aksi open, deliver, publish, close, initiate, atau apply.

| Proses | Staff | Supervisor/Admin operasional | Catatan route dan status aktual |
| --- | --- | --- | --- |
| Estimasi | Buat draft; edit hanya bila SOP mengizinkan. | Deliver, approve, reject, dan convert bila diberi `sale-estimate.edit`. | Seluruh aksi status tersebut memakai `sale-estimate.edit`; belum ada permission approve terpisah. |
| Faktur | Buat draft. | Deliver, delete, atau write-off sesuai SOP. | Deliver memakai `sale-invoice.edit`. Write-off route web menerima `sale-invoice.writeoff` atau `sale-invoice.edit`, sehingga permission write-off belum menjadi pemisah supervisor yang ketat. |
| Sale Receipt | Buat transaksi sesuai SOP. | Close atau delete sesuai SOP. | Close memakai `sale-receipt.edit`. |
| Nota Kredit | Buat dokumen koreksi. | Open, apply, refund, atau delete sesuai SOP. | Open memakai `credit-note.create`, apply memakai `.edit`, sedangkan refund route web menerima `.refund`, `.edit`, atau `.create`. Refund Credit Note belum dapat dipisahkan ketat hanya melalui permission web saat ini. |
| Bill | Buat draft. | Open atau delete sesuai SOP. | Open memakai `bill.edit`; Bill memengaruhi AP setelah dibuka. |
| Vendor Credit | Buat dokumen koreksi. | Open/apply dengan `.edit`; refund dengan `.refund`; delete sesuai SOP. | Refund dan delete refund Vendor Credit sudah memakai `vendor-credit.refund` pada web dan API v1. |
| Payment Receive | Input pembayaran sesuai SOP. | Review, koreksi, atau delete sesuai SOP. | Create langsung memengaruhi AR dan kas/bank; tidak ada tahap approval formal. |
| Bill Payment | Input pembayaran sesuai SOP. | Review, koreksi, atau delete sesuai SOP. | Create langsung memengaruhi AP dan kas/bank. Hati-hati bila sudah matched dengan transaksi bank. |
| Expense | Buat draft. | Publish atau delete sesuai SOP. | Publish memakai `expense.edit`; belum ada permission publish terpisah. |
| Manual Journal | Buat draft balanced. | Publish atau delete sesuai SOP ketat. | Publish memakai `manual-journal.edit`; belum ada permission publish terpisah. |
| Inventory Adjustment | Buat draft koreksi stok. | Publish atau delete sesuai SOP. | Publish memakai `inventory-adjustment.edit`; memengaruhi stok dan nilai persediaan. |
| Warehouse Transfer | Buat draft transfer. | Initiate, deliver, atau delete sesuai SOP. | Initiate/deliver memakai `warehouse-transfer.edit`; memengaruhi lokasi stok. |
| Reports | Tidak melakukan mutasi. | Management atau auditor melihat report terpilih. | Gunakan permission granular `report-*.view`. |

Strategi implementasi operasional:

1. Berikan permission minimum pada Staff.
2. Berikan permission `.edit`, `.delete`, `.refund`, dan `.writeoff` hanya setelah memeriksa dampak route aktual di tabel di atas.
3. Gunakan status Draft sebagai titik serah-terima review jika modul memilikinya.
4. Catat persetujuan pada SOP atau media kontrol internal sampai workflow formal tersedia.
5. Uji setiap kombinasi role memakai user non-admin sebelum production.

Ini bukan workflow approval formal dengan notifikasi. Jika production membutuhkan approval formal, perlu phase desain dan implementasi khusus untuk approval request, approve/reject, approval history, notification, lock action sebelum approved, dan approval berdasarkan batas nominal.

## 16. Checklist Production Untuk Role dan Akses

1. Pastikan minimal satu Super Admin aktif dan kredensialnya dikelola secara ketat.
2. Siapkan minimal satu Admin cadangan.
3. Jangan memakai Super Admin untuk transaksi harian.
4. Buat role kustom sebelum mengundang user operasional.
5. Pisahkan role sales, purchases, inventory, banking, accounting, reports, dan master data sesuai kebutuhan organisasi.
6. Berikan refund, write-off, dan delete hanya kepada supervisor yang membutuhkan.
7. Uji login setiap role dengan user non-admin.
8. Uji akses URL langsung untuk memastikan halaman terlarang menghasilkan 403.
9. Uji tombol aksi create, edit, delete, refund, write-off, open, publish, deliver, initiate, dan apply yang relevan.
10. Uji role view-only pada laporan granular.
11. Dokumentasikan pemilik dan tujuan setiap role.
12. Review role setiap bulan atau kuartal.
13. Nonaktifkan user yang keluar atau tidak lagi memerlukan akses.
14. Jangan berbagi akun login.
15. Aktifkan backup dan monitoring log di production.

## 17. Checklist Setelah Semua Preferensi Diisi

- General tersimpan: RS UMMI, IDR, January fiscal year, Indonesian language bila demo memakai bahasa Indonesia, `DD/MM/yyyy`, `Asia/Jakarta`.
- Logo, warna aksen, dan footer tampil pada PDF contoh.
- Currencies dan kurs siap untuk skenario multi-currency.
- COA minimal tersedia untuk AR, AP, Tax Payable, Cash/Bank, Income, Expense, COGS, dan Inventory.
- Cabang Utama dan Gudang Utama tersedia bila demo memakai dimensi lokasi atau stok.
- Accountant preference memiliki basis Akrual dan akun default payment yang sesuai AccountType.
- Item Preferences, Item Categories, dan Items master siap untuk sales dan purchases.
- Default teks EST/INV/REC/CN tersimpan dan muncul pada dokumen baru.
- Role kustom dibuat berdasarkan template, user diundang, dan akses diuji memakai user non-admin.
- Refund, write-off, dan delete hanya diberikan kepada supervisor yang membutuhkan.
- `setting.edit` tetap admin-only dan tidak diberikan kepada role operasional biasa.
- Strategi approval sederhana berbasis status, permission, dan SOP sudah disepakati.
- Transaction Locking hanya aktif untuk periode yang memang sudah siap dikunci.

## 18. Checklist Presentasi/Demo

1. Mulai dari General dan tunjukkan identitas RS UMMI, NPWP dummy, IDR, Asia/Jakarta, dan branding PDF.
2. Buka Currencies dan jelaskan base currency serta exchange rate.
3. Buka Cabang dan Gudang, tampilkan `Cabang Utama` dan `Gudang Utama`.
4. Buka Accountant, jelaskan basis Akrual, aturan kode akun, dan akun Bank BCA sebagai akun default.
5. Buka Items preference, Item Categories, dan Items master; jelaskan service, inventory, non-inventory, sellable, dan purchasable.
6. Buka Estimates, Invoices, Receipts, dan Credit Notes; jelaskan default notes/terms serta contoh nomor EST/INV/PAY/CN sebagai konvensi nomor dokumen.
7. Buka Users tab Users; undang user demo Finance.
8. Buka Users tab Roles; tunjukkan matrix permission, role predefined, dan role kustom.
9. Login atau impersonasi manual sesuai prosedur demo untuk membuktikan sidebar berubah mengikuti role.
10. Tunjukkan satu role non-admin dengan menu terbatas dan uji satu URL langsung yang menghasilkan 403.
11. Jelaskan bahwa approval sederhana memakai status, permission, dan SOP; jangan klaim workflow approval formal sudah tersedia.
12. Tutup dengan checklist transaksi kecil: buat invoice baru, cek default teks, deliver, download PDF, lalu record payment bila akun default siap.

## 19. Catatan Field Yang Belum Terverifikasi

| Catatan | Status audit | Cara membawakan saat presentasi |
| --- | --- | --- |
| `number_prefix`, `next_number`, dan `auto_increment` | Ditemukan di `SettingSeeder` untuk sales estimates, sales invoices, sales receipts, payment receives, credit notes, dan dokumen lain. Field edit untuk nilai tersebut tidak muncul pada halaman Preferensi yang diaudit. | Boleh disebut sebagai konvensi nomor default seperti `EST-`, `INV-`, `PAY-`, dan `CN-`, tetapi jangan instruksikan admin mengeditnya dari halaman Preferensi. |
| `organization_email` | Dibaca pada shared settings/report tertentu, tetapi tidak ada dalam `UpdateSettingRequest::GENERAL_ORGANIZATION_KEYS` dan tidak terlihat sebagai input General pada audit ini. | Jangan jadikan field wajib setup General pada demo ini. |
| Permission route bernama `permissions` | `php artisan route:list --name=permissions` tidak menemukan route. Permission dikelola melalui role form, bukan halaman permissions terpisah. | Arahkan admin ke Preferences > Users > Roles. |
| Cash-basis report toggle | Basis `cash` ada pada enum dan Accountant preference, tetapi Balance Sheet, Trial Balance, dan Tax Summary tidak mengekspos toggle basis cash. | Jelaskan sebagai kebijakan produk saat ini; gunakan basis Akrual untuk demo utama. |
| Workflow approval formal multi-user | Belum terverifikasi dari kode pada phase ini. Sistem saat ini memakai status dokumen dan permission sebagai kontrol operasional sederhana. | Jangan klaim notifikasi approval, approval history, lock sebelum approved, atau threshold nominal sudah tersedia. Jika dibutuhkan, siapkan phase desain dan implementasi terpisah. |
