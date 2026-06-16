# Invoices — Faktur Penjualan

Prefix: `/api/v1/invoices` · Autentikasi: Bearer token · Lihat [00-panduan-umum.md](00-panduan-umum.md).

Faktur penjualan adalah tagihan resmi ke pelanggan. Faktur dibuat sebagai **draft**, lalu **diterbitkan (deliver)** untuk membentuk jurnal piutang. Lihat penjelasan bisnis di `docs/presentation/02-penjualan.md`.

---

## Daftar Endpoint

| Method | URL | Deskripsi | Permission |
| --- | --- | --- | --- |
| GET | `/api/v1/invoices` | Daftar faktur (terpaginasi) | `sale-invoice.view` |
| GET | `/api/v1/invoices/{id}` | Detail faktur (atau PDF) | `sale-invoice.view` |
| POST | `/api/v1/invoices` | Buat faktur baru (draft) | `sale-invoice.create` |
| PUT/PATCH | `/api/v1/invoices/{id}` | Ubah faktur | `sale-invoice.edit` |
| POST | `/api/v1/invoices/{id}/deliver` | Terbitkan faktur (buat jurnal) | `sale-invoice.edit` |
| GET | `/api/v1/invoices/{id}/mail` | Lihat status/draft email | `sale-invoice.edit` |
| POST | `/api/v1/invoices/{id}/mail` | Kirim faktur via email | `sale-invoice.edit` |
| POST | `/api/v1/invoices/validate-delete` | Validasi penghapusan massal | `sale-invoice.delete` |
| POST | `/api/v1/invoices/bulk-delete` | Hapus massal | `sale-invoice.delete` |
| DELETE | `/api/v1/invoices/{id}` | Hapus satu faktur | `sale-invoice.delete` |

**Status valid (`status`):** `draft`, `unpaid`, `partially-paid`, `paid`, `overdue`, `written-off`.

---

## GET /api/v1/invoices — Daftar Faktur

### Request

```
GET /api/v1/invoices?status=unpaid&per_page=10&page=1
Authorization: Bearer {token}
Accept: application/json
```

**Query Parameters (opsional):**

| Parameter | Tipe | Default | Keterangan |
| --- | --- | --- | --- |
| `page` | integer | `1` | Halaman ke- |
| `per_page` | integer | `10` | 5 / 10 / 20 / 50 / 100 |
| `search` | string | – | Cari nomor faktur, reference_no, atau nama pelanggan |
| `status` | string | – | `draft` / `delivered` / `unpaid` / `overdue` (filter), juga `paid` / `partiallypaid` / `writtenoff` |
| `customer_id` | integer | – | Filter per pelanggan |
| `from_date` | date | – | Tanggal faktur ≥ (YYYY-MM-DD) |
| `to_date` | date | – | Tanggal faktur ≤ (YYYY-MM-DD) |

### Response 200

```json
{
  "data": [
    {
      "id": 1,
      "document_number": "INV-000001",
      "status": "unpaid",
      "date": "2026-06-01",
      "due_date": "2026-07-01",
      "customer": { "id": 5, "display_name": "PT Maju Bersama", "email": "po@maju.co.id" },
      "currency": { "code": "IDR", "exchange_rate": "1.00000000" },
      "subtotal": "12000000.00000",
      "tax_total": "1320000.00000",
      "total": "13320000.00000",
      "balance_due": "13320000.00000",
      "amount_due": "13320000.00000",
      "payment_amount": "0.00000",
      "credited_amount": "0.00000",
      "project": null,
      "lines": [
        {
          "id": 10,
          "index": 1,
          "item": { "id": 3, "name": "Jasa Konsultasi IT", "code": "SVC-001" },
          "description": "Konsultasi implementasi",
          "quantity": "2.00000",
          "rate": "6000000.00000",
          "discount": "0.00000",
          "discount_type": "percentage",
          "tax_rate": "11.0000",
          "is_inclusive_tax": false,
          "amount": "12000000.00000"
        }
      ],
      "created_at": "2026-06-01T08:00:00+00:00",
      "updated_at": "2026-06-01T08:00:00+00:00"
    }
  ],
  "links": { "first": "...", "last": "...", "prev": null, "next": "...?page=2" },
  "meta": { "current_page": 1, "per_page": 10, "total": 25, "last_page": 3 }
}
```

**Penjelasan Field:**

| Field | Tipe | Keterangan |
| --- | --- | --- |
| `id` | integer | ID unik faktur |
| `document_number` | string | Nomor faktur (format `INV-XXXXXX`) |
| `status` | string | `draft` / `unpaid` / `partially-paid` / `paid` / `overdue` / `written-off` |
| `date` | date | Tanggal faktur |
| `due_date` | date | Jatuh tempo |
| `customer` | object\|null | `{id, display_name, email}` |
| `currency` | object | `{code, exchange_rate}` |
| `subtotal` | string | Total baris setelah diskon, sebelum pajak |
| `tax_total` | string | Total pajak |
| `total` | string | Total faktur (= kolom `balance`, dalam mata uang dokumen) |
| `balance_due` / `amount_due` | string | Sisa tagihan = total − payment_amount − credited_amount |
| `payment_amount` | string | Total sudah dibayar |
| `credited_amount` | string | Total nota kredit yang diterapkan |
| `project` | object\|null | `{id, name}` bila ada |
| `lines` | array | Baris item (lihat di bawah) |

