# Payment Receives — Penerimaan Pembayaran

Prefix: `/api/v1/payment-receives` · Autentikasi: Bearer token · Lihat [00-panduan-umum.md](00-panduan-umum.md).

Penerimaan Pembayaran mencatat **pelunasan faktur** pelanggan. Satu pembayaran bisa dialokasikan ke beberapa faktur. Efek: Kas/Bank naik, Piutang turun. Lihat `docs/presentation/02-penjualan.md`.

---

## Daftar Endpoint

| Method | URL | Deskripsi | Permission |
| --- | --- | --- | --- |
| GET | `/api/v1/payment-receives` | Daftar pembayaran | `payment-receive.view` |
| GET | `/api/v1/payment-receives/{id}` | Detail (atau PDF) | `payment-receive.view` |
| POST | `/api/v1/payment-receives` | Buat pembayaran | `payment-receive.create` |
| PUT/PATCH | `/api/v1/payment-receives/{id}` | Ubah pembayaran | `payment-receive.edit` |
| GET | `/api/v1/payment-receives/{id}/mail` | Status/draft email | `payment-receive.edit` |
| POST | `/api/v1/payment-receives/{id}/mail` | Kirim via email | `payment-receive.edit` |
| POST | `/api/v1/payment-receives/validate-delete` | Validasi hapus massal | `payment-receive.delete` |
| POST | `/api/v1/payment-receives/bulk-delete` | Hapus massal | `payment-receive.delete` |
| DELETE | `/api/v1/payment-receives/{id}` | Hapus satu | `payment-receive.delete` |

**Status (`status`):** selalu `received` (pembayaran langsung diakui saat dibuat).

---

## GET /api/v1/payment-receives — Daftar

**Query Parameters:** `page`, `per_page` (5/10/20/50/100), `search`, `customer_id`, `from_date`, `to_date`.

### Response 200
```json
{
  "data": [
    {
      "id": 1,
      "document_number": "PR-00001",
      "status": "received",
      "date": "2026-06-20",
      "customer": { "id": 5, "display_name": "PT Maju Bersama", "email": "po@maju.co.id" },
      "currency": { "code": "IDR", "exchange_rate": "1.00000000" },
      "subtotal": "13320000.00000",
      "tax_total": "0.00000",
      "total": "13320000.00000",
      "amount_due": null,
      "deposit_account": { "id": 2, "name": "Bank BCA", "code": "1-1200" },
      "project": null,
      "project_summary": { "projects": [], "has_no_project": true },
      "lines": [
        {
          "id": 3, "index": 1,
          "invoice": { "id": 42, "document_number": "INV-000042", "project": null },
          "amount": "13320000.00000"
        }
      ],
      "created_at": "2026-06-20T10:00:00+00:00",
      "updated_at": "2026-06-20T10:00:00+00:00"
    }
  ],
  "links": { "first": "...", "last": "...", "prev": null, "next": null },
  "meta": { "current_page": 1, "per_page": 10, "total": 1, "last_page": 1 }
}
```

**Penjelasan Field:**

| Field | Tipe | Keterangan |
| --- | --- | --- |
| `document_number` | string | Nomor pembayaran (`PR-XXXXX`) |
| `status` | string | Selalu `received` |
| `total` / `subtotal` | string | Jumlah pembayaran (= kolom `amount`); `tax_total` selalu `0` |
| `deposit_account` | object | Akun Kas/Bank tujuan |
| `project_summary` | object | `{projects: [...], has_no_project: bool}` ringkasan proyek dari faktur yang dilunasi |
| `lines[]` | array | Alokasi per faktur: `{id, index, invoice: {id, document_number, project}, amount}` |

---

## GET /api/v1/payment-receives/{id} — Detail

`{ "data": { ... } }`. Tambahkan `?format=pdf` untuk PDF. 404 bila tidak ditemukan.

---

## POST /api/v1/payment-receives — Buat Pembayaran

### Request Body
```json
{
  "customer_id": 5,
  "payment_date": "2026-06-20",
  "amount": 13320000,
  "deposit_account_id": 2,
  "currency_code": "IDR",
  "exchange_rate": 1.0,
  "payment_receive_no": "PR-00042",
  "reference_no": "TRF-998",
  "statement": "Transfer BCA",
  "branch_id": 1,
  "entries": [
    { "invoice_id": 42, "payment_amount": 13320000 }
  ]
}
```

**Keterangan Field Request:**

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `customer_id` | ✓ | integer | `exists:contacts` |
| `payment_date` | ✓ | date | Tanggal pembayaran |
| `amount` | ✓ | numeric | > 0. Harus sama dengan total `entries[].payment_amount` |
| `deposit_account_id` | ✓ | integer | Akun Kas/Bank tujuan |
| `currency_code` | – | string | `exists:currencies,code` |
| `exchange_rate` | – | numeric | > 0 |
| `payment_receive_no` | – | string | Maks 50 (auto bila kosong) |
| `reference_no` | – | string | Maks 50 |
| `statement` | – | string | Maks 2000 |
| `branch_id` | – | integer | `exists:branches` |
| `entries` | ✓ | array | Min 1, maks 100 |
| `entries[].invoice_id` | ✓ | integer | `exists:sale_invoices` (faktur milik pelanggan ini) |
| `entries[].payment_amount` | ✓ | numeric | > 0 |

> **Field yang ditolak (prohibited) di API:** `project_id`, `payment_amount` (level atas), `user_id`, `created_by`, `updated_by`, `deleted_by`, dan `entries.*.project_id`. Mengirimnya menghasilkan error 422.

### Response 201
```json
{ "data": { "id": 42, "document_number": "PR-00042", "status": "received", "total": "13320000.00000", "...": "..." } }
```

### Response 422 — Validasi
```json
{
  "message": "The amount field must be greater than 0.",
  "errors": {
    "amount": ["The amount field must be greater than 0."],
    "entries": ["The entries field is required."]
  }
}
```

### Response 422 — Jumlah tidak konsisten
Bila `amount` ≠ total alokasi `entries`:
```json
{ "message": "The payment amount does not match the total of allocated entries." }
```

---

## PUT /api/v1/payment-receives/{id} — Ubah

Field sama dengan POST. Sistem mengunci & menghitung ulang saldo faktur terkait. Response 200 berisi objek terbaru.

---

## Email, Validasi Hapus, Hapus Massal, Hapus

Pola **identik** dengan invoices (lihat [03-sales-invoices.md](03-sales-invoices.md)):

- `GET /{id}/mail`, `POST /{id}/mail` → `{ "message": "Payment receive email sent.", "data": {...} }`.
- `POST /validate-delete`, `POST /bulk-delete` → `{ "ids": [...] }`.
- `DELETE /{id}` → 204 (faktur terkait dihitung ulang otomatis).
