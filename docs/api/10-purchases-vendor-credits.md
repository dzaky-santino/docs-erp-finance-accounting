# Vendor Credits — Kredit Vendor

Prefix: `/api/v1/vendor-credits` · Autentikasi: Bearer token · Lihat [00-panduan-umum.md](00-panduan-umum.md).

Kredit Vendor adalah nota kredit dari sisi pembelian (vendor memberi Anda kredit, mis. retur/potongan). Bisa **diterapkan ke tagihan** atau **di-refund** (uang kembali). Lihat `docs/presentation/03-pembelian.md`.

---

## Daftar Endpoint

| Method | URL | Deskripsi | Permission |
| --- | --- | --- | --- |
| GET | `/api/v1/vendor-credits` | Daftar kredit vendor | `vendor-credit.view` |
| GET | `/api/v1/vendor-credits/{id}` | Detail (atau PDF) | `vendor-credit.view` |
| POST | `/api/v1/vendor-credits` | Buat kredit vendor (draft) | `vendor-credit.create` |
| PUT/PATCH | `/api/v1/vendor-credits/{id}` | Ubah kredit vendor | `vendor-credit.edit` |
| POST | `/api/v1/vendor-credits/{id}/open` | Buka (buat jurnal) | `vendor-credit.edit` |
| POST | `/api/v1/vendor-credits/{id}/applications` | Terapkan ke tagihan | `vendor-credit.edit` |
| DELETE | `/api/v1/vendor-credits/{id}/applications/{applicationId}` | Hapus penerapan | `vendor-credit.edit` |
| POST | `/api/v1/vendor-credits/{id}/refunds` | Catat pengembalian uang | `vendor-credit.edit` |
| DELETE | `/api/v1/vendor-credits/{id}/refunds/{refundId}` | Hapus refund | `vendor-credit.edit` |
| GET | `/api/v1/vendor-credits/{id}/mail` | Status/draft email | `vendor-credit.edit` |
| POST | `/api/v1/vendor-credits/{id}/mail` | Kirim via email | `vendor-credit.edit` |
| POST | `/api/v1/vendor-credits/validate-delete` | Validasi hapus massal | `vendor-credit.delete` |
| POST | `/api/v1/vendor-credits/bulk-delete` | Hapus massal | `vendor-credit.delete` |
| DELETE | `/api/v1/vendor-credits/{id}` | Hapus satu | `vendor-credit.delete` |

**Status valid (`status`):** `draft`, `open`, `closed` (kebab-case di response).

---

## GET /api/v1/vendor-credits — Daftar

**Query Parameters:** `page`, `per_page` (5/10/20/50/100), `search`, `status`, `vendor_id`, `from_date`, `to_date`.

### Response 200
```json
{
  "data": [
    {
      "id": 1,
      "document_number": "VC-00001",
      "status": "open",
      "date": "2026-06-22",
      "reference_no": "RET-VND-01",
      "note": "Retur 1 unit cacat",
      "vendor": { "id": 9, "display_name": "PT Sumber Makmur", "email": "sales@sumber.co.id" },
      "currency": { "code": "IDR", "exchange_rate": "1.00000000" },
      "subtotal": "2000000.00000",
      "tax_total": "220000.00000",
      "total": "2220000.00000",
      "refunded_amount": "0.00000",
      "applied_amount": "0.00000",
      "invoiced_amount": "0.00000",
      "remaining_amount": "2220000.00000",
      "discount": "0.00",
      "discount_type": "percentage",
      "adjustment": "0.00",
      "project": null,
      "lines": [
        {
          "id": 6, "index": 1,
          "item": { "id": 7, "name": "Laptop 14\"", "code": "DEMO-PRD-001" },
          "description": "Retur 1 unit",
          "quantity": "1.00000", "rate": "2000000.00000",
          "discount": "0.00000", "discount_type": "percentage",
          "tax_rate": "11.0000", "is_inclusive_tax": false,
          "amount": "2000000.00000"
        }
      ],
      "applications": [],
      "refunds": [],
      "opened_at": "2026-06-22T08:00:00+00:00",
      "created_at": "2026-06-22T08:00:00+00:00",
      "updated_at": "2026-06-22T08:00:00+00:00"
    }
  ],
  "links": { "first": "...", "last": "...", "prev": null, "next": null },
  "meta": { "current_page": 1, "per_page": 10, "total": 1, "last_page": 1 }
}
```

**Field khas:** `document_number` (`VC-XXXXX`), `refunded_amount`, `applied_amount`/`invoiced_amount` (jumlah yang sudah diterapkan ke tagihan), `remaining_amount` (sisa kredit), `applications[]` (penerapan ke tagihan, muncul di detail), `refunds[]` (riwayat refund, muncul di detail), `opened_at`.

**`applications[]`:** `{id, bill_id, amount}`.
**`refunds[]`:** `{id, deposit_account_id, date, amount, reference_no}`.

---

## POST /api/v1/vendor-credits — Buat Kredit Vendor

