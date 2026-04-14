# Desain Database — ERP Finance & Accounting

---

## BAGIAN 1 — GAMBARAN UMUM DATABASE

### Statistik Keseluruhan

| Kategori | Jumlah |
|----------|--------|
| Total migration files | 39 (3 Laravel default + 36 custom) |
| Total tabel utama bisnis | 43+ tabel |
| Total Eloquent Model | 46 model |
| Tabel dengan Soft Delete | Semua tabel transaksi dan master data |
| Tabel dengan Audit Trail | Semua 46 tabel bisnis |

---

### Pengelompokan Tabel per Domain

| Kelompok | Tabel | Fungsi |
|----------|-------|--------|
| **1. Sistem** | `users`, `sessions`, `cache`, `jobs`, `personal_access_tokens` | Autentikasi, session, queue |
| **2. RBAC** | `roles`, `permissions`, `model_has_roles`, `model_has_permissions`, `role_has_permissions` | Otorisasi berbasis peran |
| **3. Master Data** | `currencies`, `exchange_rates`, `accounts`, `contacts`, `items`, `item_categories`, `tax_rates`, `settings`, `branches`, `warehouses` | Data referensi |
| **4. Transaksi AR** | `sale_invoices`, `sale_estimates`, `payment_receives`, `payment_receive_entries`, `credit_notes`, `credit_note_applied_invoices`, `refund_credit_notes`, `sale_receipts` | Piutang usaha |
| **5. Transaksi AP** | `bills`, `bill_payments`, `bill_payment_entries`, `vendor_credits`, `vendor_credit_applied_bills`, `refund_vendor_credits` | Hutang usaha |
| **6. Akuntansi** | `account_transactions`, `manual_journals`, `manual_journal_entries`, `expenses`, `expense_categories` | Buku besar dan jurnal |
| **7. Pendukung** | `item_entries`, `tax_rate_transactions`, `landed_costs`, `landed_cost_entries`, `inventory_adjustments`, `inventory_adjustment_entries`, `inventory_transactions`, `cashflow_transactions`, `cashflow_transaction_lines`, `warehouse_transfers`, `warehouse_transfer_entries`, `documents`, `document_links`, `projects`, `tasks`, `time_entries` | Fitur operasional |

---

### Strategi Desain

- **Soft Delete:** Semua tabel transaksi keuangan dan master data menggunakan `deleted_at` timestamp. Data tidak pernah dihapus permanen.
- **Audit Trail:** Kolom `created_by`, `updated_by`, `deleted_by` ada di semua 46 tabel, diisi otomatis oleh `Auditable` trait.
- **Tipe Data Uang:** `DECIMAL(15,5)` untuk semua nilai moneter — mendukung nilai hingga 999,999,999,999.99999.
- **Tipe Data Kurs:** `DECIMAL(13,9)` untuk kurs tukar — presisi tinggi untuk perhitungan multi-currency.
- **Foreign Key Design:** Semua FK menggunakan `nullOnDelete()` sehingga data tidak orphan saat referensi dihapus.

---

## BAGIAN 2 — DOKUMENTASI PER TABEL

---

### KELOMPOK 1 — TABEL SISTEM

---

#### Tabel: `users`

**Tujuan:** Menyimpan akun pengguna sistem. Mendukung login web (session) dan API (token).

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT UNSIGNED | NO | Primary key auto-increment |
| `name` | VARCHAR(255) | NO | Nama lengkap pengguna |
| `email` | VARCHAR(255) | NO | Email unik, digunakan untuk login |
| `email_verified_at` | TIMESTAMP | YES | Waktu verifikasi email |
| `password` | VARCHAR(255) | NO | Bcrypt hash password |
| `remember_token` | VARCHAR(100) | YES | Token "Remember Me" |
| `created_by` | BIGINT UNSIGNED | YES | Audit: user yang membuat |
| `updated_by` | BIGINT UNSIGNED | YES | Audit: user terakhir mengubah |
| `deleted_by` | BIGINT UNSIGNED | YES | Audit: user yang menghapus |
| `deleted_at` | TIMESTAMP | YES | Soft delete timestamp |
| `created_at` | TIMESTAMP | YES | Waktu dibuat |
| `updated_at` | TIMESTAMP | YES | Waktu diperbarui |

**Index:** `email` (UNIQUE), `deleted_at`
**Soft Delete:** Ya
**Audit Trail:** Ya

---

#### Tabel: `personal_access_tokens`

**Tujuan:** Menyimpan Sanctum API tokens untuk autentikasi eksternal.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT | Primary key |
| `tokenable_type` | VARCHAR | Polymorphic: model pemilik token |
| `tokenable_id` | BIGINT | ID model pemilik |
| `name` | VARCHAR | Nama token |
| `token` | VARCHAR(64) | Hash token (UNIQUE) |
| `abilities` | TEXT | JSON array permission token |
| `last_used_at` | TIMESTAMP | Waktu terakhir digunakan |
| `expires_at` | TIMESTAMP | Waktu kadaluarsa |

---

### KELOMPOK 2 — TABEL RBAC (Spatie Laravel Permission)

---

#### Tabel: `roles`

**Tujuan:** Mendefinisikan peran pengguna (admin, staff, viewer).

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT | Primary key |
| `name` | VARCHAR | Nama role (UNIQUE per guard) |
| `guard_name` | VARCHAR | Guard Laravel ('web') |

---

#### Tabel: `permissions`

**Tujuan:** Mendefinisikan hak akses atomik dalam format `{module}.{action}`.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT | Primary key |
| `name` | VARCHAR | Nama permission, contoh: `sale-invoice.create` |
| `guard_name` | VARCHAR | Guard Laravel ('web') |

**Total permission yang dibuat seeder:** 79 permission dari 20 modul.

---

#### Tabel: `model_has_roles`

**Tujuan:** Relasi many-to-many antara user dan role.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `role_id` | BIGINT | FK → roles.id |
| `model_type` | VARCHAR | Polymorphic: 'App\Models\User' |
| `model_id` | BIGINT | FK → users.id |

---

#### Tabel: `role_has_permissions`

