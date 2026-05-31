# Presentasi Admin Barang/Jasa

## 1. Tujuan Dokumen

Dokumen ini adalah bahan presentasi untuk admin dan superadmin saat menjelaskan cara menyiapkan dan memakai menu Barang/Jasa. Fokusnya adalah cara orang finance/accounting memahami item sebagai master data transaksi, bukan sebagai pembahasan teknis kode.

Barang/Jasa dipakai untuk:

- Menentukan item yang bisa muncul di estimasi, faktur, sale receipt, credit note, bill, dan vendor credit.
- Menyimpan harga jual, harga beli/cost, pajak default, dan akun default item.
- Memisahkan item jasa, barang persediaan, dan barang non-inventory.
- Menjadi sumber daftar item untuk Penyesuaian Persediaan dan Transfer Gudang jika item bertipe persediaan.
- Menjadi dasar beberapa laporan item dan inventory.

Dokumen ini hanya mendokumentasikan fitur yang diaudit dari kode pada phase ini. Field atau perilaku yang belum dapat dipastikan ditaruh di bagian catatan dengan status eksplisit.

## 2. Gambaran Umum Menu Barang/Jasa

Di sidebar, area utama Barang/Jasa berada pada grup `Items` atau `Items & Inventory`. Menu yang terverifikasi:

| Menu | Route halaman | Page React | Fungsi awam | Permission utama |
| --- | --- | --- | --- | --- |
| Barang/Jasa | `/settings/items` | `resources/js/pages/settings/items.tsx` | Membuat master item yang bisa dijual/dibeli. | `item.view`, `item.create`, `item.edit`, `item.delete` |
| Penyesuaian Persediaan | `/accounting/inventory-adjustments` | `resources/js/pages/accounting/inventory-adjustments/index.tsx` | Menambah atau mengurangi stok karena stok awal, koreksi, atau stock opname. | `inventory-adjustment.view`, `inventory-adjustment.create`, `inventory-adjustment.edit`, `inventory-adjustment.delete` |
| Kategori | `/settings/item-categories` | `resources/js/pages/settings/item-categories.tsx` | Mengelompokkan item dan menyimpan akun default kategori. | `item.view`, `item.create`, `item.edit`, `item.delete` |
| Transfer Gudang | `/accounting/warehouse-transfers` | `resources/js/pages/accounting/warehouse-transfers/index.tsx` | Memindahkan stok item inventory antar gudang. | `warehouse-transfer.view`, `warehouse-transfer.create`, `warehouse-transfer.edit`, `warehouse-transfer.delete` |
| Preferensi Item | `/settings/item-preferences` | `resources/js/pages/settings/item-preferences.tsx` | Menyimpan preferensi akun default item. | `setting.edit`, admin-like |
| Gudang | `/settings/warehouses` | `resources/js/pages/settings/location-preferences.tsx` | Master lokasi stok untuk inventory adjustment dan warehouse transfer. | `setting.edit`, admin-like |

Route pendukung yang terverifikasi:

| Area | Endpoint web/API | Status audit |
| --- | --- | --- |
| Barang/Jasa | `GET /settings/items`, `POST /api/items`, `PUT /api/items/{id}`, `DELETE /api/items/{id}`, `DELETE /api/items` | Tersedia |
| Kategori | `GET /settings/item-categories`, `POST /api/item-categories`, `PUT /api/item-categories/{id}`, `DELETE /api/item-categories/{id}`, `DELETE /api/item-categories` | Tersedia |
| Preferensi Item | `GET /settings/item-preferences`, `PUT /api/settings/item-preferences` | Tersedia |
| Penyesuaian Persediaan | `GET /accounting/inventory-adjustments`, `GET /accounting/inventory-adjustments/create`, `GET /accounting/inventory-adjustments/{id}`, `POST /api/inventory-adjustments`, `POST /api/inventory-adjustments/{id}/publish`, `DELETE /api/inventory-adjustments/{id}` | Tersedia |
| Transfer Gudang | `GET /accounting/warehouse-transfers`, `GET /accounting/warehouse-transfers/create`, `GET /accounting/warehouse-transfers/{id}`, `GET /accounting/warehouse-transfers/{id}/edit`, API create/update/initiate/deliver/delete | Tersedia |
| Laporan item/inventory | `/reports/sales-by-items`, `/reports/purchases-by-items`, `/reports/inventory-valuation-sheet`, `/reports/inventory-item-details` | Tersedia |

## 3. Perbedaan Jasa, Persediaan, dan Non-Inventory

| Tipe item | Backing value kode | Stok dilacak? | Masuk Penyesuaian Persediaan? | Masuk Transfer Gudang? | Dipakai di sales? | Dipakai di purchase? | Contoh |
| --- | --- | ---: | ---: | ---: | ---: | ---: | --- |
| Jasa/Service | `service` | Tidak | Tidak | Tidak | Ya, jika `is_sellable` aktif | Ya, jika `is_purchasable` aktif | Jasa Konsultasi |
| Persediaan/Inventory | `inventory` | Ya | Ya | Ya | Ya, jika `is_sellable` aktif | Ya, jika `is_purchasable` aktif | Barang A |
| Non-Inventory | `non-inventory` | Tidak | Tidak | Tidak | Ya, jika `is_sellable` aktif | Ya, jika `is_purchasable` aktif | ATK Sekali Pakai |

Penjelasan awam:

- Jasa adalah layanan yang tidak punya stok fisik. Contoh: konsultasi, instalasi, maintenance.
- Persediaan adalah barang fisik yang stoknya perlu dipantau. Jika user ingin stok terlihat di transaksi dan laporan inventory, item harus bertipe `inventory`.
- Non-inventory adalah barang yang bisa dijual atau dibeli, tetapi stok fisiknya tidak dilacak. Contoh: perlengkapan habis pakai yang tidak perlu saldo gudang.
- Penyesuaian Persediaan dan Transfer Gudang hanya menerima item bertipe `inventory` dari request dan route create.

## 4. Kapan Barang/Jasa Perlu Dibuat

Buat Barang/Jasa sebelum:

- Menjual barang/jasa di Estimate, Invoice, Sale Receipt, atau Credit Note.
- Membeli barang/jasa di Bill atau Vendor Credit.
- Mencatat stok awal atau koreksi stok lewat Penyesuaian Persediaan.
- Memindahkan stok antar gudang lewat Transfer Gudang.
- Menampilkan ringkasan penjualan/pembelian per item dan laporan inventory.

Tidak semua transaksi biaya membutuhkan item. Modul Expense yang diaudit memakai kategori/akun biaya (`ExpenseCategory`) dan bukan master Barang/Jasa.

## 5. Urutan Penggunaan Yang Disarankan

Urutan demo atau implementasi yang paling aman:

1. Siapkan Bagan Akun: akun pendapatan, akun biaya/HPP, akun persediaan, akun penyesuaian, kas/bank, piutang, utang, dan pajak sesuai kebutuhan transaksi.
2. Siapkan Tarif Pajak jika item perlu default PPN/PPh.
3. Siapkan Gudang minimal satu untuk Penyesuaian Persediaan, dan minimal dua untuk Transfer Gudang.
4. Isi Preferensi Item jika ingin menyimpan akun default di level preferensi.
5. Buat Kategori Barang/Jasa untuk pengelompokan dan akun default kategori.
6. Buat item Jasa, Persediaan, dan Non-Inventory sesuai kebutuhan.
7. Untuk item persediaan, input stok awal lewat Penyesuaian Persediaan lalu publish.
8. Buat transaksi sales/purchase untuk menunjukkan harga, deskripsi, pajak, dan stock hint.
9. Jika ada dua gudang, buat Transfer Gudang.
10. Buka laporan Sales by Items, Purchases by Items, Inventory Valuation Sheet, dan Inventory Item Details.

Rujukan silang:

- Setup akun dan pajak: [admin-keuangan.md](admin-keuangan.md)
- Preferensi dan gudang: [admin-preferensi.md](admin-preferensi.md)
- Penggunaan item sellable di dokumen penjualan: [admin-penjualan.md](admin-penjualan.md)
- Penggunaan item purchasable di dokumen pembelian: [admin-pembelian.md](admin-pembelian.md)
- Biaya/Expense: [admin-biaya.md](admin-biaya.md)
- Laporan: [admin-laporan.md](admin-laporan.md)

## 6. Sub Menu/Area Barang/Jasa Dalam Sistem

| Area Barang/Jasa | Route/page aktual | Request/Form | Service | Permission | Status audit |
| --- | --- | --- | --- | --- | --- |
| Barang/Jasa | `/settings/items`, `settings/items.tsx` | `StoreItemRequest`, `UpdateItemRequest` | `ItemService` | `item.view/create/edit/delete` | Tersedia |
| Kategori Barang/Jasa | `/settings/item-categories`, `settings/item-categories.tsx` | `StoreItemCategoryRequest`, `UpdateItemCategoryRequest` | `ItemCategoryService` | `item.view/create/edit/delete` | Tersedia |
| Preferensi Item | `/settings/item-preferences`, `settings/item-preferences.tsx` | `UpdateItemPreferenceRequest` | `PreferenceController::updateItems` | `setting.edit`, admin-like | Tersedia |
| Gudang | `/settings/warehouses`, `settings/location-preferences.tsx` | `StoreWarehouseRequest`, `UpdateWarehouseRequest` | `WarehouseService` | `setting.edit`, admin-like | Tersedia sebagai dependensi stok |
| Penyesuaian Persediaan | `/accounting/inventory-adjustments`, create/show/index pages | `StoreInventoryAdjustmentRequest` | `InventoryAdjustmentService` | `inventory-adjustment.view/create/edit/delete` | Tersedia |
| Transfer Gudang | `/accounting/warehouse-transfers`, create/show/index pages | `StoreWarehouseTransferRequest`, `UpdateWarehouseTransferRequest` | `WarehouseTransferService` | `warehouse-transfer.view/create/edit/delete` | Tersedia |
| Laporan Item/Inventory | `/reports/sales-by-items`, `/reports/purchases-by-items`, `/reports/inventory-valuation-sheet`, `/reports/inventory-item-details` | Route query date/item | `ReportService`, `ReportExportService` | Permission granular report | Tersedia |

Delete guard yang terverifikasi:

| Area | Guard hapus |
| --- | --- |
| Barang/Jasa | Item tidak bisa dihapus jika sudah direferensikan di `item_entries`; bulk delete akan skip item yang sudah dipakai. |
| Kategori Barang/Jasa | Saat kategori dihapus, `category_id` pada item yang memakai kategori itu diset menjadi `null`; item tidak ikut terhapus. |
| Penyesuaian Persediaan | Penyesuaian yang sudah `published_at` tidak bisa dihapus. |
| Transfer Gudang | Transfer yang sudah `transfer_initiated_at` tidak bisa diedit atau dihapus. |