**Field per baris (`lines[]`):** `id`, `index`, `item` (`{id,name,code}`|null), `description`, `quantity`, `rate`, `discount`, `discount_type`, `tax_rate`, `is_inclusive_tax`, `amount` (net setelah diskon).

---

## GET /api/v1/invoices/{id} — Detail Faktur

### Request
```
GET /api/v1/invoices/1
Authorization: Bearer {token}
Accept: application/json
```

### Response 200
```json
{ "data": { "id": 1, "document_number": "INV-000001", "status": "unpaid", "...": "..." } }
```
Struktur objek sama dengan elemen `data[]` pada daftar.

### Mengunduh PDF
Tambahkan `?format=pdf` atau header `Accept: application/pdf`:
```
GET /api/v1/invoices/1?format=pdf
Authorization: Bearer {token}
Accept: application/pdf
```
Tambahkan `?disposition=attachment` untuk memaksa unduh (default `inline`). Response berupa berkas PDF, bukan JSON.

### Response 404
```json
{ "message": "No query results for model [App\\Models\\SaleInvoice] 999" }
```

---

## POST /api/v1/invoices — Buat Faktur Baru

Faktur baru selalu dibuat berstatus **draft** (belum ada jurnal).

### Request Body
```json
{
  "customer_id": 5,
  "invoice_no": "INV-000042",
  "invoice_date": "2026-06-17",
  "due_date": "2026-07-17",
  "currency_code": "IDR",
  "exchange_rate": 1.0,
  "reference_no": "PO-2026-001",
  "invoice_message": "Terima kasih atas kepercayaan Anda.",
  "terms_conditions": "Pembayaran dalam 30 hari.",
  "is_inclusive_tax": false,
  "discount": 0,
  "discount_type": "percentage",
  "adjustment": 0,
  "branch_id": 1,
  "project_id": null,
  "entries": [
    {
      "item_id": 3,
      "description": "Jasa Konsultasi IT",
      "quantity": 2,
      "rate": 6000000,
      "discount": 0,
      "discount_type": "percentage",
      "sell_account_id": 12,
      "tax_rate_id": 1,
      "tax_rate": 11
    }
  ]
}
```

**Keterangan Field Request:**

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `customer_id` | ✓ | integer | ID pelanggan (`exists:contacts`) |
| `invoice_no` | ✓ | string | **Wajib di API** (maks 50, unik). Pada web auto-generate; di API harus diisi |
| `invoice_date` | ✓ | date | Tanggal faktur |
| `due_date` | ✓ | date | Harus ≥ `invoice_date` |
| `reference_no` | – | string | Maks 50 |
| `invoice_message` | – | string | Maks 2000 |
| `terms_conditions` | – | string | Maks 2000 |
| `currency_code` | – | string | `exists:currencies,code` (default mata uang dasar) |
| `exchange_rate` | – | numeric | > 0 |
| `is_inclusive_tax` | – | boolean | Pajak termasuk harga? |
| `discount` | – | numeric | ≥ 0 (diskon header) |
| `discount_type` | – | string | `percentage` / `amount` |
| `adjustment` | – | numeric | Penyesuaian nominal |
| `warehouse_id` | – | integer | `exists:warehouses` |
| `branch_id` | – | integer | `exists:branches` |
| `project_id` | – | integer | `exists:projects` (jika fitur Proyek aktif) |
| `entries` | ✓ | array | Min 1, maks 500 baris |
| `entries[].item_id` | – | integer | Boleh kosong (baris bebas) |
| `entries[].description` | – | string | Maks 1000 |
| `entries[].quantity` | ✓ | numeric | Harus > 0 |
| `entries[].rate` | ✓ | numeric | ≥ 0 |
| `entries[].discount` | – | numeric | ≥ 0 |
| `entries[].discount_type` | – | string | `percentage` / `amount` |
| `entries[].sell_account_id` | – | integer | Akun pendapatan |
| `entries[].tax_rate_id` | – | integer | ID tarif pajak |
| `entries[].tax_rate` | – | numeric | ≥ 0 (snapshot persen pajak) |

> `discount_type` mengikuti enum `DiscountType`: nilai valid **`percentage`** dan **`amount`**.

### Response 201 — Berhasil Dibuat
```json
{
  "data": {
    "id": 42,
    "document_number": "INV-000042",
    "status": "draft",
    "date": "2026-06-17",
    "due_date": "2026-07-17",
    "total": "13320000.00000",
    "balance_due": "13320000.00000",
    "lines": [ { "...": "..." } ],
    "...": "..."
  }
}
```

