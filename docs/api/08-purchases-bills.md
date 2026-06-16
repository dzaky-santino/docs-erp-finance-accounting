# Bills — Tagihan Pembelian

Prefix: `/api/v1/bills` · Autentikasi: Bearer token · Lihat [00-panduan-umum.md](00-panduan-umum.md).

Tagihan (Bill) adalah hutang ke vendor atas pembelian. Dibuat sebagai **draft**, lalu **dibuka (open)** untuk membentuk jurnal hutang. Lihat `docs/presentation/03-pembelian.md`.

---

## Daftar Endpoint

| Method | URL | Deskripsi | Permission |
| --- | --- | --- | --- |
| GET | `/api/v1/bills` | Daftar tagihan | `bill.view` |
| GET | `/api/v1/bills/{id}` | Detail (atau PDF) | `bill.view` |
| POST | `/api/v1/bills` | Buat tagihan (draft) | `bill.create` |
| PUT/PATCH | `/api/v1/bills/{id}` | Ubah tagihan | `bill.edit` |
| POST | `/api/v1/bills/{id}/open` | Buka tagihan (buat jurnal) | `bill.edit` |
| POST | `/api/v1/bills/validate-delete` | Validasi hapus massal | `bill.delete` |
| POST | `/api/v1/bills/bulk-delete` | Hapus massal | `bill.delete` |
| DELETE | `/api/v1/bills/{id}` | Hapus satu | `bill.delete` |

**Status valid (`status`):** `draft`, `unpaid`, `partially-paid`, `paid` (kebab-case di response).

> Tagihan **tidak punya** endpoint email (`mail`/`mail-state`) di v1.

---

## GET /api/v1/bills — Daftar

**Query Parameters:** `page`, `per_page` (5/10/20/50/100), `search`, `status`, `vendor_id`, `from_date`, `to_date`.

### Response 200
```json
{
  "data": [
    {
      "id": 1,
      "document_number": "BILL-000001",
      "status": "unpaid",
      "date": "2026-06-05",
      "due_date": "2026-07-05",
      "reference_no": "INV-VND-77",
      "note": "Pembelian barang dagangan",
      "vendor": { "id": 9, "display_name": "PT Sumber Makmur", "email": "sales@sumber.co.id" },
      "currency": { "code": "IDR", "exchange_rate": "1.00000000" },
      "subtotal": "20000000.00000",
      "tax_total": "2200000.00000",
      "total": "22200000.00000",
      "balance_due": "22200000.00000",
      "amount_due": "22200000.00000",
      "payment_amount": "0.00000",
      "credited_amount": "0.00000",
      "landed_cost_amount": "0.00000",
      "allocated_cost_amount": "0.00000",
      "is_inclusive_tax": false,
      "project": null,
      "lines": [
        {
          "id": 11, "index": 1,
          "item": { "id": 7, "name": "Laptop 14\"", "code": "DEMO-PRD-001" },
          "description": "Pembelian 10 unit",
          "quantity": "10.00000", "rate": "2000000.00000",
          "discount": "0.00000", "discount_type": "percentage",
          "tax_rate": "11.0000", "is_inclusive_tax": false,
          "amount": "20000000.00000"
        }
      ],
      "opened_at": "2026-06-05T08:00:00+00:00",
      "created_at": "2026-06-05T08:00:00+00:00",
      "updated_at": "2026-06-05T08:00:00+00:00"
    }
  ],
  "links": { "first": "...", "last": "...", "prev": null, "next": null },
  "meta": { "current_page": 1, "per_page": 10, "total": 1, "last_page": 1 }
}
```

**Field khas tagihan:** `document_number` (`BILL-XXXXXX`), `vendor` (`{id,display_name,email}`), `reference_no`, `note`, `total` (= kolom `amount`), `balance_due`/`amount_due` (= amount − payment − credited), `landed_cost_amount`, `allocated_cost_amount`, `is_inclusive_tax`, `opened_at`.

---

## GET /api/v1/bills/{id} — Detail

`{ "data": { ... } }`. Tambahkan `?format=pdf` untuk PDF. 404 bila tidak ditemukan.

---

## POST /api/v1/bills — Buat Tagihan