## 7. Daftar Input Barang/Jasa

### Kegunaan Setiap Field

| Field | Fungsi awam | Berpengaruh ke |
| --- | --- | --- |
| Nama | Nama item yang tampil di daftar dan transaksi. | Dropdown item, dokumen, laporan item |
| Tipe | Menentukan apakah item adalah jasa, inventory, atau non-inventory. | Stok, inventory adjustment, transfer gudang |
| Kode | Identitas item untuk pencarian dan pembeda item. | Daftar item, dropdown, laporan |
| Kategori | Kelompok item agar mudah dikelola. | Pengelompokan master data |
| Dapat dijual | Menentukan item masuk dropdown transaksi penjualan. | Estimate, Invoice, Sale Receipt, Credit Note |
| Harga jual | Harga default saat item dipilih di transaksi penjualan. | Sales line item |
| Akun pendapatan | Akun pendapatan untuk sisi penjualan jika terbawa ke line item/GL. | Posting sales |
| Pajak penjualan | Pajak default saat item dipilih di transaksi penjualan. | Sales line item |
| Deskripsi penjualan | Teks default di baris sales. | Dokumen sales |
| Dapat dibeli | Menentukan item masuk dropdown transaksi pembelian. | Bill, Vendor Credit |
| Harga beli/cost | Cost default saat item dipilih di transaksi pembelian dan inventory form. | Purchase line item, inventory adjustment, transfer gudang |
| Akun biaya/HPP | Akun biaya atau HPP untuk sisi pembelian jika terbawa ke line item/GL. | Posting purchase |
| Pajak pembelian | Pajak default saat item dipilih di transaksi pembelian. | Purchase line item |
| Deskripsi pembelian | Teks default di baris purchase. | Dokumen purchase |
| Akun persediaan | Akun persediaan untuk item inventory. | Penyesuaian stok dan nilai persediaan |
| Stok tersedia | Saldo stok item inventory. | Stock hint, laporan inventory |
| Status aktif | Menandai item aktif/tidak aktif di daftar. | Tampilan daftar item dan filter transaksi tertentu |

### Wajib/Opsional

| Field | Wajib/Opsional | Validasi dari kode | Contoh input |
| --- | --- | --- | --- |
| `name` | Wajib | String, maksimal 50 karakter. | Barang A |
| `type` | Wajib | Enum `service`, `inventory`, `non-inventory`. | `inventory` |
| `code` | Opsional | String maksimal 50, unik untuk item aktif/non-deleted. Jika kosong dibuat otomatis. | `INV-000001` |
| `category_id` | Opsional | ID kategori valid. | Alat Kesehatan |
| `is_sellable` | Opsional boolean | Jika true, harga jual dan akun pendapatan wajib. | Ya |
| `sell_price` | Wajib jika sellable | Numeric minimal 0. | 1000000 |
| `sell_account_id` | Wajib jika sellable | ID akun valid; service memvalidasi root type income. | Pendapatan Penjualan |
| `sell_tax_rate_id` | Opsional | ID tax rate valid. | PPN 11% |
| `sell_description` | Opsional | String maksimal 100 karakter. | Barang untuk pasien |
| `is_purchasable` | Opsional boolean | Jika true, harga beli dan akun biaya wajib. | Ya |
| `cost_price` | Wajib jika purchasable | Numeric minimal 0. | 800000 |
| `cost_account_id` | Wajib jika purchasable | ID akun valid; service memvalidasi root type expense. | HPP |
| `purchase_tax_rate_id` | Opsional | ID tax rate valid. | PPN 11% |
| `purchase_description` | Opsional | String maksimal 100 karakter. | Pembelian dari supplier |
| `inventory_account_id` | Wajib jika tipe inventory | ID akun valid; service mensyaratkan account type `Inventory`. | Persediaan Barang |
| `quantity_on_hand` | Read-only dari proses stok | Tersimpan decimal, tetapi tidak ada input utama pada form item yang diaudit. | 10 |
| `is_active` | Ada di model dan badge daftar | Input langsung pada form item belum terverifikasi dari kode pada phase ini. | Active |

Kode otomatis dari `ItemService`:

| Tipe | Prefix otomatis | Contoh |
| --- | --- | --- |
| Service | `JS-` | `JS-000001` |
| Inventory | `INV-` | `INV-000001` |
| Non-Inventory | `NI-` | `NI-000001` |

### Contoh Input

| Tipe | Kode | Nama | Kategori | Harga Jual | Harga Beli | Akun Pendapatan | Akun Biaya/HPP | Akun Persediaan | Pajak Jual | Pajak Beli |
| --- | --- | --- | --- | ---: | ---: | --- | --- | --- | --- | --- |
| Persediaan | `INV-000001` | Barang A | Alat Kesehatan | 1.000.000 | 800.000 | Pendapatan Penjualan | HPP | Persediaan Barang | PPN 11% | PPN 11% |
| Jasa | `JS-000001` | Jasa Konsultasi | Jasa | 500.000 | 0 | Pendapatan Jasa | Beban Jasa | - | PPN 11% | - |
| Non-Inventory | `NI-000001` | ATK Sekali Pakai | Operasional | 50.000 | 35.000 | Pendapatan Penjualan | Beban ATK | - | PPN 11% | PPN 11% |

Catatan presentasi:

- Untuk item jasa dan non-inventory, jelaskan bahwa stok tidak dilacak.
- Untuk item inventory, pastikan akun persediaan dipilih agar item bisa dipakai pada proses stok.
- Jika kode dikosongkan, sistem membuat kode otomatis sesuai tipe.

## 8. Kategori Barang/Jasa

Kategori Barang/Jasa membantu admin mengelompokkan item. Kategori juga punya field akun default, tetapi efek otomatis akun kategori ke item/transaksi belum terlihat sebagai alur penuh pada audit phase ini.

| Field kategori | Wajib/Opsional | Validasi dari kode | Fungsi awam |
| --- | --- | --- | --- |
| `name` | Wajib | String maksimal 255. | Nama kelompok item. |
| `description` | Opsional | String maksimal 255. | Penjelasan kategori. |
| `sell_account_id` | Opsional | ID akun valid; UI memfilter Income/Other Income. | Akun pendapatan default kategori. |
| `cost_account_id` | Opsional | ID akun valid; UI memfilter Expense/Other Expense/COGS. | Akun biaya/HPP default kategori. |
| `inventory_account_id` | Opsional | ID akun valid; UI menampilkan inventory/current asset/other current asset. | Akun persediaan default kategori. |
| `cost_method` | Ada di model/migration | Input UI belum terverifikasi dari kode pada phase ini. | Metode biaya kategori. |

Perilaku hapus kategori:

- Kategori bisa dihapus.
- Item yang memakai kategori tersebut tidak dihapus.
- `category_id` pada item terkait akan dikosongkan.

Contoh kategori:

| Kategori | Deskripsi | Akun pendapatan | Akun biaya/HPP | Akun persediaan |
| --- | --- | --- | --- | --- |
| Alat Kesehatan | Barang fisik yang disimpan di gudang. | Pendapatan Penjualan | HPP | Persediaan Barang |
| Jasa | Layanan konsultasi dan instalasi. | Pendapatan Jasa | Beban Jasa | - |
| Operasional | Barang non-inventory untuk operasional. | Pendapatan Penjualan | Beban Operasional | - |

Error umum kategori:

| Error | Penyebab | Cara menghindari |
| --- | --- | --- |
| Nama kosong | `name` wajib. | Isi nama kategori. |
| Deskripsi terlalu panjang | Maksimal 255 karakter. | Ringkas deskripsi. |
| Akun tidak valid | ID akun tidak ditemukan. | Pilih akun dari dropdown. |
| Duplicate name | Belum terverifikasi dari kode pada phase ini. | Gunakan nama kategori yang jelas dan unik secara operasional. |
| Kategori sudah dipakai item | Tidak diblokir; relasi item dikosongkan saat kategori dihapus. | Cek item terkait sebelum menghapus kategori. |

## 9. Penyesuaian Persediaan

Penyesuaian Persediaan dipakai untuk stok awal, koreksi stok, dan hasil stock opname. Menu ini tidak dipakai untuk transaksi penjualan/pembelian normal.

Alur status:

| Status | Arti | Efek |
| --- | --- | --- |
| Draft | Penyesuaian dibuat tetapi belum diposting. | Belum mengubah stok/GL. |
| Published | Penyesuaian sudah dipublish. | Membuat GL, memperbarui `items.quantity_on_hand`, memperbarui stok gudang jika gudang dipilih, dan membuat `inventory_transactions`. |

Field header:

| Field | Wajib/Opsional | Validasi dari kode | Contoh |
| --- | --- | --- | --- |
| `type` | Wajib | `increment` atau `decrement`. | Increment |
| `date` | Wajib | Tanggal valid. | 2026-05-01 |
| `reference_no` | Opsional | String maksimal 100, unik untuk record non-deleted. Jika kosong dibuat otomatis `IA-000001`. | `IA-000001` |
| `adjustment_account_id` | Wajib | Akun valid. | Selisih Persediaan |
| `reason` | Opsional | String maksimal 100. | Opening stock |
| `warehouse_id` | Opsional | Gudang valid. | Gudang Utama |
| `branch_id` | Opsional di request/model | Input halaman create belum terverifikasi dari kode pada phase ini. | Cabang Jakarta |
| `description` | Opsional | String maksimal 500. | Stok awal presentasi |

Field item:

| Field | Wajib/Opsional | Validasi dari kode | Contoh |
| --- | --- | --- | --- |
| `entries` | Wajib | Array minimal 1 baris. | 1 item |
| `entries.*.item_id` | Wajib | Item harus ada, bertipe `inventory`, dan tidak soft-deleted. | Barang A |
| `entries.*.quantity` | Wajib | Numeric minimal 0.00001. | 10 |
| `entries.*.cost` | Wajib | Numeric minimal 0. | 800000 |

Pengaruh posting:

| Jenis | Stok | GL | Inventory transaction |
| --- | --- | --- | --- |
| Increment | Menambah stok item; menambah stok gudang jika `warehouse_id` dipilih. | Debit akun persediaan item, kredit akun penyesuaian. | Direction `IN`. |
| Decrement | Mengurangi stok item; stok gudang dikurangi sampai minimal 0 jika `warehouse_id` dipilih. | Debit akun penyesuaian, kredit akun persediaan item. | Direction `OUT`. |

Prasyarat route create:

- Minimal satu akun aktif tersedia untuk akun penyesuaian.
- Minimal satu item aktif bertipe `inventory` tersedia.
- Minimal satu gudang tersedia.

