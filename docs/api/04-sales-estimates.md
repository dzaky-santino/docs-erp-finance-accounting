# Estimates — Estimasi Penjualan

Prefix: `/api/v1/estimates` · Autentikasi: Bearer token · Lihat [00-panduan-umum.md](00-panduan-umum.md).

Estimasi adalah penawaran harga ke pelanggan. **Tidak membentuk jurnal GL** (non-GL) — baru berdampak akuntansi setelah dikonversi menjadi faktur lewat jalur web/mirror. Lihat `docs/presentation/02-penjualan.md`.

---

## Daftar Endpoint

| Method | URL | Deskripsi | Permission |
| --- | --- | --- | --- |
| GET | `/api/v1/estimates` | Daftar estimasi | `sale-estimate.view` |
| GET | `/api/v1/estimates/{id}` | Detail estimasi (atau PDF) | `sale-estimate.view` |
| POST | `/api/v1/estimates` | Buat estimasi baru | `sale-estimate.create` |
| PUT/PATCH | `/api/v1/estimates/{id}` | Ubah estimasi | `sale-estimate.edit` |
| POST | `/api/v1/estimates/{id}/deliver` | Tandai terkirim | `sale-estimate.edit` |
| GET | `/api/v1/estimates/{id}/mail` | Status/draft email | `sale-estimate.edit` |
| POST | `/api/v1/estimates/{id}/mail` | Kirim via email | `sale-estimate.edit` |
| POST | `/api/v1/estimates/validate-delete` | Validasi hapus massal | `sale-estimate.delete` |
| POST | `/api/v1/estimates/bulk-delete` | Hapus massal | `sale-estimate.delete` |
| DELETE | `/api/v1/estimates/{id}` | Hapus satu estimasi | `sale-estimate.delete` |

**Status valid (`status`):** `draft`, `delivered`, `approved`, `rejected`, `expired`, `invoiced` (nilai dikembalikan dalam kebab-case).

> Aksi `approve`, `reject`, dan `convert` (ke faktur) **tidak** tersedia di `/api/v1`; gunakan jalur mirror `/api/estimates/{id}/approve|reject|convert`.

---

## GET /api/v1/estimates — Daftar Estimasi

**Query Parameters:** `page`, `per_page` (5/10/20/50/100), `search`, `status`, `customer_id`, `from_date`, `to_date`. (Sama seperti invoices.)

### Response 200
```json
{
  "data": [
    {
      "id": 1,
      "document_number": "EST-00001",
      "status": "draft",
      "date": "2026-06-10",
      "expiration_date": "2026-06-30",
      "customer": { "id": 5, "display_name": "PT Maju Bersama", "email": "po@maju.co.id" },
      "currency": { "code": "IDR", "exchange_rate": "1.00000000" },
      "subtotal": "10000000.00000",
      "tax_total": "1100000.00000",
      "total": "11100000.00000",
      "project": null,
      "lines": [
        {
          "id": 5, "index": 1,
          "item": { "id": 3, "name": "Jasa Konsultasi IT", "code": "SVC-001" },
          "description": "Penawaran konsultasi",
          "quantity": "1.00000", "rate": "10000000.00000",
          "discount": "0.00000", "discount_type": "percentage",
          "tax_rate": "11.0000", "is_inclusive_tax": false,
          "amount": "10000000.00000"
        }
      ],
      "created_at": "2026-06-10T08:00:00+00:00",
      "updated_at": "2026-06-10T08:00:00+00:00"
    }
  ],
  "links": { "first": "...", "last": "...", "prev": null, "next": null },
  "meta": { "current_page": 1, "per_page": 10, "total": 4, "last_page": 1 }
}
```

**Field khas estimasi:** `document_number` (`EST-XXXXX`), `expiration_date` (tanggal kedaluwarsa penawaran), `total` (= kolom `amount`). Field baris `lines[]` sama dengan invoices.

---

## GET /api/v1/estimates/{id} — Detail

