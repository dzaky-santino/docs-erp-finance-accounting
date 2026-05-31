# Presentasi Admin Penjualan

Dokumen ini adalah panduan presentasi dan pengisian awal menu **Penjualan** untuk role admin/superadmin. Isinya disusun dari audit sidebar, route Laravel, halaman Inertia React, Form Request, service transaksi, model status, attachment, dan laporan yang tersedia pada kode saat ini.

Rujukan silang:

- Preferensi dokumen dan nomor otomatis: [admin-preferensi.md](admin-preferensi.md)
- Pelanggan: [admin-kontak.md](admin-kontak.md)
- Barang, jasa, dan stok awal: [admin-barang-jasa.md](admin-barang-jasa.md)
- Akun piutang, kas, bank, pendapatan, dan pajak: [admin-keuangan.md](admin-keuangan.md)
- Akun bank dan rekonsiliasi: [admin-perbankan.md](admin-perbankan.md)
- Proyek dan profitability: [admin-proyek.md](admin-proyek.md)
- Laporan: [admin-laporan.md](admin-laporan.md)

## 1. Tujuan Dokumen

Panduan ini membantu admin/superadmin:

1. memahami perbedaan dokumen Penjualan;
2. menyiapkan pelanggan, barang/jasa, akun, pajak, proyek, dan preferensi dokumen sebelum transaksi;
3. memilih dokumen yang tepat untuk penawaran, tagihan, pembayaran faktur, penjualan langsung lunas, dan koreksi;
4. menjalankan demo end-to-end dengan data yang konsisten;
5. memahami pengaruh transaksi ke piutang, kas/bank, pajak, proyek, dan laporan;
6. membedakan perilaku yang sudah terverifikasi dari area yang masih perlu diuji manual.

Dokumen ini tidak menggantikan SOP accounting perusahaan. Admin tetap perlu memastikan akun, periode transaksi, pajak, dan hak akses sudah sesuai kebijakan organisasi.

## 2. Gambaran Umum Menu Penjualan

Sidebar **Penjualan** menampilkan lima sub menu utama:

| Sub menu sidebar | Istilah dokumen | Kegunaan utama | Route halaman |
| --- | --- | --- | --- |
| Estimates | Estimasi | Membuat penawaran sebelum menjadi tagihan. | `/finance/estimates` |
| Invoices | Faktur | Mencatat tagihan resmi kepada pelanggan. | `/finance/invoices` |
| Receipts | Penerimaan Penjualan / Sale Receipt | Mencatat penjualan yang langsung lunas. | `/finance/sale-receipts` |
| Credit Notes | Nota Kredit | Mengurangi tagihan atau mencatat kredit pelanggan. | `/finance/credit-notes` |
| Payments Received | Penerimaan Pembayaran / Payment Receive | Mencatat pembayaran pelanggan atas faktur. | `/finance/payment-receives` |

Sidebar juga menyediakan shortcut pembuatan dokumen jika user memiliki permission create:

| Shortcut | Route | Permission |
| --- | --- | --- |
| New Estimate | `/finance/estimates/create` | `sale-estimate.create` |
| New Invoice | `/finance/invoices/create` | `sale-invoice.create` |
| New Receipt | `/finance/sale-receipts/create` | `sale-receipt.create` |
| New Credit Note | `/finance/credit-notes/create` | `credit-note.create` |
| New Payment Received | `/finance/payment-receives/create` | `payment-receive.create` |

## 3. Urutan Alur Penjualan Yang Disarankan

Urutan setup:

1. Isi Preferensi dokumen Penjualan agar prefix dan nomor berikutnya sesuai kebutuhan.
2. Buat kontak pelanggan, misalnya `RS UMMI`.
3. Buat akun piutang usaha, kas, bank, pendapatan, dan akun pajak yang diperlukan.
4. Buat tarif pajak, misalnya `PPN 11%`.
5. Buat item `Barang A` dan `Jasa Konsultasi`.
6. Jika memakai item inventory, isi stok awal melalui Penyesuaian Persediaan.
7. Jika transaksi perlu dianalisis per pekerjaan, buat proyek terlebih dahulu.

Urutan transaksi dengan penawaran:

1. Buat Estimasi berstatus Draft.
2. Deliver Estimasi.
3. Approve Estimasi.
4. Convert Estimasi yang Approved menjadi Faktur Draft.
5. Periksa Faktur lalu Deliver.
6. Catat Penerimaan Pembayaran ketika pelanggan membayar.
7. Gunakan Nota Kredit jika ada retur, diskon setelah faktur, atau koreksi.
8. Apply Nota Kredit ke Faktur atau Refund sisa kredit kepada pelanggan.

Urutan transaksi langsung lunas:

1. Buat Penerimaan Penjualan.
2. Pilih akun Kas atau Bank.
3. Isi barang/jasa dan pajak.
4. Simpan Draft lalu Close.

## 4. Perbedaan Estimasi, Faktur, Penerimaan Penjualan, Penerimaan Pembayaran, dan Nota Kredit

| Dokumen | Dipakai kapan | Pengaruh utama | Bukan untuk |
| --- | --- | --- | --- |
| Estimasi | Saat menawarkan barang/jasa sebelum pelanggan setuju. | Menyimpan penawaran. Belum menjadi piutang dan belum membuat jurnal GL. | Mencatat pembayaran pelanggan. |
| Faktur | Saat perusahaan menagih pelanggan. | Setelah Deliver, menambah piutang. Baris pendapatan dan pajak mengikuti data transaksi yang dibawa saat posting. | Penjualan tunai yang ingin dicatat langsung lunas. |
| Penerimaan Pembayaran | Saat pelanggan membayar satu atau beberapa faktur. | Menambah kas/bank dan mengurangi piutang. | Menambah item penjualan baru. |
| Penerimaan Penjualan | Saat penjualan langsung dibayar pada waktu yang sama. | Setelah Close, menambah kas/bank dan mencatat pendapatan serta pajak sesuai data posting. Tidak melalui piutang. | Menagih pelanggan untuk dibayar kemudian. |
| Nota Kredit | Saat mengoreksi tagihan, retur, atau memberikan kredit kepada pelanggan. | Setelah Open, mengurangi piutang. Kredit dapat di-apply ke faktur atau di-refund. | Mencatat pembayaran faktur biasa. |

Prinsip praktis:

- Jika masih berupa penawaran, gunakan **Estimasi**.
- Jika pelanggan berutang, gunakan **Faktur** lalu **Penerimaan Pembayaran**.
- Jika pelanggan langsung membayar, gunakan **Penerimaan Penjualan**.
- Jika nilai tagihan harus dikurangi setelah transaksi, gunakan **Nota Kredit**.

## 5. Sub Menu/Area Penjualan Dalam Sistem

