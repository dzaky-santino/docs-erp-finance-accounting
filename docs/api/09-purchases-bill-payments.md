# Bill Payments — Pembayaran Tagihan

Prefix: `/api/v1/bill-payments` · Autentikasi: Bearer token · Lihat [00-panduan-umum.md](00-panduan-umum.md).

Pembayaran Tagihan melunasi tagihan vendor. Satu pembayaran bisa dialokasikan ke beberapa tagihan. Efek: Kas/Bank turun, Hutang turun. Lihat `docs/presentation/03-pembelian.md`.

---

## Daftar Endpoint

| Method | URL | Deskripsi | Permission |
| --- | --- | --- | --- |
| GET | `/api/v1/bill-payments` | Daftar pembayaran | `bill-payment.view` |
| GET | `/api/v1/bill-payments/{id}` | Detail (atau PDF) | `bill-payment.view` |
| POST | `/api/v1/bill-payments` | Buat pembayaran | `bill-payment.create` |
| PUT/PATCH | `/api/v1/bill-payments/{id}` | Ubah pembayaran | `bill-payment.edit` |
| GET | `/api/v1/bill-payments/{id}/mail` | Status/draft email | `bill-payment.edit` |
| POST | `/api/v1/bill-payments/{id}/mail` | Kirim via email | `bill-payment.edit` |
| GET | `/api/v1/bill-payments/new-page` | Data bantu form buat | `bill-payment.create` |
| GET | `/api/v1/bill-payments/payment-bills` | Daftar tagihan untuk dibayar | `bill-payment.create` \| `bill-payment.edit` |
| GET | `/api/v1/bill-payments/{id}/edit-page` | Data bantu form ubah | `bill-payment.edit` |
| POST | `/api/v1/bill-payments/validate-delete` | Validasi hapus massal | `bill-payment.delete` |
| POST | `/api/v1/bill-payments/bulk-delete` | Hapus massal | `bill-payment.delete` |
| DELETE | `/api/v1/bill-payments/{id}` | Hapus satu | `bill-payment.delete` |

**Status (`status`):** selalu `paid`.

---

## GET /api/v1/bill-payments — Daftar

**Query Parameters:** `page`, `per_page` (5/10/20/50/100), `search`, `vendor_id`, `from_date`, `to_date`.

### Response 200
```json
{
  "data": [
    {
      "id": 1,
      "document_number": "BPAY-00001",
      "status": "paid",
      "date": "2026-06-25",
      "payment_date": "2026-06-25",
      "payment_number": "BPAY-00001",
      "payment_method": "transfer",
      "reference": "TRF-555",
      "statement": "Pelunasan BILL-000001",
      "vendor": { "id": 9, "display_name": "PT Sumber Makmur", "email": "sales@sumber.co.id" },
      "currency": { "code": "IDR", "exchange_rate": "1.00000000" },
      "subtotal": "22200000.00000",
      "tax_total": "0.00000",
      "total": "22200000.00000",
      "amount_due": null,
      "payment_account": { "id": 2, "name": "Bank BCA", "code": "1-1200" },
      "project": null,
      "project_summary": { "projects": [], "has_no_project": true },
      "lines": [
        { "id": 4, "index": 1, "bill": { "id": 1, "document_number": "BILL-000001", "project": null }, "amount": "22200000.00000" }
      ],
      "created_at": "2026-06-25T10:00:00+00:00",
      "updated_at": "2026-06-25T10:00:00+00:00"
    }
  ],
  "links": { "first": "...", "last": "...", "prev": null, "next": null },
  "meta": { "current_page": 1, "per_page": 10, "total": 1, "last_page": 1 }
}
```

**Field khas:** `document_number` (`BPAY-XXXXX`), `payment_method`, `reference`, `statement`, `payment_account` (akun Kas/Bank sumber), `lines[]` berisi alokasi per tagihan (`bill: {id, document_number, project}`).

---

## POST /api/v1/bill-payments — Buat Pembayaran