Mengembalikan `{ "data": { ... } }`. Tambahkan `?format=pdf` untuk unduh PDF. 404 bila tidak ditemukan.

---

## POST /api/v1/estimates — Buat Estimasi

### Request Body
```json
{
  "customer_id": 5,
  "estimate_number": "EST-00042",
  "estimate_date": "2026-06-17",
  "expiration_date": "2026-07-01",
  "currency_code": "IDR",
  "exchange_rate": 1.0,
  "reference": "RFQ-001",
  "note": "Penawaran berlaku 14 hari.",
  "terms_conditions": "Harga belum termasuk ongkir.",
  "send_to_email": "po@maju.co.id",
  "discount": 0,
  "discount_type": "percentage",
  "adjustment": 0,
  "branch_id": 1,
  "project_id": null,
  "entries": [
    { "item_id": 3, "description": "Konsultasi", "quantity": 1, "rate": 10000000, "tax_rate_id": 1, "tax_rate": 11 }
  ]
}
```

**Keterangan Field Request:**

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `customer_id` | ✓ | integer | `exists:contacts` |
| `estimate_number` | ✓ | string | **Wajib di API** (maks 50, unik) |
| `estimate_date` | ✓ | date | Tanggal estimasi |
| `expiration_date` | – | date | ≥ `estimate_date` |
| `reference` | – | string | Maks 50 |
| `note` | – | string | Maks 2000 |
| `terms_conditions` | – | string | Maks 2000 |
| `send_to_email` | – | string | Email valid, maks 255 |
| `currency_code` | – | string | `exists:currencies,code` |
| `exchange_rate` | – | numeric | > 0 |
| `discount` | – | numeric | ≥ 0 |
| `discount_type` | – | string | `percentage` / `amount` |
| `adjustment` | – | numeric | Penyesuaian |
| `warehouse_id` / `branch_id` / `project_id` | – | integer | Relasi opsional |
| `entries` | ✓ | array | Min 1, maks 500 |
| `entries[].quantity` | ✓ | numeric | > 0 |
| `entries[].rate` | ✓ | numeric | ≥ 0 |
| `entries[].item_id` | – | integer | Boleh kosong (baris bebas) |
| `entries[].description` | – | string | Maks 1000 |
| `entries[].discount` / `discount_type` / `sell_account_id` / `tax_rate_id` / `tax_rate` | – | – | Opsional (sama seperti invoices) |

### Response 201
```json
{ "data": { "id": 42, "document_number": "EST-00042", "status": "draft", "total": "11100000.00000", "...": "..." } }
```

### Response 422
```json
{
  "message": "The customer field is required. (and 1 more error)",
  "errors": {
    "customer_id": ["The customer field is required."],
    "estimate_number": ["The estimate number field is required."]
  }
}
```

---

## PUT /api/v1/estimates/{id} — Ubah Estimasi

Field sama dengan POST (`estimate_number` tidak diwajibkan saat update). Response 200 berisi objek estimasi terbaru.

---

## POST /api/v1/estimates/{id}/deliver — Tandai Terkirim

Mengubah status menjadi `delivered`. Tidak membuat jurnal (estimasi non-GL).

### Response 200
```json
{ "data": { "id": 42, "status": "delivered", "...": "..." } }
```

### Response 422 — Sudah terkirim
```json
{ "message": "Estimate has already been delivered." }
```

---

## Email, Validasi Hapus, Hapus Massal, Hapus

Pola **identik** dengan invoices:

- `GET /{id}/mail` → draft email.
- `POST /{id}/mail` → body `{to, cc, bcc, subject, message, attach_pdf}`; response `{ "message": "Estimate email sent.", "data": {...} }`.
- `POST /validate-delete` → body `{ "ids": [...] }` → `{deletable, blocked}`.
- `POST /bulk-delete` → body `{ "ids": [...] }` → `{deleted}` (200) atau terblokir (422).
- `DELETE /{id}` → 204 No Content.

Lihat contoh lengkap di [03-sales-invoices.md](03-sales-invoices.md).
