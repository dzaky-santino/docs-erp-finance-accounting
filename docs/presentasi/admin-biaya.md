# Presentasi Admin Biaya

## 1. Tujuan Dokumen

Dokumen ini menjadi panduan presentasi untuk super admin atau admin saat menjelaskan menu Biaya. Fokusnya adalah alur yang sudah terverifikasi dari kode: daftar biaya, tambah biaya, detail biaya, edit biaya draft, publish biaya, hapus biaya draft, bulk delete biaya draft, dan lampiran pada detail biaya.

Dokumen ini juga membedakan biaya langsung dari tagihan vendor. Biaya dipakai untuk pengeluaran yang sudah dibayar dari akun kas, bank, atau aset lancar lain. Tagihan vendor tetap memakai modul Bills karena Bills membentuk utang usaha sebelum dibayar.

Rujukan setup yang berkaitan:

- Preferensi sistem: [admin-preferensi.md](admin-preferensi.md)
- Bagan akun, jurnal, dan penguncian transaksi: [admin-keuangan.md](admin-keuangan.md)
- Akun bank dan review transaksi bank: [admin-perbankan.md](admin-perbankan.md)
- Kontak vendor/customer bila biaya perlu atribusi pihak eksternal: [admin-kontak.md](admin-kontak.md)
- Proyek untuk atribusi biaya: [admin-proyek.md](admin-proyek.md)
- Laporan biaya, GL, dan Project Profitability: [admin-laporan.md](admin-laporan.md)

## 2. Gambaran Umum Menu Biaya

Menu Biaya yang terverifikasi berada pada group sidebar `Expenses`.

| Area | Route aktual | Fungsi | Permission utama | Status audit |
| --- | --- | --- | --- | --- |
| Daftar Biaya | `/accounting/expenses` | Melihat, mencari, memilih, dan menghapus draft biaya. | `expense.view`, `expense.delete` | Terverifikasi. |
| Biaya Baru | `/accounting/expenses/create` | Membuat draft biaya atau langsung publish. | `expense.create` | Terverifikasi. |
| Detail Biaya | `/accounting/expenses/{id}` | Melihat ringkasan, baris biaya, jurnal saat published, dan lampiran. | `expense.view` | Terverifikasi. |
| Edit Biaya | `/accounting/expenses/{id}/edit` | Mengubah biaya berstatus draft. | `expense.edit` | Terverifikasi. |
| Publish Biaya | `POST /api/expenses/{expense}/publish` | Mengubah draft menjadi published dan membuat jurnal. | `expense.edit` | Terverifikasi. |
| Bulk Delete Draft | `DELETE /api/expenses/bulk` | Menghapus beberapa biaya draft. | `expense.delete` | Terverifikasi. |
| Lampiran Biaya | Panel lampiran di detail biaya | Upload, download, dan hapus bukti pengeluaran. | `expense.view`, `expense.edit` | Terverifikasi melalui attachment generik. |

Status biaya:

| Status | Dampak akuntansi | Aksi yang tampak di UI |
| --- | --- | --- |
| Draft | Belum membuat jurnal umum. | Bisa diedit, dipublish, dan dihapus. |
| Published | Membuat jurnal: debit akun biaya, kredit akun pembayaran. | Detail menampilkan dampak jurnal. Edit dan delete tidak ditampilkan di index/detail. |

Prasyarat dari route create:

| Prasyarat | Sumber opsi | Jika kosong |
| --- | --- | --- |
| Akun pembayaran aktif | Account type `Cash`, `Bank`, atau `OtherCurrentAsset`. | User diarahkan ke Bagan Akun. |
| Akun biaya aktif | Account type `Expense`, `OtherExpense`, atau `CostOfGoodsSold`. | User diarahkan ke Bagan Akun. |
| Vendor sebagai payee | Contact dengan service vendor. | Field payee tetap opsional. |
| Project | Master project aktif. | Field project tetap opsional. |

## 3. Urutan Penggunaan Yang Disarankan