**Tujuan:** Relasi many-to-many antara role dan permission.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `permission_id` | BIGINT | FK → permissions.id |
| `role_id` | BIGINT | FK → roles.id |

---

### KELOMPOK 3 — TABEL MASTER DATA

---

#### Tabel: `currencies`

**Tujuan:** Daftar mata uang yang didukung sistem.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `name` | VARCHAR | NO | Nama mata uang, contoh: "Indonesian Rupiah" |
| `code` | VARCHAR(4) | NO | Kode ISO 4217, contoh: "IDR" (UNIQUE) |
| `symbol` | VARCHAR(10) | NO | Simbol, contoh: "Rp" |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

**Index:** `code` (UNIQUE), `name`
**Soft Delete:** Ya — restore-or-create pattern saat kode yang sama dibuat ulang
**Audit Trail:** Ya

---

#### Tabel: `exchange_rates`

**Tujuan:** Riwayat kurs tukar harian per mata uang terhadap base currency (IDR).

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `currency_id` | BIGINT | NO | FK → currencies.id |
| `rate` | DECIMAL(13,9) | NO | Nilai kurs (misalnya: 15750.000000000 untuk USD→IDR) |
| `date` | DATE | NO | Tanggal berlakunya kurs |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |

**Foreign Key:** `currency_id` → `currencies.id`
**Soft Delete:** Tidak

---

#### Tabel: `accounts`

**Tujuan:** Bagan Akun (Chart of Accounts) — fondasi seluruh sistem double-entry accounting.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `name` | VARCHAR | NO | Nama akun, contoh: "Accounts Receivable" |
| `slug` | VARCHAR | YES | URL slug dari nama akun |
| `account_type` | VARCHAR | NO | Tipe akun (lihat enum AccountType) |
| `code` | VARCHAR(20) | YES | Kode akun, contoh: "12001" |
| `description` | TEXT | YES | Deskripsi akun |
| `parent_account_id` | BIGINT | YES | FK → accounts.id (self-referential) |
| `is_active` | BOOLEAN | NO | Status aktif (default: true) |
| `index` | INT UNSIGNED | NO | Urutan tampilan (default: 0) |
| `is_predefined` | BOOLEAN | NO | Akun seeded, tidak bisa dihapus |
| `is_system_account` | BOOLEAN | NO | Akun kritis sistem (AR, AP, PPN, dll.) |
| `balance` | DECIMAL(15,5) | NO | Saldo berjalan akun |
| `currency_code` | VARCHAR(4) | YES | Mata uang akun (untuk Cash/Bank multi-currency) |
| `seeded_at` | DATE | YES | Tanggal seeding |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

**Index:** `name`, `account_type`, `code`, `is_active`, `is_predefined`, `currency_code`
**Foreign Key:** `parent_account_id` → `accounts.id` (NULL on delete)
**Soft Delete:** Ya
**Audit Trail:** Ya

**Hierarki Akun:** Maksimal 5 level kedalaman. Akun 4-digit = grup struktural, akun 5-digit = akun transaksional.

**Aturan Penghapusan:**
1. `is_predefined = true` → tidak bisa dihapus (PredefinedAccountException)
2. `children_count > 0` → tidak bisa dihapus (AccountHasChildrenException)
3. Ada `account_transactions` → tidak bisa dihapus (AccountHasTransactionsException)

---

#### Tabel: `contacts`

**Tujuan:** Direktori pelanggan (customer) dan vendor dalam satu tabel, dibedakan oleh `contact_service`.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `contact_service` | VARCHAR | NO | Tipe: 'customer' atau 'vendor' |
| `display_name` | VARCHAR | NO | Nama tampilan |
| `salutation` | VARCHAR | YES | Sapaan (Bpk/Ibu/dll.) |
| `first_name` | VARCHAR | YES | Nama depan |
| `last_name` | VARCHAR | YES | Nama belakang |
| `company_name` | VARCHAR | YES | Nama perusahaan |
| `email` | VARCHAR | YES | Alamat email |
| `work_phone` | VARCHAR | YES | Telepon kantor |
| `personal_phone` | VARCHAR | YES | Telepon pribadi |
| `website` | VARCHAR | YES | Website |
| `balance` | DECIMAL(15,5) | NO | Saldo berjalan (piutang/hutang) |
| `currency_code` | VARCHAR(4) | YES | Mata uang default kontak |
| `opening_balance` | DECIMAL(15,5) | NO | Saldo awal (migrasi dari sistem lama) |
| `opening_balance_at` | DATE | YES | Tanggal saldo awal |
| `billing_address_*` | VARCHAR | YES | Alamat penagihan (8 kolom) |
| `shipping_address_*` | VARCHAR | YES | Alamat pengiriman (8 kolom) |
| `note` | TEXT | YES | Catatan internal |
| `is_active` | BOOLEAN | NO | Status aktif |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

**Index:** `contact_service`
**Soft Delete:** Ya
**Audit Trail:** Ya

---

#### Tabel: `items`

**Tujuan:** Katalog barang dan jasa yang dapat digunakan di invoice, tagihan, dan estimasi.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `name` | VARCHAR | NO | Nama item |
| `type` | VARCHAR | NO | Tipe: 'service', 'inventory', 'non-inventory' |
| `code` | VARCHAR | YES | Kode item |
| `is_sellable` | BOOLEAN | NO | Bisa dijual (default: true) |
| `is_purchasable` | BOOLEAN | NO | Bisa dibeli (default: true) |
| `sell_price` | DECIMAL(15,5) | NO | Harga jual default |
| `cost_price` | DECIMAL(15,5) | NO | Harga beli/biaya default |
| `sell_account_id` | BIGINT | YES | FK → accounts.id (akun pendapatan) |
| `cost_account_id` | BIGINT | YES | FK → accounts.id (akun HPP/beban) |
| `inventory_account_id` | BIGINT | YES | FK → accounts.id (akun persediaan) |
| `sell_description` | TEXT | YES | Deskripsi penjualan |
| `purchase_description` | TEXT | YES | Deskripsi pembelian |
| `quantity_on_hand` | DECIMAL(15,5) | NO | Stok saat ini |
| `is_landed_cost` | BOOLEAN | NO | Bisa digunakan sebagai biaya landed |
| `category_id` | BIGINT | YES | FK → item_categories.id |
| `sell_tax_rate_id` | BIGINT | YES | FK → tax_rates.id |
| `purchase_tax_rate_id` | BIGINT | YES | FK → tax_rates.id |
| `is_active` | BOOLEAN | NO | Status aktif |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