| Area | Halaman aktual | Form Request / aksi | Service utama | Permission utama | Status audit |
| --- | --- | --- | --- | --- | --- |
| Estimasi | list, create, edit, show, PDF | store, update, deliver, approve, reject, convert, mail, delete | `SaleEstimateService` | `sale-estimate.view/create/edit/delete` | Terverifikasi |
| Faktur | list, create, edit, show, print, PDF | store, update, deliver, write-off, duplicate, mail, delete | `SaleInvoiceService` | `sale-invoice.view/create/edit/delete/writeoff` | Terverifikasi |
| Penerimaan Pembayaran | list, create, edit, show, PDF | store, update, mail, delete | `PaymentReceiveService` | `payment-receive.view/create/edit/delete` | Terverifikasi |
| Penerimaan Penjualan | list, create, edit, show, PDF | store, update, close, mail, delete | `SaleReceiptService` | `sale-receipt.view/create/edit/delete` | Terverifikasi |
| Nota Kredit | list, create, edit, show | store, update, open, apply, refund, delete refund, delete | `CreditNoteService` | `credit-note.view/create/edit/delete/refund` | Terverifikasi |
| Apply Nota Kredit | Form pada detail Nota Kredit | `applications` berisi faktur dan amount | `CreditNoteService` | `credit-note.edit` | Terverifikasi |
| Refund Nota Kredit | Form pada detail Nota Kredit | akun sumber, amount, tanggal, reference, description | `CreditNoteService` | `credit-note.refund` | Terverifikasi |
| Lampiran | Panel pada halaman detail | upload, download, detach | `DocumentService` | view/edit sesuai dokumen | Terverifikasi |

Catatan route:

- Probe `php artisan route:list --name=customers` tidak menemukan route bernama `customers`.
- Master pelanggan memakai area Kontak dengan route `/settings/contacts`.
- Probe `php artisan route:list --path=sales` terutama menemukan route report `sales-by-items`.
- Halaman transaksi Penjualan aktual berada di prefix `/finance/...`.

## 6. Estimasi

### Kegunaan

Estimasi adalah penawaran kepada pelanggan. Dokumen ini cocok dipakai sebelum ada komitmen penagihan. Estimasi tidak menambah piutang, tidak menambah kas/bank, dan tidak membuat jurnal GL.

Jika pelanggan menerima penawaran, Estimasi harus berada pada status **Approved** sebelum dapat dikonversi menjadi Faktur Draft.

### Daftar Input

| Field UI | Wajib/Opsional | Validasi atau perilaku aktual | Fungsi awam | Contoh |
| --- | --- | --- | --- | --- |
| Customer | Wajib | Kontak customer harus dipilih. | Pelanggan penerima penawaran. | `RS UMMI` |
| Project | Opsional | Proyek aktif dapat dipilih. | Menghubungkan penawaran ke pekerjaan tertentu. | `PRJ-ERP-UMMI` |
| Estimate Number | Opsional | Maksimal 50 karakter dan unik untuk data aktif. | Nomor dokumen penawaran. | `EST-0001` |
| Reference | Opsional | Maksimal 50 karakter. | Referensi eksternal. | `RFQ-UMMI-01` |
| Estimate Date | Wajib | Tanggal penawaran harus diisi. | Tanggal dokumen dibuat. | `2026-01-10` |
| Expiration Date | Opsional | Tidak boleh sebelum tanggal estimasi. | Batas masa berlaku penawaran. | `2026-01-24` |
| Currency | Opsional | Mengacu ke mata uang terdaftar. | Mata uang transaksi. | `IDR` |
| Exchange Rate | Kondisional | Harus lebih besar dari nol saat digunakan. | Kurs transaksi jika bukan mata uang dasar. | `1` |
| Item | Opsional per baris | Item sellable dapat dipilih; deskripsi tetap dapat diisi. | Barang/jasa yang ditawarkan. | `Barang A` |
| Description | Opsional per baris | Maksimal 1.000 karakter. | Penjelasan barang/jasa. | `Barang A untuk instalasi` |
| Quantity | Wajib per baris | Harus lebih besar dari nol. | Jumlah yang ditawarkan. | `2` |
| Rate | Wajib per baris | Tidak boleh negatif. | Harga jual per unit. | `1.000.000` |
| Discount | Opsional per baris | Diskon baris tersedia dengan tipe percentage atau amount. | Potongan harga per baris. | `5%` |
| Tax | Opsional per baris | Tarif pajak dapat dipilih. | Pajak penjualan. | `PPN 11%` |
| Note | Opsional | Maksimal 2.000 karakter. | Catatan untuk pelanggan. | `Harga termasuk instalasi dasar.` |
| Terms | Opsional | Maksimal 2.000 karakter. | Syarat penawaran. | `Berlaku 14 hari.` |

Form harus memiliki minimal satu baris item entry. Saat item dipilih, tabel baris mengisi deskripsi, harga, dan pajak dari master item jika tersedia. Untuk item inventory, UI menampilkan stok saat ini dan sisa stok setelah draft; kekurangan stok tampil sebagai warning.

### Status dan Aksi

| Status | Makna | Aksi utama |
| --- | --- | --- |
| Draft | Penawaran masih disiapkan. | Edit, Deliver, hapus. |
| Delivered | Penawaran sudah dikirimkan/disampaikan. | Approve atau Reject, kirim email, PDF. |
| Approved | Pelanggan menyetujui penawaran. | Convert menjadi Faktur. |
| Rejected | Pelanggan menolak penawaran. | Simpan sebagai histori. |
| Converted | Estimasi sudah dibuatkan Faktur. | Buka dokumen hasil konversi. |

Convert hanya valid untuk Estimasi **Approved** dan menghasilkan Faktur Draft. Saat convert, tanggal Faktur memakai tanggal saat proses dilakukan dan due date awal dibuat 30 hari setelahnya.

### Pengaruh Ke Modul Lain

| Area | Pengaruh |
| --- | --- |
| Kontak | Estimasi terhubung ke customer. |
| Barang/Jasa | Baris memakai item sellable dan default harga/pajak item. |
| Stok | UI menampilkan preview stok inventory dan sisa setelah draft; Estimasi tidak membuat mutasi stok. |
| Pajak | Pajak dihitung untuk nilai penawaran, tetapi belum menjadi posting pajak GL. |
| Proyek | Project dapat dipilih dan disalin saat convert ke Faktur. |
| Piutang dan kas | Tidak berubah. |

### Contoh Input

Contoh penawaran:

| Field | Nilai |
| --- | --- |
| Customer | `RS UMMI` |
| Project | `PRJ-ERP-UMMI` |
| Estimate Number | `EST-0001` |
| Estimate Date | `2026-01-10` |
| Expiration Date | `2026-01-24` |
| Item | `Barang A` |
| Quantity | `2` |
| Rate | `1.000.000` |
| Tax | `PPN 11%` |
| Terms | `Berlaku 14 hari.` |