## 10. Transfer Gudang

Transfer Gudang dipakai untuk memindahkan stok antar gudang. Service yang diaudit menyatakan transfer tidak membuat jurnal GL, hanya mengubah kuantitas per gudang dan membuat `inventory_transactions`.

Alur status:

| Status | Kondisi | Efek |
| --- | --- | --- |
| Draft | `transfer_initiated_at` kosong. | Bisa diedit dan dihapus. |
| In Transit | `transfer_initiated_at` terisi, `transfer_delivered_at` kosong. | Stok gudang asal dikurangi dan dicatat sebagai OUT. |
| Transferred | `transfer_delivered_at` terisi. | Stok gudang tujuan ditambah dan dicatat sebagai IN. |

Field header:

| Field | Wajib/Opsional | Validasi dari kode | Contoh |
| --- | --- | --- | --- |
| `date` | Wajib | Tanggal valid. | 2026-05-10 |
| `transaction_number` | Opsional | String maksimal 100; service memeriksa duplikat jika diisi. Jika kosong pada UI, route menyediakan nomor `WT-000001`. | `WT-000001` |
| `from_warehouse_id` | Wajib | Gudang valid. | Gudang Utama |
| `to_warehouse_id` | Wajib | Gudang valid dan berbeda dari gudang asal. | Gudang Cabang |

Field item:

| Field | Wajib/Opsional | Validasi dari kode | Contoh |
| --- | --- | --- | --- |
| `entries` | Wajib | Array minimal 1 baris. | 1 item |
| `entries.*.item_id` | Wajib | Item harus ada, bertipe `inventory`, dan tidak soft-deleted. | Barang A |
| `entries.*.quantity` | Wajib | Numeric minimal 0.00001. | 3 |
| `entries.*.cost` | Opsional | Numeric minimal 0. | 800000 |
| `entries.*.description` | Opsional | String maksimal 500. | Transfer untuk demo |

Prasyarat route create:

- Minimal dua gudang tersedia.
- Minimal satu item aktif bertipe `inventory` tersedia.
- User harus punya permission warehouse transfer sesuai aksi.

Catatan stok:

- Transfer Gudang mengubah lokasi stok per gudang.
- Service tidak mengubah total `items.quantity_on_hand` organisasi.
- Pada initiate, stok gudang asal dikurangi dengan ekspresi minimal 0.
- Hard-block untuk stok kurang sebelum transfer belum terverifikasi dari kode pada phase ini.

## 11. Pengaruh Barang/Jasa Ke Transaksi Penjualan

Transaksi penjualan memakai item yang difilter server-side sebagai `sellable`.

| Modul | Memakai item? | Harga default | Pajak default | Stock hint | Stok berubah? | Status audit |
| --- | ---: | --- | --- | --- | --- | --- |
| Estimate | Ya | `sell_price`, fallback `cost_price` di `LineItemsTable` | `sell_tax_rate_id` | Sales context: stok dan sisa draft untuk inventory; non-inventory/jasa tidak dilacak | Tidak ada GL; mutasi stok tidak ditemukan di service estimate | Tersedia |
| Invoice | Ya | `sell_price`, fallback `cost_price` | `sell_tax_rate_id` | Sales context | Service membuat GL saat deliver; mutasi `quantity_on_hand`/`inventory_transactions` belum terverifikasi dari kode pada phase ini | Tersedia |
| Sale Receipt | Ya | `sell_price`, fallback `cost_price` | `sell_tax_rate_id` | Sales context | Service membuat GL saat close; mutasi `quantity_on_hand`/`inventory_transactions` belum terverifikasi dari kode pada phase ini | Tersedia |
| Credit Note | Ya | `sell_price`, fallback `cost_price` | `sell_tax_rate_id` | Neutral context: stok saat ini saja untuk inventory | Service membuat GL reversal saat open; mutasi stok belum terverifikasi dari kode pada phase ini | Tersedia |

Perilaku line item yang terverifikasi:

- Saat item dipilih, deskripsi sales memakai `sell_description`, fallback ke `purchase_description`.
- Rate default sales memakai `sell_price`, fallback ke `cost_price`.
- Pajak default sales memakai `sell_tax_rate_id`.
- Item inventory menampilkan `Stok: ...` dan `Sisa draft: ...`.
- Jika draft melebihi stok, UI menampilkan pesan `Melebihi stok tersedia`.
- Jasa dan non-inventory menampilkan `Stok tidak dilacak`.

Catatan penting:

- Stock hint adalah preview di UI, bukan bukti mutasi stok aktual.
- Auto-fill `sell_account_id` dari master item ke line item belum terverifikasi dari kode pada phase ini.

## 12. Pengaruh Barang/Jasa Ke Transaksi Pembelian

Transaksi pembelian memakai item yang difilter server-side sebagai `purchasable`.

| Modul | Memakai item? | Harga default | Pajak default | Stock hint | Stok berubah? | Status audit |
| --- | ---: | --- | --- | --- | --- | --- |
| Bill | Ya | `cost_price`, fallback `sell_price` | `purchase_tax_rate_id` | Purchases context: stok dan setelah draft untuk inventory | Service membuat GL saat open; mutasi `quantity_on_hand`/`inventory_transactions` belum terverifikasi dari kode pada phase ini | Tersedia |
| Vendor Credit | Ya | `cost_price`, fallback `sell_price` | `purchase_tax_rate_id` | Neutral context: stok saat ini saja untuk inventory | Service membuat GL reversal saat open; mutasi stok belum terverifikasi dari kode pada phase ini | Tersedia |
| Expense | Tidak memakai master item pada UI yang diaudit | Tidak berlaku | Tidak berlaku | Tidak berlaku | Expense membuat GL biaya saat publish, bukan stok item | Tersedia sebagai modul biaya, bukan item |