1. Pastikan preferensi organisasi, mata uang dasar, format tanggal, dan tahun fiskal sudah siap.
2. Buat akun pembayaran di Bagan Akun, minimal satu akun Bank atau Cash aktif.
3. Buat akun biaya di Bagan Akun, misalnya Beban ATK, Beban Transportasi, Beban Administrasi Bank, atau Harga Pokok Penjualan jika biaya berhubungan dengan HPP.
4. Buat kontak vendor jika biaya ingin dikaitkan dengan pihak penerima pembayaran.
5. Buat project jika biaya perlu dianalisis per project.
6. Input biaya sebagai draft untuk review internal, atau langsung publish jika bukti dan akun sudah benar.
7. Buka detail biaya untuk memeriksa jurnal yang terbentuk setelah publish.
8. Upload bukti pengeluaran pada panel lampiran di detail biaya.
9. Cek dampaknya di Buku Besar, Laporan Laba Rugi, Arus Kas, dan Profitabilitas Proyek bila memakai project.

## 4. Sub Menu Biaya

### Kegunaan

Sub menu Biaya dipakai untuk mencatat pengeluaran yang sudah dibayar. Contoh penggunaan yang cocok:

- Pembelian ATK yang langsung dibayar dengan kas kecil.
- Biaya administrasi bank.
- Biaya transportasi atau parkir operasional.
- Biaya listrik, air, internet, atau langganan yang langsung dibayar.
- Biaya yang akan menjadi landed cost bila ditandai sebagai landed cost.

Gunakan Bills, bukan Biaya, bila transaksi masih berupa tagihan vendor yang belum dibayar dan harus membentuk utang usaha.

### Daftar Input

| Field | Wajib | Validasi/form aktual | Dampak |
| --- | --- | --- | --- |
| Tanggal Pembayaran | Ya | `payment_date` wajib date. | Menjadi tanggal jurnal saat biaya dipublish. |
| Akun Pembayaran | Ya | `payment_account_id` wajib, opsi UI dari akun aktif type Cash, Bank, atau Other Current Asset. | Dikredit sebesar total biaya. |
| Penerima Pembayaran | Tidak | `payee_id` nullable, opsi UI dari kontak vendor. | Menjadi kontak pada baris jurnal akun pembayaran. |
| Project Header | Tidak | `project_id` nullable. | Disimpan pada dokumen biaya untuk konteks header. |
| Nomor Referensi | Tidak | `reference_no` nullable maksimal 50 karakter. | Membantu pencarian dan rekonsiliasi manual. |
| Catatan | Tidak | `description` nullable maksimal 2000 karakter. | Menjadi catatan dokumen. |
| Baris Biaya | Ya | `categories` wajib array minimal 1, maksimal 100. | Menentukan rincian debit biaya. |
| Akun Biaya Baris | Ya | `categories.*.expense_account_id` wajib, opsi UI dari Expense, Other Expense, atau Cost Of Goods Sold. | Didebit saat publish. |
| Deskripsi Baris | Tidak | `categories.*.description` nullable maksimal 1000 karakter. | Menjadi deskripsi baris dan catatan jurnal debit. |
| Project Baris | Tidak | `categories.*.project_id` nullable. | Menjadi project pada baris jurnal debit dan laporan project. |
| Jumlah Baris | Ya | `categories.*.amount` wajib numeric dan lebih besar dari 0. | Menambah total biaya. |
| Landed Cost | Tidak | `categories.*.is_landed_cost` boolean. | Menandai baris sebagai sumber landed cost. |

Field backend yang ada tetapi tidak terlihat sebagai input utama pada halaman React yang diaudit:

| Field backend | Catatan presentasi |
| --- | --- |
| `currency_code` | Ada di request dan model, tetapi halaman create/edit yang diaudit tidak menampilkan input mata uang. |
| `exchange_rate` | Ada di request dan model, tetapi halaman create/edit yang diaudit tidak menampilkan input kurs. |
| `branch_id` | Ada di request dan model, tetapi halaman create/edit yang diaudit tidak menampilkan input cabang. |

### Pengaruh Ke Modul Lain