Nomor otomatis default dari service mengikuti format `EST-00001`. Contoh `EST-0001` dapat dipakai sebagai nomor demo manual karena field nomor dapat diedit.

### Error Umum

| Error | Penyebab umum | Cara menghindari |
| --- | --- | --- |
| Customer wajib diisi | Pelanggan belum dipilih. | Buat dan pilih kontak customer. |
| Item entries wajib ada | Semua baris dihapus. | Sisakan minimal satu baris transaksi. |
| Expiration date tidak valid | Batas berlaku lebih awal dari tanggal estimasi. | Isi tanggal yang sama atau setelah estimate date. |
| Convert ditolak | Status belum Approved atau sudah Converted. | Jalankan Deliver lalu Approve sebelum Convert. |
| Email gagal dikirim | Customer tidak memiliki email atau dokumen masih Draft. | Lengkapi email customer dan Deliver terlebih dahulu. |

### Checklist

- [ ] Customer sudah dibuat.
- [ ] Item sellable sudah tersedia.
- [ ] Pajak item sudah diperiksa.
- [ ] Tanggal dan masa berlaku konsisten.
- [ ] Stock hint inventory sudah diperiksa.
- [ ] Estimasi sudah Deliver dan Approve sebelum Convert.

## 7. Faktur

### Kegunaan

Faktur adalah tagihan resmi kepada pelanggan. Faktur Draft belum diposting. Setelah **Deliver**, service membuat transaksi GL piutang usaha dan dokumen masuk ke status tagihan berdasarkan saldo, pembayaran, kredit, serta due date.

### Daftar Input

| Field UI | Wajib/Opsional | Validasi atau perilaku aktual | Fungsi awam | Contoh |
| --- | --- | --- | --- | --- |
| Customer | Wajib | Kontak customer harus dipilih. | Pihak yang ditagih. | `RS UMMI` |
| Project | Opsional | Proyek aktif dapat dipilih. | Analisis transaksi per proyek. | `PRJ-ERP-UMMI` |
| Invoice Number | Opsional | Maksimal 50 karakter dan unik untuk data aktif. | Nomor faktur. | `INV-0001` |
| Reference Number | Opsional | Maksimal 50 karakter. | Nomor referensi pelanggan atau internal. | `PO-UMMI-01` |
| Invoice Date | Wajib | Tanggal faktur harus diisi. | Tanggal tagihan. | `2026-01-15` |
| Due Date | Wajib | Tidak boleh sebelum invoice date. | Tanggal jatuh tempo. | `2026-02-14` |
| Currency | Opsional | Mengacu ke mata uang terdaftar. | Mata uang faktur. | `IDR` |
| Exchange Rate | Kondisional | Harus lebih besar dari nol saat digunakan. | Kurs jika bukan mata uang dasar. | `1` |
| Item, Description, Quantity, Rate | Minimal satu baris | Quantity lebih besar dari nol; rate tidak negatif. | Rincian penjualan. | `Barang A`, qty `2`, rate `1.000.000` |
| Discount dan Tax | Opsional per baris | Diskon percentage/amount; tarif pajak dapat dipilih. | Potongan dan pajak per item. | `PPN 11%` |
| Invoice Message | Opsional | Maksimal 2.000 karakter. | Pesan pada faktur. | `Terima kasih atas kerja sama Anda.` |
| Terms | Opsional | Maksimal 2.000 karakter. | Syarat pembayaran. | `Pembayaran maksimal 30 hari.` |

### Status dan Aksi

| Status | Makna | Aksi utama |
| --- | --- | --- |
| Draft | Faktur belum diposting. | Edit, Deliver, hapus. |
| Unpaid | Faktur sudah Deliver dan belum dibayar. | Record Payment, kirim email, PDF, print, write-off. |
| Overdue | Faktur belum lunas dan melewati due date. | Record Payment, tindak lanjuti pelanggan. |
| Partially Paid | Sebagian nilai sudah dibayar atau dikurangi kredit. | Record Payment untuk sisa saldo. |
| Paid | Saldo faktur sudah nol. | Lihat histori pembayaran/kredit. |
| Written Off | Saldo dihapuskan melalui aksi write-off. | Lihat atau cancel write-off sesuai kondisi. |

Setelah Deliver, model status biasanya langsung menampilkan `Unpaid`, bukan berhenti pada label `Delivered`, jika saldo masih terbuka. Halaman detail juga mendukung duplicate faktur menjadi Draft baru.

### Pengaruh Ke Modul Lain

| Area | Pengaruh setelah Deliver |
| --- | --- |
| Piutang usaha | Debit Accounts Receivable sebesar total faktur. |
| Pendapatan | Credit pendapatan per baris hanya dibuat jika `sell_account_id` tersedia pada entry saat posting. |
| Pajak | Credit akun Tax Payable dibuat jika transaksi memiliki pajak. |
| Proyek | Project header dibawa ke transaksi GL. |
| Pembayaran | Faktur terbuka dapat dipilih pada Penerimaan Pembayaran. |
| Nota Kredit | Faktur terbuka dapat dikurangi melalui Apply Nota Kredit. |
| Laporan | Memengaruhi Receivables Aging, Customer Balance Summary, General Ledger, Income Statement, Tax Summary, dan laporan terkait sesuai filter. |

Penting untuk demo: UI baris Penjualan yang diaudit tidak menampilkan atau mengisi otomatis `sell_account_id`. Karena posting pendapatan bergantung pada field tersebut, uji manual hasil GL pendapatan sebelum presentasi.

### Contoh Input

Contoh Faktur hasil convert atau input manual:

| Field | Nilai |
| --- | --- |
| Customer | `RS UMMI` |
| Project | `PRJ-ERP-UMMI` |
| Invoice Number | `INV-0001` |
| Invoice Date | `2026-01-15` |
| Due Date | `2026-02-14` |
| Item | `Barang A` |
| Quantity | `2` |
| Rate | `1.000.000` |
| Tax | `PPN 11%` |
| Total contoh | `2.220.000` |

Nomor otomatis default dari service mengikuti format `INV-000001`. Contoh `INV-0001` dapat dipakai sebagai nomor demo manual.

### Error Umum

| Error | Penyebab umum | Cara menghindari |
| --- | --- | --- |
| Due date tidak valid | Due date lebih awal dari invoice date. | Isi due date pada atau setelah invoice date. |
| Tidak dapat Deliver | Akun Accounts Receivable atau akun pajak yang diperlukan belum ada. | Siapkan Chart of Accounts terlebih dahulu. |
| Periode terkunci | Tanggal faktur masuk periode Sales yang dikunci. | Gunakan tanggal yang diizinkan atau koordinasikan pembukaan lock. |
| Pendapatan GL tidak muncul | Entry tidak membawa `sell_account_id`. | Uji hasil jurnal sebelum demo dan catat gap UI pada bagian 18. |
| Stok kurang | UI inventory menampilkan warning sisa stok. | Periksa stok awal dan stock hint sebelum menyimpan. |
| Record Payment tidak sesuai | Faktur belum Deliver atau sudah lunas. | Buka Faktur yang masih memiliki outstanding balance. |