**Index:** `name`, `type`, `is_sellable`, `is_purchasable`
**Foreign Key:** `sell_account_id`, `cost_account_id`, `inventory_account_id` → `accounts.id`; `sell_tax_rate_id`, `purchase_tax_rate_id` → `tax_rates.id`
**Soft Delete:** Ya
**Audit Trail:** Ya

---

#### Tabel: `tax_rates`

**Tujuan:** Daftar tarif pajak yang tersedia untuk digunakan di baris invoice/tagihan.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `name` | VARCHAR | NO | Nama tarif, contoh: "PPN 11%" |
| `rate` | DECIMAL(8,4) | NO | Persentase, contoh: 11.0000 |
| `is_active` | BOOLEAN | NO | Status aktif |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

**Soft Delete:** Ya
**Audit Trail:** Ya

---

#### Tabel: `settings`

**Tujuan:** Key-value store untuk konfigurasi sistem (organisasi, format, zona waktu, dll.).

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `group` | VARCHAR | NO | Kelompok setting, contoh: 'organization' |
| `key` | VARCHAR | NO | Kunci setting, contoh: 'base_currency' |
| `value` | TEXT | YES | Nilai setting |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |

**Index:** Compound `(group, key)` UNIQUE
**Audit Trail:** Ya (tanpa soft delete)

**Setting yang tersedia (group: organization):**

| Key | Default | Keterangan |
|-----|---------|------------|
| `base_currency` | IDR | Mata uang dasar perusahaan |
| `fiscal_year` | january | Awal tahun fiskal |
| `accounting_basis` | accrual | Dasar akuntansi: accrual/cash |
| `organization_name` | (kosong) | Nama perusahaan |
| `date_format` | dd/MM/yyyy | Format tanggal tampilan |
| `timezone` | UTC | Zona waktu sistem |

---

#### Tabel: `branches`

**Tujuan:** Data cabang perusahaan.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `name` | VARCHAR | NO | Nama cabang |
| `code` | VARCHAR | YES | Kode cabang |
| `address` | TEXT | YES | Alamat |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

---

#### Tabel: `warehouses`

**Tujuan:** Data gudang untuk manajemen persediaan.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `name` | VARCHAR | NO | Nama gudang |
| `code` | VARCHAR | YES | Kode gudang |
| `branch_id` | BIGINT | YES | FK → branches.id |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

---

### KELOMPOK 4 — TABEL TRANSAKSI AR (Accounts Receivable)

---

#### Tabel: `sale_invoices`

**Tujuan:** Faktur penjualan kepada pelanggan. Saat di-deliver, menghasilkan jurnal GL otomatis.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `customer_id` | BIGINT | NO | FK → contacts.id |
| `invoice_date` | DATE | NO | Tanggal invoice |
| `due_date` | DATE | YES | Tanggal jatuh tempo |
| `invoice_no` | VARCHAR | YES | Nomor invoice (unik per non-deleted) |
| `reference_no` | VARCHAR | YES | Nomor referensi eksternal |
| `invoice_message` | TEXT | YES | Pesan kepada pelanggan |
| `terms_conditions` | TEXT | YES | Syarat dan ketentuan |
| `balance` | DECIMAL(15,5) | NO | Total nilai invoice |
| `payment_amount` | DECIMAL(15,5) | NO | Total yang sudah dibayar |
| `currency_code` | VARCHAR(4) | YES | Mata uang invoice |
| `exchange_rate` | DECIMAL(13,9) | NO | Kurs saat transaksi (default: 1) |
| `credited_amount` | DECIMAL(15,5) | NO | Total nota kredit yang diterapkan |
| `delivered_at` | DATE | YES | NULL = Draft; ada isi = sudah dikirim |
| `user_id` | BIGINT | YES | FK → users.id (pembuat) |
| `warehouse_id` | BIGINT | YES | FK → warehouses.id |
| `branch_id` | BIGINT | YES | FK → branches.id |
| `writtenoff_amount` | DECIMAL(15,5) | NO | Jumlah yang dihapuskan (write-off) |
| `writtenoff_at` | DATE | YES | Tanggal write-off |
| `writtenoff_expense_account_id` | BIGINT | YES | FK → accounts.id (akun beban bad debt) |
| `is_inclusive_tax` | BOOLEAN | NO | Pajak sudah termasuk dalam harga? |
| `tax_amount_withheld` | DECIMAL(15,5) | NO | PPh yang dipotong |
| `project_id` | BIGINT | YES | FK → projects.id |
| `discount` | DECIMAL(10,2) | YES | Nilai/persentase diskon |
| `discount_type` | VARCHAR | YES | Tipe diskon: 'Percentage' atau 'Amount' |
| `adjustment` | DECIMAL(10,2) | YES | Penyesuaian tambahan |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

**Index:** `invoice_date`, `due_date`, `invoice_no`, `delivered_at`, `writtenoff_at`, `deleted_at`
**Compound Index:** `(customer_id, deleted_at)`, `(customer_id, deleted_at, invoice_date)`
**Foreign Key:** `customer_id` → `contacts.id`
**Soft Delete:** Ya
**Audit Trail:** Ya

**Status Computed (tidak tersimpan di DB):**
- `Draft` = delivered_at IS NULL
- `Unpaid` = delivered_at SET, payment = 0
- `PartiallyPaid` = 0 < payment_amount < balance
- `Paid` = payment + credited + writtenoff >= balance
- `WrittenOff` = writtenoff_at IS NOT NULL
- `Overdue` = past due_date, belum lunas

---

#### Tabel: `sale_estimates`

