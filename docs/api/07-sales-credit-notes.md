# Credit Notes — Nota Kredit

Prefix: `/api/credit-notes` **(bukan `/api/v1`)** · Autentikasi: Bearer token · Lihat [00-panduan-umum.md](00-panduan-umum.md).

> ⚠️ **PENTING — Nota Kredit tidak termasuk REST API v1.**
> Tidak ada controller `Api\V1\CreditNoteController` dan tidak ada blok `/v1/credit-notes`. Endpoint nota kredit hanya tersedia pada **jalur mirror non-versi** (`/api/credit-notes`) yang memakai controller web `Finance\CreditNoteController`. Konsekuensinya berbeda dari resource v1:
> - **Format response berbeda:** mengembalikan **model Eloquent apa adanya** (mis. `{ "id": ..., "credit_note_number": ..., "status": ..., "entries": [...] }`), **tanpa** envelope `data`/`meta`/`links` dan **tanpa** penamaan seragam (`document_number`, `currency`, dll.) seperti resource v1.
> - **Tidak memakai middleware `permission:` di route**; otorisasi ditegakkan oleh `FormRequest::authorize()` (mis. `credit-note.create`). Endpoint baca (`index`/`show`) mengikuti otorisasi di controller.
> - Endpoint **`void`**, **`update`**, dan **`bulk-delete`** hanya tersedia pada jalur web (session/Inertia), **bukan** lewat token API.

Penjelasan bisnis nota kredit (apply, refund, void): `docs/presentation/02-penjualan.md` §5.

---

## Daftar Endpoint (dapat diakses via token)

| Method | URL | Deskripsi | Permission (via FormRequest) |
| --- | --- | --- | --- |
| GET | `/api/credit-notes` | Daftar nota kredit | `credit-note.view` |
| GET | `/api/credit-notes/{id}` | Detail nota kredit | `credit-note.view` |
| POST | `/api/credit-notes` | Buat nota kredit (draft) | `credit-note.create` |
| POST | `/api/credit-notes/{id}/open` | Terbitkan nota kredit | `credit-note.edit` |
| POST | `/api/credit-notes/{id}/apply` | Terapkan ke faktur | `credit-note.edit` |
| DELETE | `/api/credit-notes/applications/{applicationId}` | Hapus penerapan | `credit-note.edit` |
| POST | `/api/credit-notes/{id}/refund` | Catat pengembalian uang | `credit-note.edit` |
| DELETE | `/api/credit-notes/{id}/refunds/{refundId}` | Hapus refund | `credit-note.edit` |
| DELETE | `/api/credit-notes/{id}` | Hapus nota kredit | `credit-note.delete` |

**Hanya via web (tidak via token):** `PUT /api/credit-notes/{id}` (update), `POST /api/credit-notes/{id}/void` (void), `DELETE /api/credit-notes` (bulk-delete).

**Status valid (`status`):** `Draft`, `Open`, `Closed`, `Voided` (accessor model — perhatikan kapitalisasi berbeda dari resource v1 yang kebab-case).

---

## GET /api/credit-notes — Daftar

### Request
```
GET /api/credit-notes
Authorization: Bearer {token}
Accept: application/json
```

**Query Parameters (opsional):** `customer_id`, `status`, `search`, `from_date`, `to_date` (diteruskan ke service). Catatan: endpoint ini mengembalikan **koleksi langsung** (tanpa envelope paginasi v1).

### Response 200
```json
[
  {
    "id": 1,
    "credit_note_number": "CN-00001",
    "customer_id": 5,
    "credit_note_date": "2026-06-18",
    "currency_code": "IDR",
    "exchange_rate": 1,
    "amount": "2220000.00",
    "refunded_amount": "0.00",
    "invoices_amount": "0.00",
    "status": "Open",
    "can_be_voided": true,
    "created_at": "2026-06-18T08:00:00.000000Z",
    "updated_at": "2026-06-18T08:00:00.000000Z"
  }
]
```

> Field persis mengikuti model `CreditNote` (termasuk accessor `status`, `can_be_voided`, `voided_by_name`). Karena ini bukan resource v1, nama & format field bisa berbeda (mis. `amount` 2 desimal, `exchange_rate` numerik).

---

## GET /api/credit-notes/{id} — Detail

### Response 200
```json
{
  "id": 1,
  "credit_note_number": "CN-00001",
  "customer_id": 5,
  "credit_note_date": "2026-06-18",
  "reference_no": "RET-001",
  "note": "Retur 2 unit rusak",
  "currency_code": "IDR",
  "exchange_rate": 1,
  "amount": "2220000.00",
  "refunded_amount": "0.00",
  "invoices_amount": "0.00",
  "status": "Open",
  "entries": [
    { "id": 9, "item_id": 8, "description": "Mouse Wireless (retur)", "quantity": "2.00", "rate": "1000000.00", "tax_rate": "11.00" }
  ],
  "created_at": "2026-06-18T08:00:00.000000Z",
  "updated_at": "2026-06-18T08:00:00.000000Z"
}
```