Perilaku line item yang terverifikasi:

- Saat item dipilih, deskripsi purchase memakai `purchase_description`, fallback ke `sell_description`.
- Rate default purchase memakai `cost_price`, fallback ke `sell_price`.
- Pajak default purchase memakai `purchase_tax_rate_id`.
- Bill menampilkan stock hint `Setelah draft: ...` untuk item inventory.
- Vendor Credit memakai stock preview neutral, sehingga hanya menampilkan stok saat ini.
- Pada Bill, checkbox landed cost tersedia; item inventory membuat checkbox landed cost disabled.

Catatan penting:

- Auto-fill `cost_account_id` dari master item ke line item belum terverifikasi dari kode pada phase ini.
- Bill memiliki halaman Landed Cost. Landed cost dapat membuat `inventory_transactions` untuk item inventory target, tetapi detail alur landed cost bukan fokus dokumen ini.

## 13. Pengaruh Barang/Jasa Ke Stok dan Gudang

| Aktivitas | Item yang terlibat | Pengaruh stok | Pengaruh gudang | Catatan |
| --- | --- | --- | --- | --- |
| Membuat item Service | `service` | Tidak menambah stok. | Tidak ada. | Untuk jasa/layanan. |
| Membuat item Inventory | `inventory` | Stok awal tetap 0 sampai ada proses stok. | Bisa masuk stok gudang melalui penyesuaian/transfer. | Wajib akun persediaan. |
| Membuat item Non-Inventory | `non-inventory` | Tidak menambah stok. | Tidak ada. | Bisa dijual/dibeli tanpa stok. |
| Penyesuaian Persediaan Increment | `inventory` | Menambah `items.quantity_on_hand`. | Menambah stok gudang jika gudang dipilih. | Membuat GL dan inventory transaction IN. |
| Penyesuaian Persediaan Decrement | `inventory` | Mengurangi `items.quantity_on_hand`. | Mengurangi stok gudang sampai minimal 0 jika gudang dipilih. | Membuat GL dan inventory transaction OUT. |
| Transfer Gudang initiate | `inventory` | Total item organisasi tidak diubah oleh service transfer. | Mengurangi stok gudang asal. | Membuat inventory transaction OUT. |
| Transfer Gudang deliver | `inventory` | Total item organisasi tidak diubah oleh service transfer. | Menambah stok gudang tujuan. | Membuat inventory transaction IN. |
| Sales/purchase document biasa | Item sellable/purchasable | Mutasi stok aktual belum terverifikasi dari kode pada phase ini. | Warehouse id ada di beberapa request transaksi, tetapi alur stok penuh belum terverifikasi. | Gunakan laporan dan uji manual sebelum demo klaim stok otomatis. |

Aturan praktis untuk presentasi:

- Jika user bertanya "kenapa stok tidak muncul?", cek tipe item. Hanya `inventory` yang dilacak stok.
- Jika user bertanya "kenapa stok masih 0?", pastikan stok awal sudah dipublish lewat Penyesuaian Persediaan.
- Jika user bertanya "kenapa tidak bisa transfer?", pastikan item bertipe inventory, aktif, dan minimal dua gudang tersedia.

## 14. Pengaruh Barang/Jasa Ke Laporan

Laporan yang terkait langsung dengan item dan inventory:

| Laporan | Route | Sumber data utama | Export/PDF | Permission |
| --- | --- | --- | --- | --- |
| Sales by Items | `/reports/sales-by-items` | `item_entries` yang terhubung ke `sale_invoices` delivered. | Export CSV/XLSX dan PDF tersedia. | `report-sales-by-items.view` atau legacy report view |
| Purchases by Items | `/reports/purchases-by-items` | `item_entries` yang terhubung ke `bills` opened. | Export CSV/XLSX dan PDF tersedia. | `report-purchases-by-items.view` atau legacy report view |
| Inventory Valuation Sheet | `/reports/inventory-valuation-sheet` | `inventory_transactions` sampai tanggal laporan. | Export CSV/XLSX dan PDF tersedia. | `report-inventory-valuation-sheet.view` atau legacy report view |
| Inventory Item Details | `/reports/inventory-item-details` | `inventory_transactions` per item dan rentang tanggal. | Export CSV/XLSX tersedia; PDF tidak muncul di route yang diaudit. | `report-inventory-item-details.view` atau legacy report view |

Dampak tidak langsung:

- Akun pendapatan, biaya/HPP, pajak, piutang, dan utang dari transaksi item masuk ke GL ketika dokumen diposting sesuai service masing-masing.
- Income Statement, General Ledger, Journal Sheet, dan Balance Sheet dapat terdampak oleh GL dari invoice, sale receipt, credit note, bill, vendor credit, dan inventory adjustment.
- Laporan inventory valuation membaca `inventory_transactions`, sehingga hanya proses yang benar-benar membuat inventory transaction yang masuk ke laporan tersebut.

## 15. Contoh Data Awal Untuk Presentasi

Master data pendukung:

| Jenis data | Contoh | Catatan |
| --- | --- | --- |
| Akun pendapatan | Pendapatan Penjualan, Pendapatan Jasa | Untuk item sellable. |
| Akun biaya/HPP | HPP, Beban Jasa, Beban ATK | Untuk item purchasable. |
| Akun persediaan | Persediaan Barang | Wajib untuk inventory item. |
| Akun penyesuaian | Selisih Persediaan | Dipakai Penyesuaian Persediaan. |
| Tax rate | PPN 11% | Dipakai default pajak jual/beli. |
| Gudang | Gudang Utama, Gudang Cabang | Minimal dua untuk transfer. |
| Kategori | Alat Kesehatan, Jasa, Operasional | Untuk pengelompokan item. |

Contoh item:

| Tipe | Kode | Nama | Kategori | Sellable | Purchasable | Harga Jual | Harga Beli | Akun Pendapatan | Akun Biaya/HPP | Akun Persediaan |
| --- | --- | --- | --- | ---: | ---: | ---: | ---: | --- | --- | --- |
| Inventory | `INV-000001` | Barang A | Alat Kesehatan | Ya | Ya | 1.000.000 | 800.000 | Pendapatan Penjualan | HPP | Persediaan Barang |
| Service | `JS-000001` | Jasa Konsultasi | Jasa | Ya | Tidak | 500.000 | 0 | Pendapatan Jasa | - | - |
| Non-Inventory | `NI-000001` | ATK Sekali Pakai | Operasional | Ya | Ya | 50.000 | 35.000 | Pendapatan Penjualan | Beban ATK | - |

Contoh stok awal:

| Field | Isi |
| --- | --- |
| Nomor | `IA-000001` |
| Tanggal | 2026-05-01 |
| Tipe | Increment |
| Akun penyesuaian | Selisih Persediaan |
| Gudang | Gudang Utama |
| Item | Barang A |
| Quantity | 10 |
| Cost/unit | 800.000 |

Contoh transfer gudang:

| Field | Isi |
| --- | --- |
| Nomor | `WT-000001` |
| Tanggal | 2026-05-10 |
| Gudang asal | Gudang Utama |
| Gudang tujuan | Gudang Cabang |
| Item | Barang A |
| Quantity | 3 |
| Cost/unit | 800.000 |

## 16. Contoh Alur Demo Barang/Jasa

1. Buka Bagan Akun dan pastikan akun Pendapatan Penjualan, HPP, Persediaan Barang, Selisih Persediaan, Piutang, Utang, dan Pajak tersedia.
2. Buka Preferences > Warehouses dan pastikan ada Gudang Utama serta Gudang Cabang.
3. Buka Items > Categories, buat kategori `Alat Kesehatan`.
4. Buka Items > Items, buat item inventory `Barang A` dengan harga jual 1.000.000 dan harga beli 800.000.
5. Buat item service `Jasa Konsultasi`.
6. Buat item non-inventory `ATK Sekali Pakai`.
7. Buka Inventory Adjustments, buat `IA-000001` tipe Increment untuk `Barang A` sebanyak 10 unit di Gudang Utama.
8. Publish inventory adjustment dan jelaskan efek stok, GL, dan inventory transaction.
9. Buka Invoice atau Estimate, pilih `Barang A`, tunjukkan harga jual, pajak default, dan stock hint `Sisa draft`.
10. Pilih `Jasa Konsultasi`, tunjukkan pesan stok tidak dilacak.
11. Buka Bill, pilih `Barang A`, tunjukkan harga beli/cost dan stock hint `Setelah draft`.
12. Buka Warehouse Transfer, buat `WT-000001` dari Gudang Utama ke Gudang Cabang sebanyak 3 unit.
13. Initiate transfer untuk menunjukkan stok keluar dari gudang asal.
14. Deliver transfer untuk menunjukkan stok masuk ke gudang tujuan.
15. Buka laporan Sales by Items, Purchases by Items, Inventory Valuation Sheet, dan Inventory Item Details.

## 17. Error Umum dan Cara Menghindari

| Error/kondisi | Penyebab | Cara menghindari |
| --- | --- | --- |
| Item inventory ditolak | `inventory_account_id` kosong atau bukan akun Inventory. | Buat/pilih akun Persediaan Barang bertipe Inventory. |
| Item sellable ditolak | Harga jual atau akun pendapatan kosong. | Isi `sell_price` dan pilih akun Income/Other Income. |
| Item purchasable ditolak | Harga beli atau akun biaya kosong. | Isi `cost_price` dan pilih Expense/Other Expense/COGS. |
| Nama item terlalu panjang | `name` maksimal 50 karakter. | Ringkas nama item. |
| Deskripsi terlalu panjang | Deskripsi sales/purchase maksimal 100 karakter. | Ringkas deskripsi. |
| Kode item duplikat | Kode sudah dipakai item aktif/non-deleted. | Gunakan kode lain atau kosongkan agar sistem membuat otomatis. |
| Item tidak bisa dihapus | Sudah dipakai di `item_entries`. | Nonaktifkan secara operasional atau biarkan sebagai histori; delete diblokir. |
| Item tidak muncul di invoice/estimate | `is_sellable` tidak aktif atau item tidak masuk filter sellable. | Aktifkan dapat dijual dan cek item aktif. |
| Item tidak muncul di bill/vendor credit | `is_purchasable` tidak aktif atau item tidak masuk filter purchasable. | Aktifkan dapat dibeli dan cek item aktif. |
| Item tidak muncul di Penyesuaian Persediaan | Bukan tipe inventory atau tidak aktif. | Ubah/buat item bertipe `inventory`. |
| Tidak bisa buat Penyesuaian Persediaan | Akun penyesuaian, item inventory, atau gudang belum ada. | Lengkapi prasyarat route create. |
| Tidak bisa hapus Penyesuaian Persediaan | Sudah dipublish. | Jangan hapus transaksi stok yang sudah diposting; buat koreksi baru jika perlu. |
| Tidak bisa buat Transfer Gudang | Gudang kurang dari dua atau item inventory belum ada. | Buat dua gudang dan item inventory aktif. |
| Gudang asal sama dengan tujuan | Request mensyaratkan `different:from_warehouse_id`. | Pilih gudang tujuan berbeda. |
| Tidak bisa edit/hapus Transfer Gudang | Transfer sudah initiated. | Edit sebelum initiate. |
| Periode terkunci untuk Inventory Adjustment/Warehouse Transfer | Belum terverifikasi dari kode pada phase ini. | Untuk demo, gunakan tanggal periode terbuka dan lakukan uji manual. |
| Stok kurang saat transfer | Hard-block belum terverifikasi dari kode pada phase ini; service mengurangi stok gudang asal sampai minimal 0. | Pastikan stok cukup sebelum demo. |