**Tujuan:** Penawaran harga kepada pelanggan sebelum menjadi invoice.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `customer_id` | BIGINT | NO | FK → contacts.id |
| `estimate_number` | VARCHAR | YES | Nomor estimasi |
| `estimate_date` | DATE | NO | Tanggal estimasi |
| `expiry_date` | DATE | YES | Tanggal kadaluarsa penawaran |
| `amount` | DECIMAL(15,5) | NO | Total nilai estimasi |
| `currency_code` | VARCHAR(4) | YES | Mata uang |
| `exchange_rate` | DECIMAL(13,9) | NO | Kurs |
| `delivered_at` | DATE | YES | Tanggal dikirim ke pelanggan |
| `approved_at` | DATE | YES | Tanggal disetujui |
| `rejected_at` | DATE | YES | Tanggal ditolak |
| `converted_at` | DATE | YES | Tanggal dikonversi ke invoice |
| `converted_to_invoice_id` | BIGINT | YES | FK → sale_invoices.id |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

**Soft Delete:** Ya
**Audit Trail:** Ya
**Catatan:** Estimasi TIDAK menghasilkan GL entries di fase apapun.

---

#### Tabel: `payment_receives`

**Tujuan:** Header pembayaran yang diterima dari pelanggan.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `customer_id` | BIGINT | NO | FK → contacts.id |
| `deposit_account_id` | BIGINT | NO | FK → accounts.id (Cash/Bank tujuan) |
| `payment_date` | DATE | NO | Tanggal pembayaran |
| `amount` | DECIMAL(15,5) | NO | Total pembayaran |
| `currency_code` | VARCHAR(4) | YES | Mata uang |
| `exchange_rate` | DECIMAL(13,9) | NO | Kurs |
| `reference_no` | VARCHAR | YES | Nomor referensi/bukti transfer |
| `note` | TEXT | YES | Catatan |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

**Index:** `(customer_id, deleted_at)`

---

#### Tabel: `payment_receive_entries`

**Tujuan:** Alokasi pembayaran ke invoice tertentu (satu payment bisa untuk banyak invoice).

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `payment_receive_id` | BIGINT | NO | FK → payment_receives.id |
| `invoice_id` | BIGINT | NO | FK → sale_invoices.id |
| `amount` | DECIMAL(15,5) | NO | Jumlah yang dialokasikan ke invoice ini |

---

#### Tabel: `credit_notes`

**Tujuan:** Nota kredit yang diterbitkan untuk pelanggan (retur, koreksi).

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `customer_id` | BIGINT | NO | FK → contacts.id |
| `credit_note_number` | VARCHAR | YES | Nomor nota kredit |
| `credit_note_date` | DATE | NO | Tanggal nota kredit |
| `amount` | DECIMAL(15,5) | NO | Total nilai nota kredit |
| `invoices_amount` | DECIMAL(15,5) | NO | Total yang sudah diterapkan ke invoice |
| `refunded_amount` | DECIMAL(15,5) | NO | Total yang sudah di-refund |
| `currency_code` | VARCHAR(4) | YES | Mata uang |
| `exchange_rate` | DECIMAL(13,9) | NO | Kurs |
| `opened_at` | DATE | YES | NULL = Draft; ada isi = Open |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

**Status:** `Draft → Open → Closed` (closed saat invoices_amount + refunded_amount >= amount)

---

#### Tabel: `credit_note_applied_invoices`

**Tujuan:** Mencatat penerapan nota kredit ke invoice tertentu.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT | Primary key |
| `credit_note_id` | BIGINT | FK → credit_notes.id |
| `invoice_id` | BIGINT | FK → sale_invoices.id |
| `amount` | DECIMAL(15,5) | Jumlah yang diterapkan |

---

### KELOMPOK 5 — TABEL TRANSAKSI AP (Accounts Payable)

---

#### Tabel: `bills`

**Tujuan:** Tagihan dari vendor atas pembelian barang/jasa. Saat dibuka, menghasilkan jurnal GL otomatis.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `vendor_id` | BIGINT | NO | FK → contacts.id |
| `bill_number` | VARCHAR | YES | Nomor tagihan (unik per non-deleted) |
| `bill_date` | DATE | NO | Tanggal tagihan |
| `due_date` | DATE | YES | Tanggal jatuh tempo |
| `reference_no` | VARCHAR | YES | Nomor referensi vendor |
| `status` | VARCHAR | NO | Status: 'draft' (default) |
| `note` | TEXT | YES | Catatan |
| `amount` | DECIMAL(15,5) | NO | Total nilai tagihan |
| `currency_code` | VARCHAR(4) | YES | Mata uang |
| `exchange_rate` | DECIMAL(13,9) | NO | Kurs |
| `payment_amount` | DECIMAL(15,5) | NO | Total yang sudah dibayar |
| `landed_cost_amount` | DECIMAL(15,5) | NO | Biaya landed cost terkait |
| `allocated_cost_amount` | DECIMAL(15,5) | NO | Biaya yang sudah dialokasikan |
| `credited_amount` | DECIMAL(15,5) | NO | Total kredit vendor yang diterapkan |
| `opened_at` | DATE | YES | NULL = Draft; ada isi = Dibuka |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

**Index:** `bill_number`, `bill_date`, `due_date`, `opened_at`, `deleted_at`
**Compound Index:** `(vendor_id, deleted_at)`, `(vendor_id, deleted_at, bill_date)`
**Soft Delete:** Ya
**Audit Trail:** Ya

---

#### Tabel: `bill_payments`

**Tujuan:** Header pembayaran yang dilakukan ke vendor.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `vendor_id` | BIGINT | NO | FK → contacts.id |
| `payment_account_id` | BIGINT | NO | FK → accounts.id (Cash/Bank sumber) |
| `payment_date` | DATE | NO | Tanggal pembayaran |
| `amount` | DECIMAL(15,5) | NO | Total pembayaran |
| `currency_code` | VARCHAR(4) | YES | Mata uang |
| `exchange_rate` | DECIMAL(13,9) | NO | Kurs |
| `reference_no` | VARCHAR | YES | Nomor bukti transfer |
| `note` | TEXT | YES | Catatan |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

---

#### Tabel: `bill_payment_entries`

**Tujuan:** Alokasi pembayaran ke tagihan tertentu.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT | Primary key |
| `bill_payment_id` | BIGINT | FK → bill_payments.id |
| `bill_id` | BIGINT | FK → bills.id |
| `amount` | DECIMAL(15,5) | Jumlah yang dialokasikan ke tagihan ini |

