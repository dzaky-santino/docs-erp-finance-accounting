# Panduan Umum API ERP Finance

Dokumentasi ini menjelaskan **REST API v1** dari sistem ERP Finance & Accounting untuk integrasi eksternal (aplikasi lain, otomatisasi, mobile, dsb.).

> Semua contoh di dokumen ini diambil **langsung dari kode aktual** (route, controller, FormRequest, API Resource). Kontrak utamanya juga tersedia sebagai OpenAPI di `openapi/customer-documents.v1.yaml` dan `openapi/vendor-documents.v1.yaml`.

---

## Base URL

| Environment | Base URL |
| --- | --- |
| Development | `http://localhost:8000/api/v1` |
| Production | `https://domain-anda.com/api/v1` |

> **Penting:** prefix yang benar adalah **`/api/v1`** (bukan `/v1`). Endpoint autentikasi (`login`, `logout`, `me`) berada di luar `/v1`, yaitu langsung di bawah `/api` (mis. `POST /api/login`). Lihat [01-autentikasi.md](01-autentikasi.md).

---

## Format

- **Request:** JSON — header `Content-Type: application/json`.
- **Response:** JSON — header `Accept: application/json`.
- **Encoding:** UTF-8.
- **Tanggal:** format `YYYY-MM-DD` (mis. `2026-06-17`).
- **Tanggal-waktu** (`created_at`, `updated_at`, `opened_at`): format ISO-8601 (mis. `2026-06-17T08:00:00+00:00`).
- **Angka desimal:** dikirim & dikembalikan sebagai **string** dengan titik sebagai pemisah desimal, tanpa pemisah ribuan (mis. `"13320000.00000"`). Saat mengirim request, angka biasa (`13320000`) juga diterima.
  - Nilai uang/kuantitas: 5 angka desimal (mis. `"100.00000"`).
  - `tax_rate`: 4 angka desimal.
  - `exchange_rate`: 8 angka desimal.

---

## Header Wajib di Setiap Request

| Header | Nilai | Keterangan |
| --- | --- | --- |
| `Authorization` | `Bearer {token}` | Token Sanctum dari endpoint login. **Wajib** untuk semua endpoint kecuali `POST /api/login`. |
| `Accept` | `application/json` | Memastikan response berupa JSON (termasuk error validasi 422). |
| `Content-Type` | `application/json` | Wajib pada request yang berisi body (POST/PUT/PATCH). |

---

## Autentikasi

API menggunakan **Laravel Sanctum** dengan **Bearer token**. Alur singkat:

1. `POST /api/login` dengan email & password → menerima `token`.
2. Sertakan `Authorization: Bearer {token}` di setiap request berikutnya.
3. `POST /api/logout` untuk mencabut token.

Detail lengkap + contoh: [01-autentikasi.md](01-autentikasi.md).

---

## Otorisasi (Permission)

Selain autentikasi, setiap endpoint dijaga **permission** (RBAC Spatie) sesuai peran pengguna. Contoh: membuat faktur memerlukan permission `sale-invoice.create`. Bila token valid tetapi pengguna tidak punya permission, API mengembalikan **403 Forbidden**.

Permission setiap endpoint dicantumkan di tabel masing-masing resource (file 03–10).

---

## Struktur Response

### Daftar (list) — terpaginasi

Endpoint `GET /...` (index) mengembalikan koleksi terpaginasi (Laravel `LengthAwarePaginator`):

```json
{
  "data": [ { /* objek dokumen */ } ],
  "links": {
    "first": "http://localhost:8000/api/v1/invoices?page=1",
    "last": "http://localhost:8000/api/v1/invoices?page=3",
    "prev": null,
    "next": "http://localhost:8000/api/v1/invoices?page=2"
  },
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 3,
    "path": "http://localhost:8000/api/v1/invoices",
    "per_page": 10,
    "to": 10,
    "total": 25
  }
}
```

### Detail / buat / ubah

Mengembalikan satu objek dibungkus `data`:

```json
{ "data": { /* objek dokumen */ } }
```

### Hapus

Mengembalikan **204 No Content** tanpa body.

---

## Paginasi

| Parameter | Tipe | Default | Keterangan |
| --- | --- | --- | --- |
| `page` | integer | `1` | Halaman ke- |
| `per_page` | integer | `10` | Jumlah per halaman. **Hanya menerima** nilai `5`, `10`, `20`, `50`, `100`. Nilai lain otomatis dikembalikan ke `10`. |

---

## Konvensi Field Response Dokumen

Semua dokumen (faktur, estimasi, tagihan, dll.) memakai format konsisten:

| Field | Tipe | Keterangan |
| --- | --- | --- |
| `id` | integer | ID unik dokumen |
| `document_number` | string | Nomor dokumen (mis. `INV-000001`, `BILL-000003`) |
| `status` | string | Status dokumen dalam **kebab-case huruf kecil** (mis. `draft`, `unpaid`, `partially-paid`, `paid`) |
| `date` | date | Tanggal utama dokumen |
| `currency` | object | `{ "code": "IDR", "exchange_rate": "1.00000000" }` |
| `subtotal` | string | Total sebelum pajak (dihitung dari baris) |
| `tax_total` | string | Total pajak |
| `total` | string | Total dokumen (dalam mata uang dokumen) |
| `lines` | array | Baris item — hanya muncul saat relasi dimuat (umumnya di index & show) |
| `created_at` / `updated_at` | datetime | Stempel waktu ISO-8601 |

> Catatan: nilai **status** di response selalu kebab-case (mis. `partially-paid`), sedangkan **filter** `status` pada query menerima nilai sesuai enum yang tercantum di tiap resource.

---

## Versi API

**v1** — endpoint stabil. Perubahan yang merusak (breaking change) tidak akan dilakukan tanpa menaikkan versi. Field baru dapat ditambahkan tanpa menaikkan versi (bersifat aditif), jadi konsumen API harus mengabaikan field yang tidak dikenal.

---

## Daftar Resource API v1

| Resource | Prefix | Dokumen |
| --- | --- | --- |
| Faktur Penjualan | `/api/v1/invoices` | [03-sales-invoices.md](03-sales-invoices.md) |
| Estimasi Penjualan | `/api/v1/estimates` | [04-sales-estimates.md](04-sales-estimates.md) |
| Penerimaan Penjualan | `/api/v1/sale-receipts` | [05-sales-receipts.md](05-sales-receipts.md) |
| Penerimaan Pembayaran | `/api/v1/payment-receives` | [06-sales-payment-receives.md](06-sales-payment-receives.md) |
| Nota Kredit | `/api/credit-notes` (non-v1) | [07-sales-credit-notes.md](07-sales-credit-notes.md) |
| Tagihan Pembelian | `/api/v1/bills` | [08-purchases-bills.md](08-purchases-bills.md) |
| Pembayaran Tagihan | `/api/v1/bill-payments` | [09-purchases-bill-payments.md](09-purchases-bill-payments.md) |
| Kredit Vendor | `/api/v1/vendor-credits` | [10-purchases-vendor-credits.md](10-purchases-vendor-credits.md) |

> **Catatan penting:** Nota Kredit (Credit Note) **tidak tersedia** di namespace REST `/api/v1`. Endpoint-nya hanya ada pada jalur mirror non-versi (`/api/credit-notes`, autentikasi token Sanctum yang sama). Lihat [07-sales-credit-notes.md](07-sales-credit-notes.md) untuk detail & implikasinya.

Kode error & status: [11-error-codes.md](11-error-codes.md).
Panduan Postman: [02-postman.md](02-postman.md).