## 18. Checklist Setelah Setup Barang/Jasa

- Akun pendapatan, biaya/HPP, dan persediaan tersedia di Bagan Akun.
- Akun penyesuaian tersedia untuk Penyesuaian Persediaan.
- Tax rate yang akan dipakai item tersedia.
- Gudang tersedia, minimal dua jika demo Transfer Gudang.
- Kategori Barang/Jasa sudah dibuat.
- Item service, inventory, dan non-inventory sudah dibuat sesuai kebutuhan.
- Item inventory memiliki akun persediaan.
- Item yang akan dijual punya harga jual dan akun pendapatan.
- Item yang akan dibeli punya harga beli dan akun biaya/HPP.
- Stok awal item inventory sudah dipublish lewat Penyesuaian Persediaan.
- Invoice/Estimate menampilkan stock hint untuk item inventory.
- Bill menampilkan stock hint setelah draft untuk item inventory.
- Service dan non-inventory menampilkan stok tidak dilacak.
- Laporan inventory/item bisa dibuka dengan filter tanggal yang berisi transaksi.

## 19. Checklist Presentasi/Demo

- Login sebagai user Inventory Officer atau admin.
- Pastikan role memiliki permission `item.*`, `inventory-adjustment.*`, `warehouse-transfer.*`, dan report terkait.
- Buka menu Items dan tunjukkan daftar Barang/Jasa.
- Tunjukkan perbedaan tipe Service, Inventory, dan Non-Inventory.
- Buat atau edit item dengan nama kurang dari 50 karakter.
- Tunjukkan kode otomatis berdasarkan tipe item.
- Tunjukkan kategori dan akun default item.
- Buat Penyesuaian Persediaan untuk stok awal, lalu publish.
- Jelaskan GL inventory adjustment dengan bahasa sederhana.
- Buat transaksi sales dan pilih item inventory untuk menunjukkan harga jual, pajak, dan stock hint.
- Pilih item service untuk menunjukkan stok tidak dilacak.
- Buat transaksi purchase dan pilih item inventory untuk menunjukkan harga beli/cost dan stock hint setelah draft.
- Buat Transfer Gudang jika minimal dua gudang tersedia.
- Buka laporan Sales by Items, Purchases by Items, Inventory Valuation Sheet, dan Inventory Item Details.
- Sampaikan dengan jelas area yang masih perlu uji manual, terutama mutasi stok dari dokumen sales/purchase biasa.

## 20. Catatan Field/Menu Yang Belum Terverifikasi

Catatan yang harus disampaikan sebagai batas presentasi:

- Input UI untuk `note`, `picture_uri`, dan `is_landed_cost` pada form Barang/Jasa belum terverifikasi dari kode pada phase ini.
- Input UI untuk mengubah `is_active` item belum terverifikasi dari kode pada phase ini; daftar item hanya menampilkan badge aktif/tidak aktif.
- `quantity_on_hand` ada di model dan dipakai stock hint, tetapi input manual stok di form Barang/Jasa belum terverifikasi dari kode pada phase ini.
- Auto-fill `sell_account_id` dan `cost_account_id` dari master item ke line item transaksi belum terverifikasi dari kode pada phase ini.
- Efek akun default kategori langsung ke item atau transaksi belum terverifikasi dari kode pada phase ini.
- Validasi duplicate name kategori belum terverifikasi dari kode pada phase ini.
- Input UI `branch_id` pada form Penyesuaian Persediaan belum terverifikasi dari kode pada phase ini, meskipun request/model mendukung field tersebut.
- Guard periode terkunci pada Penyesuaian Persediaan dan Transfer Gudang belum terverifikasi dari kode pada phase ini.
- Mutasi stok aktual dari Invoice, Sale Receipt, Credit Note, Bill, dan Vendor Credit biasa belum terverifikasi dari kode pada phase ini; service yang diaudit terutama membuat GL dan `item_entries`.
- Hard-block stok kurang pada Transfer Gudang belum terverifikasi dari kode pada phase ini.
- PDF untuk Inventory Item Details belum terverifikasi dari route pada phase ini; export CSV/XLSX terverifikasi.
- Screenshot presentasi belum dibuat pada phase ini.