---

#### Tabel: `vendor_credits`

**Tujuan:** Kredit dari vendor (retur pembelian, koreksi tagihan).

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `vendor_id` | BIGINT | NO | FK → contacts.id |
| `vendor_credit_number` | VARCHAR | YES | Nomor kredit vendor |
| `vendor_credit_date` | DATE | NO | Tanggal kredit vendor |
| `amount` | DECIMAL(15,5) | NO | Total nilai kredit |
| `bills_amount` | DECIMAL(15,5) | NO | Total yang sudah diterapkan ke tagihan |
| `refunded_amount` | DECIMAL(15,5) | NO | Total yang sudah di-refund |
| `opened_at` | DATE | YES | NULL = Draft; ada isi = Open |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

---

#### Tabel: `vendor_credit_applied_bills`

**Tujuan:** Mencatat penerapan kredit vendor ke tagihan tertentu.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT | Primary key |
| `vendor_credit_id` | BIGINT | FK → vendor_credits.id |
| `bill_id` | BIGINT | FK → bills.id |
| `amount` | DECIMAL(15,5) | Jumlah yang diterapkan |

---

### KELOMPOK 6 — TABEL AKUNTANSI

---

#### Tabel: `item_entries`

**Tujuan:** Baris detail dari invoice, tagihan, estimasi, dan nota kredit (polymorphic).

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `reference_type` | VARCHAR | NO | Tipe dokumen: 'SaleInvoice', 'Bill', dll. |
| `reference_id` | BIGINT | NO | ID dokumen referensi |
| `item_id` | BIGINT | YES | FK → items.id |
| `description` | TEXT | YES | Deskripsi baris item |
| `quantity` | DECIMAL(15,5) | NO | Jumlah |
| `unit` | VARCHAR | YES | Satuan (pcs, kg, dll.) |
| `rate` | DECIMAL(15,5) | NO | Harga per unit |
| `amount` | DECIMAL(15,5) | NO | Total = quantity × rate |
| `tax_rate_id` | BIGINT | YES | FK → tax_rates.id |
| `tax_amount` | DECIMAL(15,5) | NO | Jumlah pajak |
| `account_id` | BIGINT | YES | FK → accounts.id (akun pendapatan/beban) |
| `discount` | DECIMAL(10,2) | YES | Diskon per baris |
| `discount_type` | VARCHAR | YES | Tipe diskon |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |

**Index:** `(reference_type, reference_id)` — polymorphic lookup

---

#### Tabel: `account_transactions`

**Tujuan:** Buku Besar Umum (General Ledger). Inti dari sistem double-entry accounting. Setiap baris = satu sisi (debit atau kredit) dari satu jurnal.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `account_id` | BIGINT | NO | FK → accounts.id (akun yang terpengaruh) |
| `debit` | DECIMAL(15,5) | NO | Jumlah debit (0 jika ini sisi kredit) |
| `credit` | DECIMAL(15,5) | NO | Jumlah kredit (0 jika ini sisi debit) |
| `currency_code` | VARCHAR(4) | YES | Mata uang dokumen sumber |
| `exchange_rate` | DECIMAL(13,9) | NO | Kurs saat transaksi |
| `transaction_type` | VARCHAR | NO | Tipe transaksi (lihat enum TransactionType) |
| `reference_type` | VARCHAR | YES | Tipe dokumen sumber: 'SaleInvoice', 'Bill', dll. |
| `reference_id` | BIGINT | YES | ID dokumen sumber |
| `contact_type` | VARCHAR | YES | 'customer' atau 'vendor' |
| `contact_id` | BIGINT | YES | FK ke contacts |
| `transaction_number` | VARCHAR | YES | Nomor dokumen (untuk tampilan GL) |
| `reference_number` | VARCHAR | YES | Nomor referensi |
| `item_id` | BIGINT | YES | FK → items.id |
| `item_quantity` | INT | YES | Jumlah item terkait |
| `note` | TEXT | YES | Catatan |
| `user_id` | BIGINT | YES | FK → users.id (pembuat) |
| `index_group` | INT | YES | Pengelompokan jurnal terkait |
| `index` | INT | NO | Urutan dalam kelompok |
| `date` | DATE | NO | Tanggal transaksi |
| `is_costable` | BOOLEAN | NO | Dapat dihitung sebagai HPP? |
| `branch_id` | BIGINT | YES | FK → branches.id |
| `project_id` | BIGINT | YES | FK → projects.id |
| `tax_rate_id` | BIGINT | YES | FK → tax_rates.id |
| `tax_rate` | DECIMAL(8,4) | NO | Persentase pajak |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |

**Index:** `transaction_type`, `reference_type`, `reference_id`, `account_id`, `contact_id`, `contact_type`, `date`, `created_at`
**Compound Index:** `(reference_type, reference_id)`, `(account_id, date)`
**Soft Delete:** TIDAK — GL entries tidak boleh di-soft-delete karena GL reversal menggunakan DELETE fisik
**Audit Trail:** Ya

---

#### Tabel: `manual_journals`

**Tujuan:** Header jurnal manual yang dibuat oleh pengguna.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `journal_number` | VARCHAR | YES | Nomor jurnal (unik per non-deleted) |
| `reference` | VARCHAR | YES | Referensi eksternal |
| `journal_type` | VARCHAR | YES | Tipe jurnal |
| `amount` | DECIMAL(15,5) | NO | Total (= jumlah kredit = jumlah debit) |
| `currency_code` | VARCHAR(4) | YES | Mata uang |
| `exchange_rate` | DECIMAL(13,9) | NO | Kurs |
| `date` | DATE | NO | Tanggal jurnal |
| `description` | TEXT | YES | Deskripsi jurnal |
| `published_at` | DATE | YES | NULL = Draft; ada isi = Published |
| `user_id` | BIGINT | YES | FK → users.id |
| `branch_id` | BIGINT | YES | FK → branches.id |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

**Compound Index:** `(deleted_at, date)`
**Soft Delete:** Ya
**Audit Trail:** Ya

---

#### Tabel: `manual_journal_entries`