### Request Body
```json
{
  "vendor_id": 9,
  "bill_number": "BILL-000042",
  "bill_date": "2026-06-17",
  "due_date": "2026-07-17",
  "reference_no": "INV-VND-100",
  "note": "Pembelian barang dagangan",
  "currency_code": "IDR",
  "exchange_rate": 1.0,
  "is_inclusive_tax": false,
  "discount": 0,
  "discount_type": "percentage",
  "adjustment": 0,
  "branch_id": 1,
  "project_id": null,
  "entries": [
    {
      "item_id": 7,
      "description": "Pembelian 10 unit laptop",
      "quantity": 10,
      "rate": 2000000,
      "discount": 0,
      "discount_type": "percentage",
      "cost_account_id": 14,
      "is_landed_cost": false,
      "tax_rate_id": 1,
      "tax_rate": 11
    }
  ]
}
```

**Keterangan Field Request:**

| Field | Wajib | Tipe | Keterangan |
| --- | --- | --- | --- |
| `vendor_id` | ✓ | integer | `exists:contacts` |
| `bill_number` | – | string | Maks 50, unik (auto bila kosong) |
| `bill_date` | ✓ | date | Tanggal tagihan |
| `due_date` | ✓ | date | Harus ≥ `bill_date` |
| `reference_no` | – | string | Maks 50 (mis. nomor faktur dari vendor) |
| `note` | – | string | Maks 2000 |
| `currency_code` | – | string | `exists:currencies,code` |
| `exchange_rate` | – | numeric | > 0 |
| `is_inclusive_tax` | – | boolean | Pajak termasuk harga? |
| `discount` / `discount_type` / `adjustment` | – | – | Diskon & penyesuaian header |
| `warehouse_id` / `branch_id` / `project_id` | – | integer | Relasi opsional |
| `entries` | ✓ | array | Min 1, maks 500 |
| `entries[].quantity` | ✓ | numeric | > 0 |
| `entries[].rate` | ✓ | numeric | ≥ 0 |
| `entries[].item_id` | – | integer | Boleh kosong |
| `entries[].description` | – | string | Maks 1000 |
| `entries[].cost_account_id` | – | integer | Akun biaya/persediaan |
| `entries[].is_landed_cost` | – | boolean | Tandai baris sebagai komponen biaya tambahan |
| `entries[].discount` / `discount_type` / `tax_rate_id` / `tax_rate` | – | – | Opsional |

> **Field internal yang ditolak (prohibited) di API** (lewat trait `RejectsBillApiInternalFields`): kolom status/saldo/audit yang dikelola sistem. Kirim hanya field di tabel di atas.

### Response 201
```json
{ "data": { "id": 42, "document_number": "BILL-000042", "status": "draft", "total": "22200000.00000", "...": "..." } }
```

### Response 422
```json
{
  "message": "The vendor field is required.",
  "errors": {
    "vendor_id": ["The vendor field is required."],
    "due_date": ["The due date field must be a date after or equal to bill date."]
  }
}
```

---

## PUT /api/v1/bills/{id} — Ubah Tagihan

Field sama dengan POST. Bila tagihan sudah dibuka, jurnal ditulis ulang otomatis. Total tidak boleh di bawah jumlah yang sudah dibayar.

### Response 422 — Total di bawah yang sudah dibayar
```json
{ "message": "Bill total cannot be reduced below the already-paid amount." }
```

---

## POST /api/v1/bills/{id}/open — Buka Tagihan

Mengubah dari **draft** menjadi terbuka & **membuat jurnal GL** (Kredit Hutang Usaha, Debit Persediaan/Beban + Pajak). Dicek terhadap periode terkunci.

### Response 200
```json
{ "data": { "id": 42, "status": "unpaid", "opened_at": "2026-06-17T08:00:00+00:00", "...": "..." } }
```

### Response 422 — Sudah dibuka
```json
{ "message": "Bill has already been opened." }
```

---

## Validasi Hapus, Hapus Massal, Hapus

Pola **identik** dengan invoices (lihat [03-sales-invoices.md](03-sales-invoices.md)):

- `POST /validate-delete` → `{ "ids": [...] }` → `{deletable, blocked}`.
- `POST /bulk-delete` → `{ "ids": [...] }` → `{deleted}` (200) atau terblokir (422).
- `DELETE /{id}` → 204; bila terhalang (punya pembayaran / biaya tambahan terkait) → 422, mis.:
  ```json
  { "message": "Bill cannot be deleted because it has associated payments." }
  ```