| Modul/laporan | Pengaruh saat biaya dipublish |
| --- | --- |
| Buku Besar | Membuat transaksi dengan reference type `Expense`: debit akun biaya, kredit akun pembayaran. |
| Bagan Akun | Saldo akun biaya naik di sisi debit; saldo akun pembayaran berkurang melalui kredit. |
| Laporan Laba Rugi | Baris biaya masuk ke kelompok expense jika akun biaya berada pada root type biaya. |
| Laporan Arus Kas | Expense diperlakukan sebagai salah satu sumber transaksi operasional untuk akun Cash/Bank. |
| Profitabilitas Proyek | Baris biaya yang memiliki project masuk ke analisis project. |
| Lampiran Dokumen | Bukti pembayaran bisa diupload dari detail biaya. |
| Landed Cost | Baris bertanda landed cost menyimpan nilai landed cost dan dapat terkait proses alokasi biaya persediaan. |
| Review Bank | Kode review bank mengenali reference type `Expense`, sehingga biaya dapat menjadi referensi transaksi bank yang relevan. |

Catatan penting untuk demo: tidak ada field pajak pada halaman Biaya yang diaudit. Jika ingin menunjukkan pajak masukan/keluaran, gunakan alur pembelian, penjualan, atau jurnal yang memang membentuk akun Tax Payable sesuai setup.

### Contoh Input Demo

| Skenario | Input contoh | Hasil yang ditunjukkan |
| --- | --- | --- |
| Biaya ATK langsung bayar | Tanggal hari demo, akun pembayaran `Bank BCA Operasional`, payee `PT Sumber Medika`, referensi `EXP-ATK-001`, akun biaya `Beban ATK`, jumlah `250000`. | Draft dapat dipublish; detail menampilkan debit Beban ATK dan kredit Bank BCA. |
| Biaya admin bank | Akun pembayaran `Bank BCA Operasional`, tanpa payee, referensi `ADM-BANK-001`, akun biaya `Beban Administrasi Bank`, jumlah `6500`. | Cocok untuk menjelaskan biaya kecil yang langsung mengurangi bank. |
| Biaya transport project | Payee `CV Kurir Nusantara`, project `Implementasi ERP Cabang Bandung`, akun biaya `Beban Transportasi`, jumlah `175000`. | Biaya dapat dibaca pada Profitabilitas Proyek bila project dipilih pada baris. |
| Biaya landed cost | Akun biaya `Biaya Pengiriman Import`, jumlah `1500000`, centang landed cost. | Menjelaskan bahwa baris dapat ditandai sebagai landed cost untuk alur persediaan terkait. |

Langkah demo singkat:

1. Buka `/accounting/expenses/create`.
2. Isi tanggal, akun pembayaran, payee opsional, referensi, dan catatan.
3. Tambahkan minimal satu baris biaya.
4. Simpan sebagai draft.
5. Buka detail dan jelaskan status draft belum membentuk jurnal.
6. Klik publish.
7. Tunjukkan ringkasan jurnal pada detail biaya.
8. Upload bukti pembayaran pada panel lampiran.

### Error Umum

| Kondisi | Penyebab umum | Cara menjelaskan ke peserta demo |
| --- | --- | --- |
| Tidak bisa membuka form biaya baru | Tidak ada akun pembayaran aktif atau tidak ada akun biaya aktif. | Setup Bagan Akun dulu. Sistem mengarahkan ke menu akun. |
| Validasi gagal pada baris biaya | Tidak ada baris, akun biaya kosong, atau jumlah kurang dari/sama dengan 0. | Minimal satu baris biaya wajib dan nominal harus positif. |
| Publish gagal karena periode terkunci | Tanggal pembayaran masuk periode transaksi yang dikunci. | Ubah tanggal atau buka penguncian transaksi sesuai otorisasi. |
| Publish gagal karena sudah published | Dokumen sudah pernah dipublish. | Status published tidak perlu dipublish ulang. |
| Hapus gagal pada biaya landed cost yang sudah dialokasikan | Biaya sudah terkait alokasi landed cost. | Batalkan atau koreksi alokasi terkait sesuai prosedur. |
| Upload lampiran gagal | File terlalu besar atau ekstensi tidak didukung. | Gunakan file maksimal 10 MB dengan ekstensi `pdf`, `jpg`, `jpeg`, `png`, `webp`, `doc`, `docx`, `xls`, `xlsx`, `csv`, atau `txt`. |
| Akun yang dipilih tidak sesuai tujuan | UI normal memfilter opsi akun, tetapi request backend yang diaudit memvalidasi keberadaan akun. | Jangan bypass UI; gunakan akun pembayaran dan akun biaya yang benar. |