### Request Body
```json
{
  "vendor_id": 9,
  "payment_date": "2026-06-25",
  "amount": 22200000,
  "payment_account_id": 2,
  "currency_code": "IDR",
  "exchange_rate": 1.0,
  "payment_number": "BPAY-00042",
  "payment_method": "transfer",
  "reference": "TRF-555",
  "statement": "Pelunasan tagihan",
  "branch_id": 1,
  "entries": [
    { "bill_id": 1, "payment_amount": 22200000 }
  ]
}
```

**Keterangan Field Request:**

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `vendor_id` | ✓ | integer | `exists:contacts` |
| `payment_date` | ✓ | date | Tanggal pembayaran |
| `amount` | ✓ | numeric | > 0. Harus sama dengan total `entries[].payment_amount` |
| `payment_account_id` | ✓ | integer | Akun Kas/Bank sumber dana |
| `currency_code` | – | string | `exists:currencies,code` |
| `exchange_rate` | – | numeric | > 0 |
| `payment_number` | – | string | Maks 50 (auto bila kosong) |
| `payment_method` | – | string | Maks 50 |
| `reference` | – | string | Maks 50 |
| `statement` | – | string | Maks 2000 |
| `branch_id` | – | integer | `exists:branches` |
| `entries` | ✓ | array | Min 1, maks 100 |
| `entries[].bill_id` | ✓ | integer | `exists:bills` (tagihan milik vendor ini) |
| `entries[].payment_amount` | ✓ | numeric | > 0 |

> **Field internal yang ditolak (prohibited):** `status`, `balance`, `balance_due`, `amount_due`, `payment_amount` (level atas), `paid_amount`, `remaining_amount`, `project_id`, kolom audit, dan beberapa field internal pada `entries.*` (`id`, `bill_payment_id`, `project_id`, `reference_type`, `reference_id`, `index`, kolom audit). Kirim hanya field di tabel di atas.

### Response 201
```json
{ "data": { "id": 42, "document_number": "BPAY-00042", "status": "paid", "total": "22200000.00000", "...": "..." } }
```

### Response 422
```json
{
  "message": "The amount field must be greater than 0.",
  "errors": {
    "amount": ["The amount field must be greater than 0."],
    "payment_account_id": ["The payment account field is required."]
  }
}
```

---

## PUT /api/v1/bill-payments/{id} — Ubah

Field sama dengan POST. Saldo tagihan terkait dihitung ulang. Response 200 berisi objek terbaru.

---

## Endpoint Bantu (Helper)

Endpoint ini menyediakan data untuk membangun form pembayaran (daftar vendor, akun, dan tagihan yang masih ada saldo).

### GET /api/v1/bill-payments/new-page
**Query (opsional):** `vendor_id`, `currency_code`, `date`. Mengembalikan `{ "data": { ... } }` berisi data bantu (vendor dengan `payable_bills_count`, akun pembayaran, dll.).

### GET /api/v1/bill-payments/payment-bills
**Query (opsional):** `vendor_id`, `currency_code`, `date`, `exclude_payment_id`. Mengembalikan koleksi tagihan yang bisa dibayar:
```json
{
  "data": [
    {
      "id": 1,
      "document_number": "BILL-000001",
      "bill_number": "BILL-000001",
      "status": "unpaid",
      "date": "2026-06-05",
      "due_date": "2026-07-05",
      "vendor_id": 9,
      "currency": { "code": "IDR", "exchange_rate": "1.00000000" },
      "total": "22200000.00000"
    }
  ],
  "meta": { "filters": { "vendor_id": 9 } }
}
```

### GET /api/v1/bill-payments/{id}/edit-page
Mengembalikan `{ "data": { ..., "payment": {...}, "metadata": {...} } }` untuk mengisi ulang form ubah.

---

## Email, Validasi Hapus, Hapus Massal, Hapus

Pola **identik** dengan invoices (lihat [03-sales-invoices.md](03-sales-invoices.md)):

- `GET /{id}/mail`, `POST /{id}/mail` → `{ "message": "Bill payment email sent.", "data": {...} }`.
- `POST /validate-delete`, `POST /bulk-delete` → `{ "ids": [...] }`.
- `DELETE /{id}` → 204 (tagihan terkait dihitung ulang).