**Tujuan:** Baris debit/kredit dari jurnal manual.

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `manual_journal_id` | BIGINT | NO | FK → manual_journals.id |
| `account_id` | BIGINT | NO | FK → accounts.id |
| `debit` | DECIMAL(15,5) | NO | Jumlah debit (0 jika kredit) |
| `credit` | DECIMAL(15,5) | NO | Jumlah kredit (0 jika debit) |
| `description` | TEXT | YES | Deskripsi baris |
| `contact_id` | BIGINT | YES | FK → contacts.id (opsional) |
| `tax_rate_id` | BIGINT | YES | FK → tax_rates.id (opsional) |
| `branch_id` | BIGINT | YES | FK → branches.id |
| `project_id` | BIGINT | YES | FK ke projects |

---

#### Tabel: `expenses`

**Tujuan:** Pengeluaran langsung yang dibayar dari kas/bank (bukan via tagihan vendor).

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `payment_account_id` | BIGINT | NO | FK → accounts.id (Cash/Bank sumber) |
| `payee_id` | BIGINT | YES | FK → contacts.id (penerima) |
| `reference_no` | VARCHAR | YES | Nomor referensi |
| `total_amount` | DECIMAL(15,5) | NO | Total pengeluaran |
| `currency_code` | VARCHAR(4) | YES | Mata uang |
| `exchange_rate` | DECIMAL(13,9) | NO | Kurs |
| `description` | TEXT | YES | Keterangan pengeluaran |
| `payment_date` | DATE | NO | Tanggal pembayaran |
| `published_at` | DATE | YES | NULL = Draft; ada isi = Published |
| `user_id` | BIGINT | YES | FK → users.id |
| `branch_id` | BIGINT | YES | FK → branches.id |
| `project_id` | BIGINT | YES | FK ke projects |
| `landed_cost_amount` | DECIMAL(15,5) | NO | Landed cost terkait |
| `allocated_cost_amount` | DECIMAL(15,5) | NO | Biaya yang dialokasikan |
| `created_by` | BIGINT | YES | Audit trail |
| `updated_by` | BIGINT | YES | Audit trail |
| `deleted_by` | BIGINT | YES | Audit trail |
| `deleted_at` | TIMESTAMP | YES | Soft delete |

**Compound Index:** `(deleted_at, payment_date)`
**Soft Delete:** Ya
**Audit Trail:** Ya

---

#### Tabel: `expense_categories`

**Tujuan:** Baris kategori beban dalam satu pengeluaran (satu expense bisa multi-kategori).

| Kolom | Tipe | Nullable | Keterangan |
|-------|------|----------|------------|
| `id` | BIGINT | NO | Primary key |
| `expense_id` | BIGINT | NO | FK → expenses.id |
| `account_id` | BIGINT | NO | FK → accounts.id (akun beban) |
| `amount` | DECIMAL(15,5) | NO | Jumlah kategori ini |
| `description` | TEXT | YES | Keterangan |
| `tax_rate_id` | BIGINT | YES | FK → tax_rates.id |

---

## BAGIAN 3 — RELASI ANTAR TABEL

Daftar lengkap foreign key relationship:

| Tabel (FK) | Kolom | Referensi | Tipe Relasi |
|------------|-------|-----------|-------------|
| `exchange_rates` | `currency_id` | `currencies.id` | ManyToOne |
| `accounts` | `parent_account_id` | `accounts.id` | ManyToOne (self) |
| `contacts` | `opening_balance_branch_id` | `branches.id` | ManyToOne |
| `items` | `sell_account_id` | `accounts.id` | ManyToOne |
| `items` | `cost_account_id` | `accounts.id` | ManyToOne |
| `items` | `inventory_account_id` | `accounts.id` | ManyToOne |
| `items` | `sell_tax_rate_id` | `tax_rates.id` | ManyToOne |
| `items` | `purchase_tax_rate_id` | `tax_rates.id` | ManyToOne |
| `items` | `category_id` | `item_categories.id` | ManyToOne |
| `warehouses` | `branch_id` | `branches.id` | ManyToOne |
| `sale_invoices` | `customer_id` | `contacts.id` | ManyToOne |
| `sale_invoices` | `user_id` | `users.id` | ManyToOne |
| `sale_invoices` | `warehouse_id` | `warehouses.id` | ManyToOne |
| `sale_invoices` | `branch_id` | `branches.id` | ManyToOne |
| `sale_invoices` | `writtenoff_expense_account_id` | `accounts.id` | ManyToOne |
| `sale_estimates` | `customer_id` | `contacts.id` | ManyToOne |
| `sale_estimates` | `converted_to_invoice_id` | `sale_invoices.id` | OneToOne |
| `payment_receives` | `customer_id` | `contacts.id` | ManyToOne |
| `payment_receives` | `deposit_account_id` | `accounts.id` | ManyToOne |
| `payment_receive_entries` | `payment_receive_id` | `payment_receives.id` | ManyToOne |
| `payment_receive_entries` | `invoice_id` | `sale_invoices.id` | ManyToOne |
| `credit_notes` | `customer_id` | `contacts.id` | ManyToOne |
| `credit_note_applied_invoices` | `credit_note_id` | `credit_notes.id` | ManyToOne |
| `credit_note_applied_invoices` | `invoice_id` | `sale_invoices.id` | ManyToOne |
| `bills` | `vendor_id` | `contacts.id` | ManyToOne |
| `bills` | `user_id` | `users.id` | ManyToOne |
| `bills` | `warehouse_id` | `warehouses.id` | ManyToOne |
| `bills` | `branch_id` | `branches.id` | ManyToOne |
| `bill_payments` | `vendor_id` | `contacts.id` | ManyToOne |
| `bill_payments` | `payment_account_id` | `accounts.id` | ManyToOne |
| `bill_payment_entries` | `bill_payment_id` | `bill_payments.id` | ManyToOne |
| `bill_payment_entries` | `bill_id` | `bills.id` | ManyToOne |
| `vendor_credits` | `vendor_id` | `contacts.id` | ManyToOne |
| `vendor_credit_applied_bills` | `vendor_credit_id` | `vendor_credits.id` | ManyToOne |
| `vendor_credit_applied_bills` | `bill_id` | `bills.id` | ManyToOne |
| `item_entries` | `item_id` | `items.id` | ManyToOne |
| `item_entries` | `tax_rate_id` | `tax_rates.id` | ManyToOne |
| `item_entries` | `account_id` | `accounts.id` | ManyToOne |
| `account_transactions` | `account_id` | `accounts.id` | ManyToOne |
| `account_transactions` | `item_id` | `items.id` | ManyToOne |
| `account_transactions` | `tax_rate_id` | `tax_rates.id` | ManyToOne |
| `account_transactions` | `user_id` | `users.id` | ManyToOne |
| `account_transactions` | `branch_id` | `branches.id` | ManyToOne |
| `manual_journals` | `user_id` | `users.id` | ManyToOne |
| `manual_journals` | `branch_id` | `branches.id` | ManyToOne |
| `manual_journal_entries` | `manual_journal_id` | `manual_journals.id` | ManyToOne |
| `manual_journal_entries` | `account_id` | `accounts.id` | ManyToOne |
| `expenses` | `payment_account_id` | `accounts.id` | ManyToOne |
| `expenses` | `payee_id` | `contacts.id` | ManyToOne |
| `expenses` | `user_id` | `users.id` | ManyToOne |
| `expenses` | `branch_id` | `branches.id` | ManyToOne |
| `expense_categories` | `expense_id` | `expenses.id` | ManyToOne |
| `expense_categories` | `account_id` | `accounts.id` | ManyToOne |
| `model_has_roles` | `role_id` | `roles.id` | ManyToMany |
| `role_has_permissions` | `permission_id` | `permissions.id` | ManyToMany |
| `role_has_permissions` | `role_id` | `roles.id` | ManyToMany |