### Request Body
```json
{
  "vendor_id": 9,
  "vendor_credit_number": "VC-00042",
  "vendor_credit_date": "2026-06-22",
  "reference_no": "RET-VND-01",
  "note": "Retur barang cacat",
  "currency_code": "IDR",
  "exchange_rate": 1.0,
  "discount": 0,
  "discount_type": "percentage",
  "adjustment": 0,
  "branch_id": 1,
  "project_id": null,
  "entries": [
    { "item_id": 7, "description": "Retur 1 unit laptop", "quantity": 1, "rate": 2000000, "cost_account_id": 14, "tax_rate_id": 1, "tax_rate": 11 }
  ]
}
```

**Keterangan Field Request:**

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `vendor_id` | ✓ | integer | `exists:contacts` |
| `vendor_credit_date` | ✓ | date | Tanggal kredit vendor |
| `vendor_credit_number` | – | string | Maks 50, unik (auto bila kosong) |
| `reference_no` | – | string | Maks 50 |
| `note` | – | string | Maks 2000 |
| `currency_code` | – | string | `exists:currencies,code` |
| `exchange_rate` | – | numeric | > 0 |
| `discount` / `discount_type` / `adjustment` | – | – | Diskon & penyesuaian header |
| `warehouse_id` / `branch_id` / `project_id` | – | integer | Relasi opsional |
| `entries` | ✓ | array | Min 1, maks 500 |
| `entries[].quantity` | ✓ | numeric | > 0 |
| `entries[].rate` | ✓ | numeric | ≥ 0 |
| `entries[].item_id` | – | integer | Boleh kosong |
| `entries[].description` | – | string | Maks 1000 |
| `entries[].cost_account_id` | – | integer | Akun biaya/persediaan |
| `entries[].discount` / `discount_type` / `tax_rate_id` / `tax_rate` | – | – | Opsional |

> **Field internal yang ditolak (prohibited):** kolom status/saldo/audit yang dikelola sistem (lewat trait `RejectsVendorCreditApiInternalFields`).

### Response 201
```json
{ "data": { "id": 42, "document_number": "VC-00042", "status": "draft", "total": "2220000.00000", "remaining_amount": "2220000.00000", "...": "..." } }
```

### Response 422
```json
{
  "message": "The vendor field is required.",
  "errors": { "vendor_id": ["The vendor field is required."] }
}
```

---

## PUT /api/v1/vendor-credits/{id} — Ubah

Field sama dengan POST. Tidak bisa diubah bila sudah dibuka / sudah ada refund/penerapan. Response 200 berisi objek terbaru.

---

## POST /api/v1/vendor-credits/{id}/open — Buka

Mengubah status menjadi `open` & membuat jurnal GL (Debit Hutang Usaha, Kredit biaya/persediaan). 

### Response 200
```json
{ "data": { "id": 42, "status": "open", "opened_at": "2026-06-22T08:00:00+00:00", "...": "..." } }
```

### Response 422 — Sudah dibuka
```json
{ "message": "Vendor Credit has already been opened." }
```

---

## POST /api/v1/vendor-credits/{id}/applications — Terapkan ke Tagihan

### Request Body
```json
{
  "applications": [
    { "bill_id": 1, "amount": 2220000 }
  ]
}
```

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `applications` | ✓ | array | Min 1, maks 100 |
| `applications[].bill_id` | ✓ | integer | `exists:bills` |
| `applications[].amount` | ✓ | numeric | > 0 |

> Field `bill_id` & `amount` di level atas, serta kolom audit pada `applications.*`, **ditolak (prohibited)**.

### Response 200
Mengembalikan objek kredit vendor terbaru (dengan `applications[]` terisi).

### Response 422 — Saldo kredit tidak cukup
```json
{ "message": "Vendor Credit remaining balance (1000000.00) is insufficient for the requested amount (2220000.00)." }
```

---

## DELETE /api/v1/vendor-credits/{id}/applications/{applicationId} — Hapus Penerapan

Membatalkan satu penerapan ke tagihan (hutang tagihan dipulihkan). Response **204 No Content**.

---

## POST /api/v1/vendor-credits/{id}/refunds — Catat Pengembalian Uang

### Request Body
```json
{
  "deposit_account_id": 2,
  "amount": 2220000,
  "date": "2026-06-23",
  "reference_no": "RFND-VND-01",
  "description": "Pengembalian dari vendor"
}
```

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `deposit_account_id` | ✓ | integer | Akun Kas/Bank tujuan uang masuk |
| `amount` | ✓ | numeric | > 0 |
| `date` | ✓ | date | Tanggal refund |
| `reference_no` | – | string | Maks 50 |
| `description` | – | string | Maks 1000 |

### Response 201
Mengembalikan objek kredit vendor terbaru (status bisa `closed` bila kredit habis).

---

## DELETE /api/v1/vendor-credits/{id}/refunds/{refundId} — Hapus Refund

Membatalkan satu refund. Response **204 No Content**.

---

## Email, Validasi Hapus, Hapus Massal, Hapus

Pola **identik** dengan invoices (lihat [03-sales-invoices.md](03-sales-invoices.md)):

- `GET /{id}/mail`, `POST /{id}/mail` → `{ "message": "Vendor credit email sent.", "data": {...} }`.
- `POST /validate-delete`, `POST /bulk-delete` → `{ "ids": [...] }`.
- `DELETE /{id}` → 204.