### Response 422 — Validasi Gagal
```json
{
  "message": "The customer field is required. (and 2 more errors)",
  "errors": {
    "customer_id": ["The customer field is required."],
    "invoice_no": ["The invoice no field is required."],
    "entries": ["The entries field is required."]
  }
}
```

### Response 422 — Error Bisnis (mis. nomor duplikat)
```json
{ "message": "Invoice number 'INV-000042' already exists." }
```

---

## PUT /api/v1/invoices/{id} — Ubah Faktur

Mengirim `entries` akan mengganti seluruh baris item. Bila faktur sudah diterbitkan, jurnal GL otomatis ditulis ulang. Total tidak boleh lebih kecil dari jumlah yang sudah dibayar.

### Request Body
Field sama dengan POST (kecuali `invoice_no` tidak diwajibkan pada update). Contoh ubah kuantitas:
```json
{
  "customer_id": 5,
  "invoice_date": "2026-06-17",
  "due_date": "2026-07-17",
  "entries": [
    { "item_id": 3, "quantity": 3, "rate": 6000000, "tax_rate_id": 1, "tax_rate": 11 }
  ]
}
```

### Response 200
```json
{ "data": { "id": 42, "status": "draft", "total": "19980000.00000", "...": "..." } }
```

### Response 422 — Total di bawah yang sudah dibayar
```json
{ "message": "Invoice total cannot be reduced below the already-paid amount." }
```

---

## POST /api/v1/invoices/{id}/deliver — Terbitkan Faktur

Mengubah faktur dari **draft** menjadi terbit & **membuat jurnal GL** (Debit Piutang, Kredit Pendapatan + Pajak). Dicek terhadap periode terkunci.

### Request
```
POST /api/v1/invoices/42/deliver
Authorization: Bearer {token}
Accept: application/json
```

### Response 200
```json
{ "data": { "id": 42, "status": "unpaid", "...": "..." } }
```

### Response 422 — Sudah diterbitkan
```json
{ "message": "Invoice has already been delivered." }
```

### Response 422 — Periode terkunci
```json
{ "message": "Transaction date 2026-05-31 is in a locked period (locked through 2026-05-31). Please unlock the period in Settings → Accounting before making changes." }
```

### Response 422 — Akun wajib belum ada
```json
{ "message": "The required Piutang Usaha account (type: Accounts Receivable) was not found in the Chart of Accounts. ..." }
```

---

## GET /api/v1/invoices/{id}/mail — Status Email

Mengembalikan draft email (penerima, subjek, pesan) untuk faktur.

### Response 200 (contoh)
```json
{
  "to": ["po@maju.co.id"],
  "cc": [],
  "bcc": [],
  "subject": "Invoice INV-000042 from PT ERP Nusantara",
  "message": "Dear PT Maju Bersama, ...",
  "attach_pdf": true
}
```

---

## POST /api/v1/invoices/{id}/mail — Kirim Email

### Request Body (semua opsional; default diambil dari status email)
```json
{
  "to": ["po@maju.co.id"],
  "cc": [],
  "bcc": [],
  "subject": "Invoice INV-000042",
  "message": "Mohon diproses pembayarannya.",
  "attach_pdf": true
}
```

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `to` | – | array | Min 1, maks 20 penerima |
| `cc` | – | array | Maks 20 |
| `bcc` | – | array | Maks 20 |
| `subject` | – | string | Maks 255 (wajib jika dikirim) |
| `message` | – | string | Maks 5000 (wajib jika dikirim) |
| `attach_pdf` | – | boolean | Lampirkan PDF |

### Response 200
```json
{
  "message": "Invoice email sent.",
  "data": { "id": 42, "document_number": "INV-000042", "...": "..." }
}
```

### Response 500 — Gagal kirim
```json
{ "message": "Unable to send invoice email." }
```

---

## POST /api/v1/invoices/validate-delete — Validasi Hapus Massal

Memeriksa ID mana yang aman dihapus sebelum benar-benar menghapus.

### Request Body
```json
{ "ids": [42, 43, 44] }
```

### Response 200
```json
{
  "deletable": [42, 44],
  "blocked": [
    { "id": 43, "message": "Invoice cannot be deleted because it has associated payments." }
  ]
}
```

---

## POST /api/v1/invoices/bulk-delete — Hapus Massal

### Request Body
```json
{ "ids": [42, 44] }
```

### Response 200 — Semua terhapus
```json
{ "deleted": [42, 44] }
```

### Response 422 — Ada yang terblokir
```json
{
  "message": "One or more documents cannot be deleted.",
  "deletable": [42],
  "blocked": [ { "id": 43, "message": "Invoice cannot be deleted because it has associated payments." } ]
}
```

---

## DELETE /api/v1/invoices/{id} — Hapus Satu Faktur

Faktur dengan pembayaran atau aplikasi nota kredit tidak bisa dihapus.

### Response 204
Tanpa body.

### Response 422 — Tidak bisa dihapus
```json
{ "message": "Invoice cannot be deleted because it has associated payments." }
```