### Response 404
```json
{ "message": "No query results for model [App\\Models\\CreditNote] 999" }
```

---

## POST /api/credit-notes — Buat Nota Kredit

### Request Body
```json
{
  "customer_id": 5,
  "credit_note_number": "CN-00042",
  "credit_note_date": "2026-06-18",
  "reference_no": "RET-001",
  "note": "Retur barang rusak",
  "terms_conditions": "",
  "currency_code": "IDR",
  "exchange_rate": 1.0,
  "discount": 0,
  "discount_type": "percentage",
  "adjustment": 0,
  "branch_id": 1,
  "project_id": null,
  "entries": [
    { "item_id": 8, "description": "Mouse Wireless (retur)", "quantity": 2, "rate": 1000000, "tax_rate_id": 1, "tax_rate": 11 }
  ]
}
```

**Keterangan Field Request:**

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `customer_id` | ✓ | integer | `exists:contacts` |
| `credit_note_date` | ✓ | date | Tanggal nota kredit |
| `credit_note_number` | – | string | Maks 50, unik (auto bila kosong) |
| `reference_no` | – | string | Maks 50 |
| `note` | – | string | Maks 2000 |
| `terms_conditions` | – | string | Maks 2000 |
| `currency_code` | – | string | `exists:currencies,code` |
| `exchange_rate` | – | numeric | > 0 |
| `discount` / `discount_type` / `adjustment` | – | – | Diskon & penyesuaian header |
| `warehouse_id` / `branch_id` / `project_id` | – | integer | Relasi opsional |
| `entries` | ✓ | array | Min 1, maks 500 |
| `entries[].quantity` | ✓ | numeric | > 0 |
| `entries[].rate` | ✓ | numeric | ≥ 0 |
| `entries[].item_id` | – | integer | Boleh kosong |
| `entries[].description` / `discount` / `discount_type` / `sell_account_id` / `tax_rate_id` / `tax_rate` | – | – | Opsional |

### Response 201
```json
{ "id": 42, "credit_note_number": "CN-00042", "status": "Draft", "amount": "2220000.00", "...": "..." }
```

### Response 422
```json
{
  "message": "The customer field is required.",
  "errors": { "customer_id": ["The customer field is required."] }
}
```

---

## POST /api/credit-notes/{id}/open — Terbitkan

Mengubah status menjadi `Open` & membuat jurnal GL. Response 200 berisi model nota kredit.

### Response 422 — Sudah terbit / periode terkunci
```json
{ "message": "Credit Note has already been opened." }
```

---

## POST /api/credit-notes/{id}/apply — Terapkan ke Faktur

### Request Body
```json
{
  "applications": [
    { "invoice_id": 42, "amount": 1110000 },
    { "invoice_id": 43, "amount": 1110000 }
  ]
}
```

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `applications` | ✓ | array | Min 1, maks 100 |
| `applications[].invoice_id` | ✓ | integer | `exists:sale_invoices` |
| `applications[].amount` | ✓ | numeric | > 0 |

### Response 200
Mengembalikan hasil penerapan (model + ringkasan). 

### Response 422 — Saldo kredit tidak cukup
```json
{ "message": "Credit Note remaining balance (1000000.00) is insufficient for the requested amount (1110000.00)." }
```

---

## DELETE /api/credit-notes/applications/{applicationId} — Hapus Penerapan

Membatalkan satu penerapan ke faktur (piutang faktur dipulihkan). Response **204 No Content**.

---

## POST /api/credit-notes/{id}/refund — Catat Pengembalian Uang

### Request Body
```json
{
  "deposit_account_id": 2,
  "amount": 2220000,
  "date": "2026-06-19",
  "reference_no": "RFND-001",
  "description": "Pengembalian tunai ke pelanggan"
}
```

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `deposit_account_id` | ✓ | integer | Akun Kas/Bank sumber dana |
| `amount` | ✓ | numeric | > 0 |
| `date` | ✓ | date | Tanggal refund |
| `reference_no` | – | string | Maks 50 |
| `description` | – | string | Maks 1000 |

### Response 201
Mengembalikan model nota kredit terbaru (status bisa menjadi `Closed` bila kredit habis).

---

## DELETE /api/credit-notes/{id}/refunds/{refundId} — Hapus Refund

Membatalkan satu refund. Response **204 No Content**.

---

## DELETE /api/credit-notes/{id} — Hapus Nota Kredit

Hanya nota kredit **Draft** (belum ada jurnal) yang aman dihapus. Response **204**; bila terhalang aturan bisnis → **422**.

---

## Catatan untuk Integrator

Karena nota kredit memakai jalur mirror (bukan v1), bila Anda membangun klien yang konsisten dengan resource v1, **tangani format response-nya secara khusus** (model mentah, status PascalCase, tanpa envelope paginasi). Bila kontrak v1 yang seragam dibutuhkan untuk nota kredit, itu perlu ditambahkan ke namespace `Api\V1` (saat ini belum ada).