### Checklist

- [ ] Accounts Receivable tersedia.
- [ ] Akun pajak tersedia jika memakai PPN.
- [ ] Customer, tanggal, due date, item, harga, dan pajak sudah benar.
- [ ] Project dipilih jika diperlukan.
- [ ] Stock hint inventory diperiksa.
- [ ] Hasil jurnal pendapatan diuji manual sebelum demo.
- [ ] Faktur Deliver sebelum mencatat pembayaran.

## 8. Penerimaan Pembayaran

### Kegunaan

Penerimaan Pembayaran dipakai saat pelanggan membayar Faktur. Satu dokumen dapat mengalokasikan pembayaran kepada satu atau beberapa faktur customer yang sama. Pembayaran dapat penuh atau parsial selama tidak melebihi outstanding balance.

Service mencatat debit ke akun deposit dan credit ke Accounts Receivable.

### Daftar Input

| Field UI | Wajib/Opsional | Validasi atau perilaku aktual | Fungsi awam | Contoh |
| --- | --- | --- | --- | --- |
| Customer | Wajib | Customer harus dipilih; dikunci saat edit. | Pelanggan yang membayar. | `RS UMMI` |
| Payment Number | Opsional | Nomor dokumen penerimaan pembayaran. | Referensi bukti penerimaan. | `PAY-0001` |
| Payment Date | Wajib | Tanggal pembayaran harus diisi. | Tanggal uang diterima. | `2026-01-20` |
| Deposit Account | Wajib | Service menerima akun bertipe Cash, Bank, atau Other Current Asset. | Tujuan dana masuk. | `Bank BCA` |
| Reference Number | Opsional | Maksimal 50 karakter. | Referensi transfer atau mutasi bank. | `TRX-BCA-200126` |
| Currency | Opsional | Mata uang transaksi. | Mata uang pembayaran. | `IDR` |
| Statement | Opsional | Maksimal 2.000 karakter. | Catatan penerimaan. | `Pelunasan INV-0001.` |
| Invoice allocation | Minimal satu faktur | Faktur harus milik customer terkait, sudah Deliver, belum write-off, dan masih memiliki outstanding. | Menentukan faktur yang dilunasi. | `INV-0001` |
| Payment Amount per invoice | Wajib untuk faktur terpilih | Lebih besar dari nol dan tidak boleh melebihi sisa faktur. | Nilai pembayaran yang dialokasikan. | `2.220.000` |

Jumlah header harus sama dengan total alokasi invoice. UI menghitung total dari alokasi faktur yang dipilih.

### Status dan Aksi

Penerimaan Pembayaran tidak memakai lifecycle Draft lalu Publish. Dokumen tersimpan langsung sebagai penerimaan yang memengaruhi GL dan saldo faktur. Halaman detail menyediakan PDF, email, attachment, edit, dan delete sesuai permission serta guard transaksi.

Dokumen tidak dapat diedit atau dihapus jika sudah matched melalui workflow banking. Delete juga ditolak jika periode Sales terkait terkunci.

### Pengaruh Ke Modul Lain

| Area | Pengaruh |
| --- | --- |
| Faktur | Menambah `payment_amount` dan mengurangi outstanding balance faktur. |
| Piutang | Credit Accounts Receivable. |
| Kas/Bank | Debit akun deposit yang dipilih. |
| Proyek | Halaman dapat merangkum proyek dari alokasi faktur; transaksi GL payment tidak memiliki input project langsung. |
| Perbankan | Dokumen dapat terhubung ke workflow match banking. |
| Laporan | Memengaruhi piutang, General Ledger, Cash Flow, laporan cash-basis yang didukung, dan transaksi kontak. |

### Contoh Input

| Field | Nilai |
| --- | --- |
| Customer | `RS UMMI` |
| Payment Number | `PAY-0001` |
| Payment Date | `2026-01-20` |
| Deposit Account | `Bank BCA` |
| Reference Number | `TRX-BCA-200126` |
| Invoice | `INV-0001` |
| Amount | `2.220.000` |
| Statement | `Pelunasan INV-0001.` |

Contoh parsial: alokasikan `1.000.000` terlebih dahulu, lalu buat Penerimaan Pembayaran berikutnya untuk sisa Faktur.

### Error Umum

| Error | Penyebab umum | Cara menghindari |
| --- | --- | --- |
| Tidak ada faktur yang dapat dipilih | Faktur belum Deliver, sudah lunas, atau customer tidak sesuai. | Deliver Faktur dan pilih customer yang benar. |
| Overpayment | Amount melebihi outstanding Faktur. | Gunakan nilai maksimum sesuai saldo tersisa. |
| Total tidak sama | Header amount berbeda dari total alokasi. | Periksa jumlah alokasi terpilih. |
| Deposit account salah | Akun bukan tipe yang diperbolehkan. | Pilih Kas, Bank, atau Other Current Asset aktif. |
| Edit/delete ditolak | Pembayaran sudah matched atau periode terkunci. | Selesaikan workflow banking atau koordinasikan lock sesuai SOP. |

### Checklist

- [ ] Faktur sudah Deliver dan masih outstanding.
- [ ] Customer pembayaran sesuai customer Faktur.
- [ ] Deposit Account aktif dan benar.
- [ ] Tanggal pembayaran sesuai mutasi aktual.
- [ ] Total alokasi sama dengan jumlah uang diterima.
- [ ] Tidak ada overpayment.

## 9. Penerimaan Penjualan

### Kegunaan

Penerimaan Penjualan atau Sale Receipt dipakai untuk penjualan langsung lunas. Dokumen ini menggabungkan pencatatan penjualan dan penerimaan uang tanpa membuat piutang customer terlebih dahulu.

### Daftar Input

