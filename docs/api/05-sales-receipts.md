# Sale Receipts — Penerimaan Penjualan (Tunai)

Prefix: `/api/v1/sale-receipts` · Autentikasi: Bearer token · Lihat [00-panduan-umum.md](00-panduan-umum.md).

Penerimaan Penjualan adalah penjualan **tunai langsung** — pelanggan membayar di tempat, tanpa piutang. Saat **ditutup (close)**, jurnal GL terbentuk: Debit Kas/Bank, Kredit Pendapatan + Pajak. Lihat `docs/presentation/02-penjualan.md`.

---

## Daftar Endpoint

| Method | URL | Deskripsi | Permission |
| --- | --- | --- | --- |
| GET | `/api/v1/sale-receipts` | Daftar penerimaan | `sale-receipt.view` |
| GET | `/api/v1/sale-receipts/{id}` | Detail (atau PDF) | `sale-receipt.view` |
| POST | `/api/v1/sale-receipts` | Buat baru (draft) | `sale-receipt.create` |
| PUT/PATCH | `/api/v1/sale-receipts/{id}` | Ubah | `sale-receipt.edit` |
| POST | `/api/v1/sale-receipts/{id}/close` | Tutup (buat jurnal) | `sale-receipt.edit` |
| GET | `/api/v1/sale-receipts/{id}/mail` | Status/draft email | `sale-receipt.edit` |
| POST | `/api/v1/sale-receipts/{id}/mail` | Kirim via email | `sale-receipt.edit` |
| POST | `/api/v1/sale-receipts/validate-delete` | Validasi hapus massal | `sale-receipt.delete` |
| POST | `/api/v1/sale-receipts/bulk-delete` | Hapus massal | `sale-receipt.delete` |
| DELETE | `/api/v1/sale-receipts/{id}` | Hapus satu | `sale-receipt.delete` |

**Status valid (`status`):** `draft`, `closed`.

---

## GET /api/v1/sale-receipts — Daftar

**Query Parameters:** `page`, `per_page` (5/10/20/50/100), `search`, `status`, `customer_id`, `from_date`, `to_date`.

### Response 200
```json
{
  "data": [
    {
      "id": 1,
      "document_number": "SR-00001",
      "status": "closed",
      "date": "2026-06-15",
      "customer": { "id": 5, "display_name": "Toko Sejahtera", "email": null },
      "currency": { "code": "IDR", "exchange_rate": "1.00000000" },
      "subtotal": "5000000.00000",
      "tax_total": "550000.00000",
      "total": "5550000.00000",
      "deposit_account": { "id": 2, "name": "Bank BCA", "code": "1-1200" },
      "project": null,
      "lines": [
        {
          "id": 7, "index": 1,
          "item": { "id": 8, "name": "Mouse Wireless", "code": "DEMO-PRD-002" },
          "description": "Penjualan tunai",
          "quantity": "10.00000", "rate": "500000.00000",
          "discount": "0.00000", "discount_type": "percentage",
          "tax_rate": "11.0000", "is_inclusive_tax": false,
          "amount": "5000000.00000"
        }
      ],
      "created_at": "2026-06-15T09:00:00+00:00",
      "updated_at": "2026-06-15T09:00:00+00:00"
    }
  ],
  "links": { "first": "...", "last": "...", "prev": null, "next": null },
  "meta": { "current_page": 1, "per_page": 10, "total": 1, "last_page": 1 }
}
```

**Field khas:** `document_number` (`SR-XXXXX`), `status` hanya `draft`/`closed`, `deposit_account` (`{id,name,code}`) = akun Kas/Bank tujuan uang, `total` (= kolom `amount`).

---

## POST /api/v1/sale-receipts — Buat Baru

### Request Body
```json
{
  "customer_id": 5,
  "deposit_account_id": 2,
  "receipt_number": "SR-00042",
  "receipt_date": "2026-06-17",
  "currency_code": "IDR",
  "exchange_rate": 1.0,
  "reference_no": "POS-001",
  "receipt_message": "Terima kasih.",
  "statement": "Pembayaran tunai diterima penuh.",
  "discount": 0,
  "discount_type": "percentage",
  "adjustment": 0,
  "branch_id": 1,
  "project_id": null,
  "entries": [
    { "item_id": 8, "description": "Mouse Wireless", "quantity": 10, "rate": 500000, "tax_rate_id": 1, "tax_rate": 11, "is_inclusive_tax": false }
  ]
}
```

**Keterangan Field Request:**

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `customer_id` | ✓ | integer | `exists:contacts` |
| `deposit_account_id` | ✓ | integer | Akun Kas/Bank tujuan (`exists:accounts`) |
| `receipt_date` | ✓ | date | Tanggal penerimaan |
| `receipt_number` | ✓ | string | **Wajib di API** (maks 50, unik) |
| `reference_no` | – | string | Maks 50 |
| `receipt_message` | – | string | Maks 2000 |
| `statement` | – | string | Maks 2000 |
| `currency_code` | – | string | `exists:currencies,code` |
| `exchange_rate` | – | numeric | > 0 |
| `discount` / `discount_type` / `adjustment` | – | – | Diskon & penyesuaian header |
| `warehouse_id` / `branch_id` / `project_id` | – | integer | Relasi opsional |
| `entries` | ✓ | array | Min 1, maks 500 |
| `entries[].quantity` | ✓ | numeric | > 0 |
| `entries[].rate` | ✓ | numeric | ≥ 0 |
| `entries[].item_id` | – | integer | Boleh kosong |
| `entries[].description` | – | string | Maks 1000 |
| `entries[].discount` / `discount_type` / `sell_account_id` / `tax_rate_id` / `tax_rate` | – | – | Opsional |
| `entries[].is_inclusive_tax` | – | boolean | Pajak termasuk harga? |

### Response 201
```json
{ "data": { "id": 42, "document_number": "SR-00042", "status": "draft", "total": "5550000.00000", "...": "..." } }
```

### Response 422
```json
{
  "message": "The deposit account field is required. (and 1 more error)",
  "errors": {
    "deposit_account_id": ["The deposit account field is required."],
    "receipt_number": ["The receipt number field is required."]
  }
}
```

---

## PUT /api/v1/sale-receipts/{id} — Ubah

Field sama dengan POST (`receipt_number` tidak diwajibkan saat update). Response 200 berisi objek terbaru.

---

## POST /api/v1/sale-receipts/{id}/close — Tutup Nota

Menutup nota & membuat jurnal GL (Debit Kas/Bank, Kredit Pendapatan + Pajak). Dicek terhadap periode terkunci.

### Response 200
```json
{ "data": { "id": 42, "status": "closed", "...": "..." } }
```

### Response 422 — Periode terkunci / akun wajib belum ada
```json
{ "message": "Transaction date 2026-05-31 is in a locked period (locked through 2026-05-31). ..." }
```

---

## Email, Validasi Hapus, Hapus Massal, Hapus

Pola **identik** dengan invoices (lihat [03-sales-invoices.md](03-sales-invoices.md)):

- `GET /{id}/mail`, `POST /{id}/mail` (`{to, cc, bcc, subject, message, attach_pdf}`) → `{ "message": "Sale receipt email sent.", "data": {...} }`.
- `POST /validate-delete`, `POST /bulk-delete` → `{ "ids": [...] }`.
- `DELETE /{id}` → 204.