---

## BAGIAN 4 — STRATEGI INDEXING

### Prinsip Indexing

Sistem keuangan memiliki pola query yang dapat diprediksi:
1. **Filter by contact + exclude soft-deleted** → compound index `(contact_id, deleted_at)`
2. **GL balance calculation by date range** → compound index `(account_id, date)`
3. **Polymorphic GL lookup** → compound index `(reference_type, reference_id)`
4. **Sort by date** → simple index `date`, `invoice_date`, `bill_date`
5. **Status filter** → index `delivered_at`, `opened_at`, `published_at`

### Index per Tabel

| Tabel | Index | Tujuan |
|-------|-------|--------|
| `users` | `email` UNIQUE | Login lookup |
| `currencies` | `code` UNIQUE | Currency lookup |
| `accounts` | `name`, `account_type`, `code`, `is_active`, `is_predefined` | Filter bagan akun |
| `sale_invoices` | `(customer_id, deleted_at)` | Daftar invoice per pelanggan |
| `sale_invoices` | `(customer_id, deleted_at, invoice_date)` | Sama, urut tanggal |
| `sale_invoices` | `invoice_date`, `due_date`, `delivered_at` | Filter status |
| `bills` | `(vendor_id, deleted_at)` | Daftar tagihan per vendor |
| `bills` | `(vendor_id, deleted_at, bill_date)` | Sama, urut tanggal |
| `bills` | `bill_date`, `due_date`, `opened_at` | Filter status |
| `account_transactions` | `(reference_type, reference_id)` | Temukan semua GL untuk dokumen tertentu |
| `account_transactions` | `(account_id, date)` | Hitung saldo akun per periode |
| `account_transactions` | `transaction_type`, `date` | Filter GL per tipe dan periode |
| `manual_journals` | `(deleted_at, date)` | Daftar jurnal aktif urut tanggal |
| `expenses` | `(deleted_at, payment_date)` | Daftar pengeluaran aktif |
| `item_entries` | `(reference_type, reference_id)` | Baris item per dokumen |

---

## BAGIAN 5 — KONVENSI PENAMAAN

### Konvensi Nama Tabel dan Kolom

| Konvensi | Aturan | Contoh |
|----------|--------|--------|
| Nama tabel | `snake_case`, plural | `sale_invoices`, `account_transactions` |
| Nama kolom | `snake_case` | `customer_id`, `invoice_date`, `is_active` |
| Primary key | `id` (BIGINT UNSIGNED) | `id` |
| Foreign key | `{tabel_singular}_id` | `customer_id`, `account_id`, `branch_id` |
| Boolean | Prefix `is_` | `is_active`, `is_predefined`, `is_system_account` |
| Tanggal aksi | Suffix `_at` (DATE) | `delivered_at`, `opened_at`, `published_at` |
| Soft delete | `deleted_at` (TIMESTAMP) | Standard Laravel |
| Timestamps | `created_at`, `updated_at` | Standard Laravel |

### Konvensi Tipe Data Keuangan

```sql
-- NILAI UANG: decimal(15,5)
-- Mendukung nilai hingga 999,999,999,999 dengan 5 desimal
-- Digunakan di: balance, amount, payment_amount, rate, dll.
balance     DECIMAL(15,5) DEFAULT 0

-- KURS TUKAR: decimal(13,9)
-- Presisi 9 desimal untuk akurasi nilai tukar internasional
-- Digunakan di: exchange_rate di semua tabel transaksi
exchange_rate DECIMAL(13,9) DEFAULT 1

-- TARIF PAJAK: decimal(8,4)
-- Contoh: PPN 11% disimpan sebagai 11.0000
-- Digunakan di: tax_rates.rate, account_transactions.tax_rate
tax_rate    DECIMAL(8,4) DEFAULT 0

-- DISKON: decimal(10,2)
-- Nilai diskon atau persentase dengan 2 desimal
discount    DECIMAL(10,2)
```

### Konvensi Audit Columns

```sql
-- Semua 46 tabel bisnis memiliki 3 kolom audit ini:
created_by  BIGINT UNSIGNED NULL  -- User ID yang membuat
updated_by  BIGINT UNSIGNED NULL  -- User ID yang terakhir mengubah
deleted_by  BIGINT UNSIGNED NULL  -- User ID yang menghapus (jika ada soft delete)
```

Kolom `deleted_by` hanya bermakna pada tabel yang menggunakan `SoftDeletes`. Diisi otomatis oleh `Auditable` trait sebelum soft delete terjadi.