| Field UI | Wajib/Opsional | Validasi atau perilaku aktual | Fungsi awam | Contoh |
| --- | --- | --- | --- | --- |
| Customer | Wajib | Kontak customer harus dipilih. | Pembeli. | `RS UMMI` |
| Project | Opsional | Proyek aktif dapat dipilih. | Analisis per proyek. | `PRJ-ERP-UMMI` |
| Deposit Account | Wajib | Halaman create menyediakan akun Cash atau Bank aktif. | Tempat uang diterima. | `Kas` |
| Receipt Date | Wajib | Tanggal transaksi harus diisi. | Tanggal penjualan tunai. | `2026-01-22` |
| Receipt Number | Opsional | Maksimal 50 karakter dan unik untuk data aktif. | Nomor bukti transaksi. | `SR-0001` |
| Reference Number | Opsional | Maksimal 50 karakter. | Referensi eksternal. | `POS-220126-01` |
| Currency | Opsional | Mengacu ke mata uang terdaftar. | Mata uang transaksi. | `IDR` |
| Exchange Rate | Kondisional | Harus lebih besar dari nol saat digunakan. | Kurs jika bukan mata uang dasar. | `1` |
| Item, Description, Quantity, Rate | Minimal satu baris | Quantity lebih besar dari nol; rate tidak negatif. | Rincian penjualan langsung. | `Jasa Konsultasi`, qty `1`, rate `500.000` |
| Discount dan Tax | Opsional per baris | Diskon dan tarif pajak tersedia per baris. | Potongan dan pajak. | `PPN 11%` |
| Receipt Message | Opsional | Maksimal 2.000 karakter. | Pesan pada dokumen. | `Pembayaran diterima tunai.` |
| Statement | Opsional | Maksimal 2.000 karakter. | Catatan transaksi. | `Penjualan langsung lunas.` |

### Status dan Aksi

| Status | Makna | Aksi utama |
| --- | --- | --- |
| Draft | Transaksi masih dapat diedit. | Edit, Close, hapus. |
| Closed | Transaksi sudah diposting. | PDF, email, attachment, lihat detail. |

Saat Close, service mencatat debit ke akun deposit, credit pendapatan per entry jika `sell_account_id` tersedia, dan credit Tax Payable jika ada pajak.

### Pengaruh Ke Modul Lain

| Area | Pengaruh setelah Close |
| --- | --- |
| Kas/Bank | Debit akun deposit yang dipilih. |
| Pendapatan | Credit pendapatan mengikuti `sell_account_id` entry. |
| Pajak | Credit Tax Payable jika ada pajak. |
| Piutang | Tidak memakai Accounts Receivable. |
| Proyek | Project header dibawa ke transaksi GL. |
| Laporan | Memengaruhi General Ledger, Income Statement, Cash Flow, Tax Summary, dan laporan proyek sesuai filter. |

Seperti pada Faktur, uji manual hasil jurnal pendapatan sebelum demo karena UI baris yang diaudit tidak menampilkan atau mengisi otomatis `sell_account_id`.

### Contoh Input

| Field | Nilai |
| --- | --- |
| Customer | `RS UMMI` |
| Receipt Number | `SR-0001` |
| Receipt Date | `2026-01-22` |
| Deposit Account | `Kas` |
| Item | `Jasa Konsultasi` |
| Quantity | `1` |
| Rate | `500.000` |
| Statement | `Penjualan langsung lunas.` |

Nomor otomatis default dari service mengikuti format `RCPT-000001`. Contoh `SR-0001` dapat dipakai sebagai nomor demo manual.

### Error Umum

| Error | Penyebab umum | Cara menghindari |
| --- | --- | --- |
| Deposit Account kosong | Akun penerimaan belum dipilih. | Pilih Kas atau Bank aktif. |
| Item entries kosong | Tidak ada rincian penjualan. | Isi minimal satu baris. |
| Tidak dapat Close | Akun pajak yang diperlukan belum tersedia atau periode Sales terkunci. | Siapkan akun dan periksa Transaction Locking. |
| Pendapatan GL tidak muncul | Entry tidak membawa `sell_account_id`. | Uji jurnal sebelum demo. |
| Stok kurang | Stock hint inventory memberi warning. | Periksa stok awal dan sisa draft. |

### Checklist

- [ ] Customer tersedia.
- [ ] Akun Kas atau Bank aktif tersedia.
- [ ] Item dan pajak diperiksa.
- [ ] Stock hint inventory diperiksa jika relevan.
- [ ] Hasil jurnal pendapatan diuji manual.
- [ ] Dokumen di-Close setelah data benar.

## 10. Nota Kredit

### Kegunaan

Nota Kredit dipakai untuk mengurangi tagihan customer, mencatat retur, memberi diskon setelah faktur, atau mengoreksi harga. Dokumen dibuat sebagai Draft lalu di-Open sebelum dapat digunakan.

### Daftar Input

| Field UI | Wajib/Opsional | Validasi atau perilaku aktual | Fungsi awam | Contoh |
| --- | --- | --- | --- | --- |
| Customer | Wajib | Kontak customer harus dipilih. | Pemilik kredit. | `RS UMMI` |
| Project | Opsional | Proyek aktif dapat dipilih. | Analisis koreksi per proyek. | `PRJ-ERP-UMMI` |
| Credit Note Number | Opsional | Maksimal 50 karakter dan unik untuk data aktif. | Nomor Nota Kredit. | `CN-0001` |
| Reference Number | Opsional | Maksimal 50 karakter. | Referensi retur atau koreksi. | `RET-UMMI-01` |
| Credit Note Date | Wajib | Tanggal dokumen harus diisi. | Tanggal koreksi. | `2026-01-25` |
| Currency | Opsional | Mata uang transaksi. | Mata uang Nota Kredit. | `IDR` |
| Exchange Rate | Kondisional | Harus lebih besar dari nol saat digunakan. | Kurs jika bukan mata uang dasar. | `1` |
| Item, Description, Quantity, Rate | Minimal satu baris | Quantity lebih besar dari nol; rate tidak negatif. | Rincian retur atau koreksi. | `Barang A`, qty `1`, rate `1.000.000` |
| Discount dan Tax | Opsional per baris | Diskon dan tarif pajak tersedia. | Koreksi nilai dan pajak. | `PPN 11%` |
| Note | Opsional | Maksimal 2.000 karakter. | Isi alasan retur/koreksi. | `Retur 1 Barang A karena koreksi pesanan.` |
| Terms | Opsional | Maksimal 2.000 karakter. | Catatan tambahan. | `Kredit diterapkan ke INV-0001.` |

Tidak ada field UI khusus bernama `Reason`. Gunakan Note, Reference Number, atau Description untuk menjelaskan alasan retur/koreksi.

### Status dan Aksi

| Status | Makna | Aksi utama |
| --- | --- | --- |
| Draft | Nota Kredit masih disiapkan. | Edit, Open, hapus. |
| Open | Kredit sudah diposting dan masih memiliki remaining credit. | Apply To Invoice, Refund, attachment, print browser. |
| Closed | Seluruh kredit sudah dipakai atau di-refund. | Lihat histori. |

Saat Open, service membuat credit Accounts Receivable, debit pendapatan per entry jika `sell_account_id` tersedia, dan debit Tax Payable jika ada pajak. Uji hasil jurnal pendapatan sebelum demo karena UI baris tidak menampilkan atau mengisi otomatis `sell_account_id`.

### Apply To Invoice

Apply To Invoice memakai kredit yang tersedia untuk mengurangi outstanding Faktur.

Alur UI:

