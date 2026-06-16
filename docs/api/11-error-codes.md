# Kode Error API

Dokumen ini menjelaskan struktur error dan kode status yang mungkin dikembalikan API.

---

## Format Error Response

### Error validasi (422 dari FormRequest)

```json
{
  "message": "The customer field is required. (and 1 more error)",
  "errors": {
    "customer_id": ["The customer field is required."],
    "entries": ["The entries field is required."]
  }
}
```

- `message` — ringkasan error pertama.
- `errors` — objek; setiap key adalah nama field, nilainya array pesan.

> Bahasa pesan validasi mengikuti **bahasa organisasi** (Indonesia/Inggris). Bila organisasi disetel ke Indonesia, pesan menjadi mis. `"Kolom pelanggan wajib diisi."`.

### Error bisnis / domain (422 dari Service)

Hanya berisi `message` tunggal (tanpa `errors`):

```json
{ "message": "Invoice has already been delivered." }
```

### Error autentikasi / otorisasi

```json
{ "message": "Unauthenticated." }
```

---

## HTTP Status Code

| Kode | Nama | Kapan Terjadi |
| --- | --- | --- |
| 200 | OK | Request berhasil (GET, PUT, aksi, kirim email) |
| 201 | Created | Data berhasil dibuat (POST store, refund) |
| 204 | No Content | Penghapusan berhasil (DELETE) — tanpa body |
| 401 | Unauthorized | Token tidak ada / tidak valid / sudah dicabut |
| 403 | Forbidden | Token valid tetapi pengguna tidak punya permission yang dibutuhkan |
| 404 | Not Found | Dokumen / sumber daya tidak ditemukan |
| 422 | Unprocessable Entity | Validasi field gagal **atau** aturan bisnis dilanggar |
| 429 | Too Many Requests | Melebihi batas laju (rate limit), bila diaktifkan |
| 500 | Internal Server Error | Error tak terduga di server (mis. gagal kirim email) |

> Catatan arsitektur: seluruh `App\Exceptions\Domain\*` (DomainException) otomatis dipetakan ke **HTTP 422** dengan body `{ "message": ... }`. Controller API v1 juga menangkapnya secara eksplisit dan mengembalikan 422.

---

## Contoh 401 — Token Tidak Valid

```
GET /api/v1/invoices
(tanpa header Authorization)
```
```json
{ "message": "Unauthenticated." }
```
**Solusi:** sertakan `Authorization: Bearer {token}` yang valid. Lihat [01-autentikasi.md](01-autentikasi.md).

## Contoh 403 — Permission Kurang

Pengguna dengan peran `cashier` mencoba membuat faktur (`POST /api/v1/invoices` yang butuh `sale-invoice.create`):
```json
{ "message": "This action is unauthorized." }
```
**Solusi:** gunakan akun dengan permission yang sesuai (lihat tabel permission tiap resource).

## Contoh 404 — Tidak Ditemukan

```json
{ "message": "No query results for model [App\\Models\\SaleInvoice] 999" }
```

---

## Error Bisnis Umum (422)

Diambil dari exception domain aktual (`app/Exceptions/Domain/`). Pesan default dalam Bahasa Inggris (kontrak domain).

| Kondisi | Exception | Contoh pesan |
| --- | --- | --- |
| Dokumen sudah diterbitkan (faktur/estimasi) | `DocumentAlreadyDeliveredException` | `Invoice has already been delivered.` |
| Dokumen sudah dibuka (tagihan/kredit vendor) | `DocumentAlreadyOpenedException` | `Bill has already been opened.` |
| Nomor dokumen duplikat | `DuplicateNumberException` | `Invoice number 'INV-000042' already exists.` |
| Hapus dokumen yang punya pembayaran | `DocumentHasPaymentsException` | `Invoice cannot be deleted because it has associated payments.` |
| Hapus dokumen yang punya nota kredit terpakai | `DocumentHasCreditApplicationsException` | `Invoice cannot be deleted because it has applied credit notes or vendor credits.` |
| Total diturunkan di bawah yang sudah dibayar | `AmountBelowPaidException` | `Invoice total cannot be reduced below the already-paid amount.` |
| Periode terkunci | `TransactionLockedException` | `Transaction date 2026-05-31 is in a locked period (locked through 2026-05-31). Please unlock the period in Settings → Accounting before making changes.` |
| Akun wajib (Piutang/Hutang/Pajak) belum dibuat | `RequiredAccountNotFoundException` | `The required Piutang Usaha account (type: Accounts Receivable) was not found in the Chart of Accounts. ...` |
| Saldo kredit tidak cukup (apply credit/vendor credit) | `InsufficientCreditBalanceException` | `Credit Note remaining balance (1000000.00) is insufficient for the requested amount (1110000.00).` |
| Estimasi sudah dikonversi ke faktur | `EstimateAlreadyConvertedException` | `This estimate has already been converted to an invoice.` |
| Nota kredit sudah di-void | `CreditNoteAlreadyVoidedException` | `This credit note has already been voided.` |
| Jurnal manual tidak seimbang | `JournalNotBalancedException` | (debit ≠ kredit) |
| Mata uang dokumen & akun tidak cocok | `CurrencyMismatchException` | (selisih mata uang) |
| Email pelanggan/vendor kosong saat kirim | `MissingCustomerEmailException` / `MissingVendorEmailException` | (alamat email tidak tersedia) |

> Total exception domain: **56** kelas di `app/Exceptions/Domain/`. Semua mengembalikan **422** dengan field `message`.

---

## Error Hapus Massal (Bulk Delete)

Endpoint `bulk-delete` mengembalikan **422** bila ada dokumen yang tidak bisa dihapus, dengan rincian:

```json
{
  "message": "One or more documents cannot be deleted.",
  "deletable": [42],
  "blocked": [
    { "id": 43, "message": "Invoice cannot be deleted because it has associated payments." }
  ]
}
```

Gunakan endpoint `validate-delete` terlebih dahulu untuk mengetahui ID mana yang aman, tanpa menghapus apa pun.

---

## Tips Penanganan Error di Klien

1. **Selalu kirim** header `Accept: application/json` agar error validasi dikembalikan sebagai JSON (bukan halaman HTML redirect).
2. Periksa **status code** dulu, baru baca body.
3. Untuk **422**, cek apakah ada `errors` (validasi per field) atau hanya `message` (aturan bisnis).
4. Untuk **401**, lakukan login ulang untuk memperbarui token.
5. Untuk **403**, periksa permission peran pengguna.
6. Untuk **500**, jangan ulangi membabi-buta — laporkan; mungkin masalah server (mis. konfigurasi email).