### Konvensi Soft Delete

```sql
-- Semua tabel transaksi dan master data:
deleted_at  TIMESTAMP NULL  -- Standard Laravel SoftDeletes
```

Ketika record di-soft-delete:
1. `deleted_at` diisi dengan timestamp saat ini
2. `deleted_by` diisi dengan ID user yang menghapus
3. Record tidak muncul di query default (Eloquent otomatis filter `deleted_at IS NULL`)
4. GL entries yang terkait di-revert (dihapus dari `account_transactions`)

---

## BAGIAN 6 — DATA AWAL (SEEDING)

### Seeder yang Tersedia

| Seeder | Wajib Production | Isi |
|--------|:----------------:|-----|
| `RolePermissionSeeder` | ✓ YA | 3 role + 79 permission |
| `AccountSeeder` | ✓ YA | 26 grup + 50+ akun leaf PSAK |
| `CurrencySeeder` | Opsional | 50+ mata uang dunia |
| `TaxRateSeeder` | Opsional | PPN dan PPh Indonesia |
| `BranchSeeder` | Opsional | Data cabang awal |
| `WarehouseSeeder` | Opsional | Data gudang awal |
| `SettingSeeder` | Opsional | Setting default organisasi |
| `UserAccountSeeder` | ✗ TIDAK | Akun demo — jangan di production |
| `ContactSeeder` | ✗ TIDAK | Data kontak demo |
| `ItemSeeder` | ✗ TIDAK | Data item demo |
| `TransactionSeeder` | ✗ TIDAK | Data transaksi demo |

---

### Data Seeding: RolePermissionSeeder

**3 Role yang dibuat:**

| Role | Total Permission | Deskripsi |
|------|:----------------:|-----------|
| `admin` | 79 | Akses penuh ke semua modul |
| `staff` | 44 | Create/edit/view di Finance & Accounting, view di modul lain |
| `viewer` | 20 | View only di semua modul |

**Modul dan Permission (format: `{modul}.{action}`):**

| Modul | Permission yang dibuat |
|-------|----------------------|
| `account` | create, edit, delete, view |
| `sale-invoice` | create, edit, delete, view, writeoff |
| `sale-estimate` | create, edit, delete, view |
| `payment-receive` | create, edit, delete, view |
| `bill` | create, edit, delete, view |
| `bill-payment` | create, edit, delete, view |
| `credit-note` | create, edit, delete, view |
| `vendor-credit` | create, edit, delete, view |
| `expense` | create, edit, delete, view |
| `manual-journal` | create, edit, delete, view |
| `item` | create, edit, delete, view |
| `contact` | create, edit, delete, view |
| `tax-rate` | create, edit, delete, view |
| `cashflow` | create, edit, delete, view |
| `inventory-adjustment` | create, edit, delete, view |
| `warehouse-transfer` | create, edit, delete, view |
| `project` | create, edit, delete, view |
| `report` | view |
| `setting` | edit, view |

---

### Data Seeding: AccountSeeder

**Struktur Bagan Akun (PSAK-compliant):**

| Kode | Nama | Tipe | is_predefined | is_system_account |
|------|------|------|:-------------:|:-----------------:|
| 1100 | Cash and Bank | other-current-asset | ✓ | — |
| 11001 | Cash on Hand | cash | ✓ | ✓ |
| 11002 | Petty Cash | cash | — | — |
| 11101 | Bank BCA - IDR | bank | ✓ | ✓ |
| 11102 | Bank Mandiri - IDR | bank | — | — |
| 11103 | Bank BNI - IDR | bank | — | — |
| 11104 | Bank BRI - IDR | bank | — | — |
| 11105 | Bank BCA - USD | bank | — | — |
| 1200 | Receivables | accounts-receivable | ✓ | — |
| 12001 | Accounts Receivable | accounts-receivable | ✓ | ✓ |
| 12002 | Allowance for Doubtful Accounts | accounts-receivable | — | — |
| 1300 | Inventories | inventory | ✓ | — |
| 13001 | Inventory - Finished Goods | inventory | ✓ | ✓ |
| 2100 | Accounts Payable | accounts-payable | ✓ | — |
| 21001 | Accounts Payable | accounts-payable | ✓ | ✓ |
| 2200 | Tax Payable | tax-payable | ✓ | — |
| 22001 | PPN Keluaran (Output VAT) | tax-payable | ✓ | ✓ |
| 3200 | Retained Earnings | equity | ✓ | — |
| 32001 | Retained Earnings | equity | ✓ | ✓ |
| 4100 | Sales Revenue | income | ✓ | — |
| 41001 | Sales Revenue - Products | income | ✓ | ✓ |
| 5100 | Cost of Goods Sold | cost-of-goods-sold | ✓ | — |
| 51001 | Cost of Goods Sold | cost-of-goods-sold | ✓ | ✓ |
| ... | *50+ akun lainnya* | ... | ... | ... |

**Akun sistem kritis (is_system_account = true, dilindungi dari penghapusan):**
- `11001` Cash on Hand
- `11101` Bank BCA - IDR
- `12001` Accounts Receivable
- `13001` Inventory - Finished Goods
- `21001` Accounts Payable
- `22001` PPN Keluaran (Output VAT)
- `32001` Retained Earnings
- `41001` Sales Revenue - Products
- `51001` Cost of Goods Sold

---

### Data Seeding: CurrencySeeder

50+ mata uang dari semua kawasan dunia, termasuk:

| Kawasan | Mata Uang |
|---------|-----------|
| Asia Tenggara | IDR, SGD, MYR, THB, PHP, VND |
| Asia Timur | JPY, CNY, KRW, HKD, TWD |
| Asia Selatan | INR, PKR, BDT, LKR |
| Americas | USD, CAD, MXN, BRL, ARS |
| Eropa | EUR, GBP, CHF, SEK, NOK, DKK, PLN, TRY |
| Timur Tengah | SAR, AED, QAR, KWD, BHD |
| Oseania | AUD, NZD |
| Afrika | ZAR, NGN, EGP |

---

*Dokumentasi database ini mencerminkan schema per migration terakhir.*
*Untuk schema yang paling akurat, selalu rujuk file migration di `database/migrations/`.*