1. Buka Nota Kredit dengan status Open.
2. Pilih **Apply To Invoice**.
3. Pilih Faktur yang tampil.
4. Isi amount yang tidak melebihi remaining credit dan outstanding Faktur.
5. Simpan aplikasi kredit.

UI menawarkan Faktur customer yang sama, sudah Deliver, belum write-off, dan masih outstanding. Apply memperbarui nilai kredit terpakai serta credited amount Faktur tanpa membuat posting GL baru.

### Refund

Refund dipakai jika sisa kredit dikembalikan sebagai uang kepada customer.

| Field refund | Wajib/Opsional | Fungsi awam | Contoh |
| --- | --- | --- | --- |
| Deposit Account / akun sumber | Wajib | Akun asal pengembalian dana. UI menyediakan akun aktif Cash, Bank, atau Other Current Asset. | `Bank BCA` |
| Amount | Wajib | Nilai pengembalian; tidak boleh melebihi remaining credit. | `555.000` |
| Refund Date | Wajib | Tanggal uang dikembalikan. | `2026-01-26` |
| Reference Number | Opsional | Referensi transfer refund. | `RF-BCA-260126` |
| Description | Opsional | Catatan pengembalian dana. | `Refund sisa kredit CN-0001.` |

Refund mencatat debit Accounts Receivable dan credit akun sumber pengembalian dana. Refund yang salah dapat dihapus sesuai permission dan guard periode.

### Pengaruh Ke Modul Lain

| Area | Pengaruh |
| --- | --- |
| Piutang | Open Nota Kredit mengurangi AR; refund mengembalikan sebagian AR sebelum uang keluar. |
| Faktur | Apply mengurangi outstanding melalui credited amount. |
| Pendapatan dan pajak | Open membalik nilai pendapatan/pajak sesuai entry yang terbawa. |
| Kas/Bank | Refund mengurangi akun sumber yang dipilih. |
| Proyek | Project header dibawa ke GL Nota Kredit; laporan cash-basis yang didukung juga memiliki kontrak untuk refund/application. |
| Laporan | Memengaruhi General Ledger, Income Statement, Tax Summary, Customer Balance, dan laporan terkait sesuai sumber/filter. |

### Contoh Input

| Field | Nilai |
| --- | --- |
| Customer | `RS UMMI` |
| Credit Note Number | `CN-0001` |
| Credit Note Date | `2026-01-25` |
| Reference Number | `RET-UMMI-01` |
| Note | `Retur 1 Barang A karena koreksi pesanan.` |
| Item | `Barang A` |
| Quantity | `1` |
| Rate | `1.000.000` |
| Tax | `PPN 11%` |
| Apply To Invoice | `INV-0001` |
| Refund jika masih ada sisa | dari `Bank BCA` |

Nomor otomatis default dari service mengikuti format `CN-00001`. Contoh `CN-0001` dapat dipakai sebagai nomor demo manual.

### Error Umum

| Error | Penyebab umum | Cara menghindari |
| --- | --- | --- |
| Apply/Refund tidak tersedia | Nota Kredit masih Draft atau remaining credit sudah nol. | Open Nota Kredit dan periksa remaining credit. |
| Apply terlalu besar | Amount melebihi remaining credit atau outstanding Faktur. | Gunakan nilai maksimum yang tersedia. |
| Refund terlalu besar | Amount melebihi remaining credit. | Kurangi nilai refund. |
| Akun refund salah | Akun sumber belum dipilih atau tidak sesuai kebijakan. | Pilih akun aktif yang benar dari opsi UI. |
| Periode terkunci | Tanggal Open atau Refund masuk periode Sales terkunci. | Periksa Transaction Locking. |
| Pendapatan GL tidak lengkap | Entry tidak membawa `sell_account_id`. | Uji hasil jurnal sebelum demo. |

### Checklist

- [ ] Customer Nota Kredit sesuai customer Faktur.
- [ ] Alasan retur/koreksi ditulis pada Note, Reference, atau Description.
- [ ] Pajak koreksi diperiksa.
- [ ] Nota Kredit di-Open sebelum Apply atau Refund.
- [ ] Apply tidak melebihi outstanding Faktur.
- [ ] Refund tidak melebihi remaining credit.
- [ ] Akun refund sesuai sumber dana aktual.
- [ ] Hasil jurnal diuji manual sebelum demo.

## 11. PDF, Email, dan Dokumen Penjualan

| Dokumen | PDF | Email | Print | Attachment | Catatan |
| --- | --- | --- | --- | --- | --- |
| Estimasi | Tersedia | Tersedia setelah bukan Draft | Melalui preview PDF | Tersedia | Email dapat melampirkan PDF. |
| Faktur | Tersedia | Tersedia setelah bukan Draft | Route print tersedia | Tersedia | Email dapat melampirkan PDF. |
| Penerimaan Pembayaran | Tersedia | Tersedia | Melalui preview PDF | Tersedia | Cocok untuk bukti penerimaan dana. |
| Penerimaan Penjualan | Tersedia | Tersedia | Melalui preview PDF | Tersedia | Email dapat melampirkan PDF. |
| Nota Kredit | Tidak ada route PDF khusus yang terverifikasi | Tidak ada route email khusus yang terverifikasi | Print browser tersedia pada halaman detail | Tersedia | Jangan menjanjikan PDF/email khusus saat demo. |

Attachment dapat dipakai untuk bukti pendukung seperti PO, bukti transfer, atau dokumen retur. Policy upload yang diaudit menerima file `pdf`, `jpg`, `jpeg`, `png`, `webp`, `doc`, `docx`, `xls`, `xlsx`, `csv`, dan `txt` sampai 10 MB.

Untuk email dokumen, lengkapi email customer terlebih dahulu. Dokumen Draft yang mensyaratkan workflow posting perlu diproses sebelum email dikirim.

## 12. Pengaruh Penjualan Ke Stok, Pajak, Proyek, dan Laporan