### Checklist

- Akun pembayaran aktif sudah ada.
- Akun biaya aktif sudah ada.
- Vendor opsional sudah dibuat bila ingin menampilkan payee.
- Project opsional sudah dibuat bila ingin demo Profitabilitas Proyek.
- Nomor referensi mudah dikenali untuk demo.
- Minimal satu baris biaya terisi.
- Nominal baris lebih besar dari 0.
- Status draft ditunjukkan sebelum publish.
- Jurnal pada detail ditunjukkan setelah publish.
- Bukti pengeluaran berhasil diupload pada detail biaya.

## 5. Sub Menu Kategori Biaya / Sub Menu Lain Jika Ada

Kode yang diaudit tidak menunjukkan route atau halaman master data Kategori Biaya yang berdiri sendiri. Tabel dan model `expense_categories` digunakan sebagai baris rincian pada dokumen Biaya, bukan sebagai master kategori yang dikelola dari menu terpisah.

Ringkasan fungsi `expense_categories`:

| Field baris | Fungsi |
| --- | --- |
| `expense_account_id` | Akun biaya yang akan didebit saat publish. |
| `index` | Urutan baris pada dokumen. |
| `description` | Deskripsi baris biaya. |
| `amount` | Nilai biaya baris. |
| `allocated_cost_amount` | Nilai landed cost yang sudah dialokasikan. |
| `is_landed_cost` | Menandai baris sebagai landed cost. |
| `expense_id` | Relasi ke dokumen biaya. |
| `project_id` | Project pada baris biaya. |

Sub menu lain seperti reimbursement, claim, recurring expense, mileage, atau expense payment terpisah tidak muncul dari route yang diaudit. Untuk presentasi, jelaskan bahwa scope menu Biaya saat ini adalah pencatatan biaya langsung yang sudah dibayar.

## 6. Contoh Data Awal Untuk Presentasi

Siapkan data berikut sebelum demo:

| Jenis data | Contoh | Tujuan |
| --- | --- | --- |
| Akun pembayaran | `1002 Bank BCA Operasional` dengan type Bank. | Sumber dana untuk biaya. |
| Akun pembayaran kecil | `1001 Kas Kecil` dengan type Cash. | Demo pengeluaran tunai. |
| Akun biaya | `6101 Beban ATK`, `6102 Beban Administrasi Bank`, `6103 Beban Transportasi`, `6104 Beban Internet`. | Opsi baris biaya. |
| Akun HPP/biaya persediaan | `5101 Harga Pokok Penjualan` atau akun COGS lain. | Demo jika biaya perlu masuk kelompok COGS. |
| Vendor | `PT Sumber Medika`, `CV Kurir Nusantara`. | Payee pada transaksi biaya. |
| Project | `Implementasi ERP Cabang Bandung`. | Demo Profitabilitas Proyek. |
| File bukti | PDF atau gambar bukti pembayaran kecil. | Demo attachment. |

Contoh nomor referensi yang mudah dibaca:

| Skenario | Nomor referensi |
| --- | --- |
| ATK | `EXP-ATK-001` |
| Administrasi bank | `EXP-BANK-001` |
| Transport project | `EXP-TRP-001` |
| Landed cost | `EXP-LC-001` |

## 7. Contoh Alur Demo Biaya

Alur 1 - biaya draft lalu publish:

1. Buka daftar Biaya.
2. Klik Biaya Baru.
3. Isi tanggal pembayaran, akun pembayaran, payee, referensi, dan catatan.
4. Tambahkan baris `Beban ATK` sebesar `250000`.
5. Simpan sebagai draft.
6. Tunjukkan bahwa status masih Draft dan belum ada jurnal yang ditampilkan sebagai dampak published.
7. Publish biaya.
8. Buka detail dan jelaskan jurnal debit akun biaya serta kredit akun pembayaran.

Alur 2 - biaya project:

1. Buat biaya transport untuk project.
2. Pilih project pada baris biaya.
3. Publish.
4. Buka laporan Profitabilitas Proyek dan gunakan project yang sama.
5. Jelaskan bahwa biaya project berasal dari baris jurnal yang memiliki project.

Alur 3 - lampiran bukti:

1. Buka detail biaya yang sudah dibuat.
2. Upload bukti pembayaran.
3. Download kembali file dari panel lampiran.
4. Hapus lampiran bila ingin menunjukkan kontrol akses edit.

Alur 4 - perbedaan Biaya dan Bills:

1. Jelaskan Biaya: uang sudah keluar dari kas/bank.
2. Jelaskan Bills: tagihan vendor belum tentu dibayar dan membentuk utang usaha.
3. Gunakan Biaya untuk transaksi langsung bayar seperti ATK atau biaya bank.
4. Gunakan Bills untuk pembelian vendor dengan proses AP.

## 8. Checklist Setelah Setup Menu Biaya

- Daftar Biaya dapat dibuka oleh role yang memiliki `expense.view`.
- Biaya Baru dapat dibuka oleh role yang memiliki `expense.create`.
- Opsi akun pembayaran muncul dan berasal dari akun aktif yang sesuai.
- Opsi akun biaya muncul dan berasal dari akun aktif yang sesuai.
- Opsi payee hanya menampilkan kontak vendor.
- Opsi project muncul bila master project sudah dibuat.
- Simpan draft berhasil.
- Publish berhasil pada periode yang tidak terkunci.
- Detail published menampilkan dampak jurnal.
- Lampiran dapat diupload dan didownload.
- Hapus draft berhasil.
- Hapus published tidak ditawarkan dari UI.

## 9. Checklist Presentasi/Demo

- Browser sudah login sebagai admin dengan permission biaya.
- Data demo tidak menggunakan nama vendor atau nomor rekening produksi.
- Saldo akun pembayaran cukup mudah dipahami oleh peserta demo.
- Satu contoh draft dan satu contoh published sudah tersedia.
- File bukti pembayaran demo berukuran kecil dan berekstensi didukung.
- Tab laporan siap dibuka untuk menunjukkan dampak ke Buku Besar, Laba Rugi, Arus Kas, dan Profitabilitas Proyek.
- Penjelasan perbedaan Biaya dan Bills disiapkan sebelum sesi tanya jawab.
- Peserta diberi tahu bahwa transaksi published memengaruhi laporan akuntansi.

## 10. Catatan Field/Menu Yang Belum Terverifikasi

- Master data Kategori Biaya sebagai submenu terpisah: Belum terverifikasi dari kode pada phase ini. Kode yang ada menunjukkan `expense_categories` sebagai baris rincian pada dokumen Biaya.
- Sub menu reimbursement, claim, recurring expense, mileage, atau expense payment terpisah: Belum terverifikasi dari kode pada phase ini.
- Field pajak atau tax rate pada halaman Biaya: Belum terverifikasi dari kode pada phase ini.
- Input mata uang, kurs, dan cabang pada halaman React Biaya: Belum terverifikasi dari kode pada phase ini, meskipun field backend `currency_code`, `exchange_rate`, dan `branch_id` ada pada request/model.
- Guard backend khusus agar akun pembayaran hanya Cash/Bank/Other Current Asset dan akun biaya hanya Expense/Other Expense/COGS: Belum terverifikasi dari kode pada phase ini. Route create/edit memfilter opsi UI, sedangkan Form Request yang diaudit memvalidasi `exists`.
- Approval workflow biaya: Belum terverifikasi dari kode pada phase ini.
- Screenshot final untuk materi presentasi: Belum terverifikasi dari kode pada phase ini.
