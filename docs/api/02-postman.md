# Panduan Menggunakan Postman

Panduan ini ditujukan untuk pembaca yang **belum pernah** memakai API. Dengan Postman, Anda bisa mencoba semua endpoint tanpa menulis kode.

---

## Apa itu Postman?

**Postman** adalah aplikasi gratis untuk menguji API. Bayangkan seperti "browser khusus API": Anda menentukan alamat (URL), metode (GET/POST/...), dan data yang dikirim, lalu Postman menampilkan respons dari server. Sangat berguna untuk memahami API sebelum mengintegrasikannya ke aplikasi.

---

## Instalasi

1. Buka https://www.postman.com/downloads/
2. Unduh sesuai sistem operasi (Windows/Mac/Linux).
3. Pasang dan buka aplikasinya. (Bisa dipakai tanpa membuat akun — pilih "Skip" bila diminta login.)

---

## Setup Environment di Postman

Environment menyimpan variabel yang dipakai berulang, seperti `base_url` dan `token`, agar tidak perlu menulis ulang di tiap request.

1. Klik ikon **Environments** (kiri) → **+** untuk membuat environment baru.
2. Beri nama, mis. `ERP Finance Local`.
3. Tambahkan dua variabel:

   | Variable | Initial value | Current value |
   | --- | --- | --- |
   | `base_url` | `http://localhost:8000/api` | `http://localhost:8000/api` |
   | `token` | *(kosongkan dulu)* | *(kosongkan dulu)* |

4. Klik **Save**, lalu pilih environment ini di pojok kanan atas.

> Catatan: `base_url` di atas berhenti di `/api`. Untuk endpoint v1 kita tulis `{{base_url}}/v1/...`, sedangkan login/logout cukup `{{base_url}}/login`.

---

## Langkah 1 — Login dan Dapatkan Token

1. Klik **New** → **HTTP Request**.
2. Setel metode ke **POST** dan URL: `{{base_url}}/login`
3. Buka tab **Headers**, tambahkan:
   - `Accept` : `application/json`
   - `Content-Type` : `application/json`
4. Buka tab **Body** → pilih **raw** → pilih tipe **JSON**, lalu isi:
   ```json
   {
     "email": "admin@erp.test",
     "password": "password"
   }
   ```
5. Klik **Send**.
6. Pada panel response bawah, salin nilai `token`.
7. Buka **Environments** → tempel nilai itu ke variabel `token` → **Save**.

### Tips: simpan token otomatis

Agar token langsung tersimpan setelah login, buka tab **Scripts → Post-response** (atau "Tests" pada versi lama) di request login dan tempel:

```javascript
const json = pm.response.json();
if (json.token) {
  pm.environment.set("token", json.token);
}
```

Dengan ini, setiap kali Anda login, variabel `token` terisi otomatis.

---

## Langkah 2 — Setup Authorization Global (Collection)

Agar tidak perlu menambah header token di setiap request:

1. Kelompokkan request Anda dalam sebuah **Collection** (klik **Collections → +**).
2. Klik kanan collection → **Edit** → tab **Authorization**.
3. Setel:
   - **Type:** `Bearer Token`
   - **Token:** `{{token}}`
4. **Save**. Semua request di dalam collection ini otomatis mewarisi token.

> Pastikan tiap request memakai **Auth Type: Inherit auth from parent**.

---

## Langkah 3 — Test Endpoint Pertama (Daftar Faktur)

1. Buat request baru: **GET** `{{base_url}}/v1/invoices`
2. Tab **Headers**: `Accept: application/json` (token sudah diwarisi dari collection).
3. Klik **Send**.
4. Anda akan melihat respons terpaginasi:
   ```json
   {
     "data": [ { "id": 1, "document_number": "INV-000001", "...": "..." } ],
     "links": { "first": "...", "last": "...", "prev": null, "next": "..." },
     "meta": { "current_page": 1, "per_page": 10, "total": 25, "last_page": 3 }
   }
   ```

### Contoh dengan query parameter

`GET {{base_url}}/v1/invoices?status=unpaid&per_page=20&page=1`

Di Postman, gunakan tab **Params** untuk menambahkan `status`, `per_page`, `page` tanpa mengetik manual di URL.

---

## Langkah 4 — Membuat Dokumen (POST)

1. **POST** `{{base_url}}/v1/invoices`
2. Tab **Headers**: `Accept` & `Content-Type` = `application/json`.
3. Tab **Body** → raw → JSON:
   ```json
   {
     "customer_id": 5,
     "invoice_no": "INV-000042",
     "invoice_date": "2026-06-17",
     "due_date": "2026-07-17",
     "entries": [
       { "item_id": 3, "quantity": 2, "rate": 2000000, "tax_rate_id": 1 }
     ]
   }
   ```
4. **Send** → respons **201 Created** berisi dokumen baru (status `draft`).

---

## Import Collection

Saat ini repo **tidak menyediakan** file Postman Collection (`.json`) siap impor. Anda punya dua pilihan:

1. **Buat manual** mengikuti langkah di atas (disarankan untuk belajar).
2. **Impor dari OpenAPI** — repo menyediakan kontrak OpenAPI di:
   - `openapi/customer-documents.v1.yaml`
   - `openapi/vendor-documents.v1.yaml`

   Di Postman: **Import → File** → pilih salah satu file YAML. Postman akan otomatis membuat collection berisi semua endpoint yang tercantum di kontrak tersebut. Setelah impor, set environment & authorization seperti di atas.

---

## Tips Membaca Response

| Bagian | Arti |
| --- | --- |
| `data` | Isi utama — objek dokumen, atau array dokumen pada endpoint daftar |
| `meta` | Info paginasi: `current_page`, `per_page`, `total`, `last_page` |
| `links` | Tautan navigasi halaman: `first`, `last`, `prev`, `next` |
| `message` | Muncul pada error / aksi tertentu (mis. pesan sukses kirim email) |
| `errors` | Muncul pada **422** — detail kesalahan per field |

### Membaca status code

Lihat angka status di kanan atas panel response (mis. `200 OK`, `201 Created`, `422 Unprocessable Entity`). Arti tiap kode: [11-error-codes.md](11-error-codes.md).

---

## Urutan Belajar yang Disarankan

1. Login → simpan token ([01-autentikasi.md](01-autentikasi.md)).
2. `GET /v1/invoices` — lihat daftar.
3. `GET /v1/invoices/{id}` — lihat detail satu faktur.
4. `POST /v1/invoices` — buat faktur draft.
5. `POST /v1/invoices/{id}/deliver` — terbitkan faktur.
6. Coba resource lain (estimates, bills, dst.) — polanya sama.