| Area | Pengaruh Penjualan | Status audit |
| --- | --- | --- |
| Kontak/Pelanggan | Customer wajib dipilih pada dokumen Penjualan. | Terverifikasi |
| Barang/Jasa | Form memuat item sellable. Pemilihan item mengisi deskripsi, harga, dan pajak default jika tersedia. | Terverifikasi |
| Stok | Shared line-items UI menampilkan quantity on hand, sisa setelah draft, dan warning kekurangan untuk inventory. Service/non-inventory tidak dilacak stok. | Terverifikasi untuk preview UI |
| Pajak | Pajak entry memengaruhi posting Tax Payable pada dokumen yang diposting. Tax Summary mengagregasi Invoice, Sale Receipt, dan Credit Note. | Terverifikasi |
| Proyek | Estimate, Invoice, Sale Receipt, dan Credit Note memiliki pilihan project. Estimate menyalin project saat convert. | Terverifikasi |
| Piutang | Invoice Deliver mendebit AR; Payment Receive mengkredit AR; Credit Note Open mengkredit AR; refund Nota Kredit mendebit AR. | Terverifikasi |
| Kas/Bank | Payment Receive dan Sale Receipt mendebit akun penerimaan; refund Nota Kredit mengkredit akun sumber dana. | Terverifikasi |
| General Ledger | Dokumen posted membuat double-entry sesuai service. | Terverifikasi dengan catatan `sell_account_id` |
| Receivables Aging | Menampilkan Faktur Deliver yang masih outstanding berdasarkan jatuh tempo. | Terverifikasi |
| Customer Balance Summary | Merangkum Faktur, pembayaran, kredit, dan outstanding customer. | Terverifikasi |
| Transactions By Contact | Memuat sumber customer seperti Invoice, Payment Receive, dan Credit Note sesuai implementasi report. | Terverifikasi |
| Sales By Items | Mengambil item entry Faktur Deliver. | Terverifikasi |
| Cash Flow | Memuat sumber operasional termasuk Invoice, Payment Received, dan Sale Receipt sesuai implementasi report. | Terverifikasi |
| Income Statement | Memuat posting pendapatan dan koreksi sesuai transaksi GL serta basis report yang didukung. | Terverifikasi dengan catatan `sell_account_id` |
| Project Profitability | Memakai project attribution dari transaksi yang didukung. | Terverifikasi |

Stok perlu dipresentasikan dengan hati-hati. Preview stok pada form sudah terverifikasi, tetapi mutasi stok aktual dari dokumen Penjualan biasa belum dapat diklaim dari audit phase ini.

## 13. Contoh Data Awal Untuk Presentasi

Siapkan data berikut sebelum presentasi:

### Kontak

| Tipe | Nama | Catatan |
| --- | --- | --- |
| Customer | `RS UMMI` | Isi email jika demo email dokumen diperlukan. |

### Akun

| Akun | Tipe/fungsi | Dipakai untuk |
| --- | --- | --- |
| `Piutang Usaha` | Accounts Receivable | Deliver Faktur, Open Nota Kredit, Payment Receive, refund kredit |
| `Kas` | Cash | Penerimaan Penjualan langsung |
| `Bank BCA` | Bank | Penerimaan Pembayaran dan refund |
| `Pendapatan Penjualan` | Income | Posting pendapatan item |
| `PPN Keluaran` | Tax Payable | Pajak penjualan |

### Pajak

| Nama | Rate | Dipakai untuk |
| --- | --- | --- |
| `PPN 11%` | `11` | `Barang A` dan transaksi kena pajak |

### Barang dan Jasa

| Item | Tipe | Harga jual | Pajak | Catatan |
| --- | --- | --- | --- | --- |
| `Barang A` | Inventory | `1.000.000` | `PPN 11%` | Isi stok awal melalui Penyesuaian Persediaan. |
| `Jasa Konsultasi` | Service | `500.000` | Sesuaikan kebutuhan demo | Stok tidak dilacak. |

### Proyek Opsional

| Kode | Nama |
| --- | --- |
| `PRJ-ERP-UMMI` | `Implementasi ERP RS UMMI` |

### Nomor Dokumen Demo

| Dokumen | Nomor demo manual | Contoh nomor otomatis service |
| --- | --- | --- |
| Estimasi | `EST-0001` | `EST-00001` |
| Faktur | `INV-0001` | `INV-000001` |
| Penerimaan Pembayaran | `PAY-0001` | `PAY-00001` |
| Penerimaan Penjualan | `SR-0001` | `RCPT-000001` |
| Nota Kredit | `CN-0001` | `CN-00001` |

## 14. Contoh Alur Demo Penjualan End-to-End

### Skenario A - Penawaran Menjadi Faktur dan Dibayar

1. Buka **Penjualan > Estimates**.
2. Buat `EST-0001` tanggal `2026-01-10` untuk `RS UMMI`.
3. Tambahkan `Barang A`, quantity `2`, rate `1.000.000`, dan `PPN 11%`.
4. Simpan Draft, lalu Deliver.
5. Approve Estimasi.
6. Convert menjadi Faktur.
7. Buka Faktur hasil convert, ubah nomor demo menjadi `INV-0001`, invoice date `2026-01-15`, dan due date `2026-02-14` jika diperlukan.
8. Deliver Faktur.
9. Periksa saldo piutang dan jurnal GL. Verifikasi manual baris pendapatan.
10. Buka **Penjualan > Payments Received**.
11. Buat `PAY-0001` tanggal `2026-01-20`, pilih `Bank BCA`, lalu alokasikan `2.220.000` ke `INV-0001`.
12. Buka kembali Faktur dan tunjukkan bahwa saldo sudah lunas.

### Skenario B - Penjualan Langsung Lunas

1. Buka **Penjualan > Receipts**.
2. Buat `SR-0001` tanggal `2026-01-22` untuk `RS UMMI`.
3. Pilih akun deposit `Kas`.
4. Tambahkan `Jasa Konsultasi`, quantity `1`, rate `500.000`.
5. Simpan lalu Close.
6. Periksa jurnal GL dan tampilkan PDF atau email jika diperlukan.

### Skenario C - Retur atau Koreksi Dengan Nota Kredit

1. Buka **Penjualan > Credit Notes**.
2. Buat `CN-0001` tanggal `2026-01-25` untuk `RS UMMI`.
3. Isi Reference `RET-UMMI-01`.
4. Isi Note `Retur 1 Barang A karena koreksi pesanan.`
5. Tambahkan `Barang A`, quantity `1`, rate `1.000.000`, dan pajak yang sesuai.
6. Simpan lalu Open.
7. Gunakan **Apply To Invoice** untuk mengurangi `INV-0001` jika Faktur masih outstanding.
8. Jika masih ada remaining credit yang memang harus dikembalikan, buat Refund dari `Bank BCA`.
9. Tunjukkan remaining credit dan status Nota Kredit.

### Skenario D - Tinjau Laporan

Setelah transaksi, buka:

1. **Customer Balance Summary** untuk melihat ringkasan saldo customer.
2. **Receivables Aging** untuk melihat faktur outstanding berdasarkan umur piutang.
3. **Transactions By Contact** untuk transaksi customer.
4. **General Ledger** untuk jurnal akun piutang, kas/bank, pendapatan, dan pajak.
5. **Income Statement** untuk dampak pendapatan.
6. **Tax Summary** untuk ringkasan pajak terposting.
7. **Project Profitability** jika memakai `PRJ-ERP-UMMI`.

## 15. Error Umum dan Cara Menghindari

| Kondisi | Dampak | Cara menghindari |
| --- | --- | --- |
| Customer belum dibuat | Form transaksi tidak dapat dilengkapi. | Buat kontak dengan tipe customer terlebih dahulu. |
| Akun AR belum ada | Deliver Faktur atau Open Nota Kredit ditolak. | Siapkan Accounts Receivable. |
| Akun pajak belum ada | Posting transaksi berpajak dapat ditolak. | Siapkan Tax Payable sebelum demo PPN. |
| Akun Kas/Bank belum aktif | Payment Receive atau Sale Receipt tidak dapat memakai akun deposit. | Aktifkan akun yang sesuai. |
| Item tidak sellable | Item tidak tampil pada form Penjualan. | Periksa konfigurasi item. |
| Stock hint memberi warning | Quantity inventory melebihi sisa stok preview. | Isi stok awal dan periksa draft lain. |
| `sell_account_id` tidak terbawa | Baris pendapatan GL tidak terbentuk. | Uji jurnal manual sebelum presentasi. |
| Periode Sales terkunci | Posting, refund, edit, atau delete tertentu ditolak. | Periksa Transaction Locking. |
| Estimasi belum Approved | Convert ditolak. | Deliver lalu Approve. |
| Faktur belum Deliver | Tidak tampil untuk Payment Receive atau Apply kredit. | Deliver Faktur. |
| Pembayaran melebihi saldo | Payment Receive ditolak. | Gunakan outstanding aktual. |
| Kredit melebihi saldo | Apply atau Refund ditolak. | Gunakan remaining credit dan outstanding aktual. |
| Pembayaran sudah matched | Edit/delete Payment Receive ditolak. | Ikuti SOP banking dan rekonsiliasi. |
| Customer tidak punya email | Pengiriman email gagal. | Lengkapi email customer. |

## 16. Checklist Sebelum Input Penjualan

- [ ] Role admin/superadmin memiliki permission dokumen Penjualan yang diperlukan.
- [ ] Preferensi nomor dokumen Penjualan sudah diperiksa.
- [ ] Customer sudah tersedia di Kontak.
- [ ] Email customer sudah diisi jika demo email diperlukan.
- [ ] Accounts Receivable tersedia.
- [ ] Akun Kas dan Bank tersedia serta aktif.
- [ ] Akun pendapatan dan akun Tax Payable tersedia.
- [ ] Tarif `PPN 11%` tersedia jika diperlukan.
- [ ] Item `Barang A` dan `Jasa Konsultasi` sudah sellable.
- [ ] Stok awal `Barang A` sudah diisi jika demo inventory diperlukan.
- [ ] Project tersedia jika analisis proyek akan ditampilkan.
- [ ] Transaction Locking tidak memblokir tanggal demo.
- [ ] Posting pendapatan dari baris sales sudah diuji manual.

## 17. Checklist Presentasi/Demo

- [ ] Login sebagai admin/superadmin.
- [ ] Tunjukkan lima sub menu Penjualan di sidebar.
- [ ] Jelaskan perbedaan Estimasi, Faktur, Payment Receive, Sale Receipt, dan Nota Kredit.
- [ ] Tunjukkan customer `RS UMMI`.
- [ ] Tunjukkan item inventory dan jasa.
- [ ] Tunjukkan stock hint inventory pada form.
- [ ] Buat Estimasi lalu jalankan Deliver, Approve, dan Convert.
- [ ] Deliver Faktur dan tunjukkan status/outstanding.
- [ ] Catat Penerimaan Pembayaran melalui `Bank BCA`.
- [ ] Buat Penerimaan Penjualan langsung melalui `Kas`.
- [ ] Buat Nota Kredit, Apply To Invoice, dan Refund jika ada remaining credit.
- [ ] Tunjukkan PDF/email hanya untuk dokumen yang terverifikasi mendukungnya.
- [ ] Tunjukkan attachment.
- [ ] Buka laporan customer, piutang, GL, pendapatan, pajak, dan proyek yang relevan.
- [ ] Jelaskan area yang masih memerlukan verifikasi manual.

## 18. Catatan Field/Menu Yang Belum Terverifikasi

Area berikut harus diperlakukan sebagai batas audit, bukan sebagai klaim fitur:

1. **Belum terverifikasi dari kode pada phase ini:** mutasi stok aktual dari Invoice, Sale Receipt, atau Credit Note biasa. Yang terverifikasi adalah preview quantity on hand, sisa setelah draft, dan warning inventory pada UI.
2. **Belum terverifikasi dari kode pada phase ini:** input atau autofill `sell_account_id` pada shared line-items UI Penjualan. Service posting hanya membuat baris pendapatan jika field tersebut tersedia pada entry. Lakukan uji jurnal manual sebelum demo.
3. **Belum terverifikasi dari kode pada phase ini:** kontrol UI untuk header discount, discount type, adjustment, inclusive tax, warehouse, dan branch pada form Penjualan. Beberapa field tersedia pada request/backend atau state, tetapi tidak tampil pada form yang diaudit.
4. **Belum terverifikasi dari kode pada phase ini:** input UI `send_to_email` khusus pada form create Estimasi. Mail state tetap dapat memakai email customer.
5. **Belum terverifikasi dari kode pada phase ini:** input UI exchange rate pada form Penerimaan Pembayaran. Request/backend mendukung exchange rate, tetapi form yang diaudit tidak menampilkan kontrolnya.
6. **Belum terverifikasi dari kode pada phase ini:** route PDF dan email khusus Nota Kredit. Halaman detail menyediakan print browser dan attachment.
7. **Belum terverifikasi dari kode pada phase ini:** field khusus `Reason` untuk Nota Kredit. Gunakan Note, Reference Number, atau Description.
8. **Belum terverifikasi dari kode pada phase ini:** enforcement backend bahwa target Apply Nota Kredit selalu milik customer yang sama. UI resmi memfilter Faktur customer yang sama; gunakan UI resmi dan uji skenario sebelum demo integrasi.
9. **Belum terverifikasi dari kode pada phase ini:** filter customer aktif pada seluruh dropdown Penjualan. Audit route helper memastikan kontak bertipe customer, tetapi tidak menemukan filter `is_active`.
10. **Belum terverifikasi dari kode pada phase ini:** cakupan mutasi stok retur dari Nota Kredit.
11. **Belum terverifikasi dari kode pada phase ini:** Sales By Items memasukkan Sale Receipt. Audit saat ini menunjukkan sumber report tersebut adalah item entry dari Faktur Deliver.
12. **Belum terverifikasi dari kode pada phase ini:** Transactions By Contact memasukkan Estimasi atau Sale Receipt. Audit saat ini menunjukkan sumber customer seperti Invoice, Payment Receive, dan Credit Note.
13. **Belum terverifikasi dari kode pada phase ini:** screenshot final untuk materi presentasi. Dokumen ini adalah panduan Markdown, bukan deck atau hasil browser smoke test visual.

Sebelum demo eksternal, jalankan satu transaksi contoh di environment demo dan cocokkan jurnal GL, saldo piutang, saldo kas/bank, pajak, project attribution, serta laporan yang akan ditampilkan.
