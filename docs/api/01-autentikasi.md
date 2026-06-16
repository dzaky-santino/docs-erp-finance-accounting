# Autentikasi

API menggunakan **Laravel Sanctum** dengan **Bearer token** (personal access token). Setiap request (kecuali login) harus menyertakan token pada header `Authorization`.

> Endpoint autentikasi berada langsung di bawah `/api` (**bukan** `/api/v1`).

---

## Mendapatkan Token (Login)

### Request

```
POST /api/login
Content-Type: application/json
Accept: application/json
```

```json
{
  "email": "admin@erp.test",
  "password": "password"
}
```

### Response 200 — Berhasil

```json
{
  "user": {
    "id": 1,
    "name": "Administrator",
    "email": "admin@erp.test",
    "is_active": true,
    "created_at": "2026-06-01T00:00:00.000000Z",
    "updated_at": "2026-06-01T00:00:00.000000Z"
  },
  "token": "12|aBcDeF0123456789GhIjKlMnOpQrStUvWxYz..."
}
```

Salin nilai `token`. Inilah yang dipakai sebagai Bearer token.

### Response 422 — Kredensial salah

Karena keamanan (mencegah enumerasi pengguna), pesan sama baik email tidak ada maupun password salah:

```json
{
  "message": "These credentials do not match our records.",
  "errors": {
    "email": ["These credentials do not match our records."]
  }
}
```

### Response 422 — Akun nonaktif

```json
{
  "message": "Your account is inactive. Please contact your administrator.",
  "errors": {
    "email": ["Your account is inactive. Please contact your administrator."]
  }
}
```

---

## Menggunakan Token

Sertakan token pada **setiap** request ke endpoint API:

```
Authorization: Bearer 12|aBcDeF0123456789GhIjKlMnOpQrStUvWxYz...
Accept: application/json
```

Contoh ambil daftar faktur:

```
GET /api/v1/invoices
Authorization: Bearer {token}
Accept: application/json
```

Bila token tidak ada / tidak valid / sudah dicabut → **401 Unauthorized**:

```json
{ "message": "Unauthenticated." }
```

---

## Memeriksa Pengguna Saat Ini

Berguna untuk mengetahui identitas & peran pemilik token.

### Request

```
GET /api/me
Authorization: Bearer {token}
Accept: application/json
```

### Response 200

Mengembalikan data pengguna beserta relasi `roles`:

```json
{
  "id": 1,
  "name": "Administrator",
  "email": "admin@erp.test",
  "is_active": true,
  "roles": [
    {
      "id": 2,
      "name": "admin",
      "guard_name": "web"
    }
  ],
  "created_at": "2026-06-01T00:00:00.000000Z",
  "updated_at": "2026-06-01T00:00:00.000000Z"
}
```

---

## Logout (Mencabut Token)

Mencabut token yang sedang dipakai (hanya token saat ini, bukan semua token pengguna).

### Request

```
POST /api/logout
Authorization: Bearer {token}
Accept: application/json
```

### Response 200

```json
{ "message": "Logged out" }
```

Setelah logout, token tersebut tidak bisa lagi dipakai (request berikutnya → 401).

---

## Akun Demo

Untuk pengujian, sistem menyediakan akun demo (password semuanya `password`):

| Email | Peran | Cocok untuk menguji |
| --- | --- | --- |
| `superadmin@erp.test` | Super Admin | Semua endpoint |
| `admin@erp.test` | Admin | Semua endpoint |
| `finance-manager@erp.test` | Manajer Keuangan | Penjualan, Pembelian, Akuntansi |
| `accountant@erp.test` | Akuntan | Lihat dokumen + Akuntansi |
| `sales@erp.test` | Staf Penjualan | Endpoint Penjualan |
| `purchasing@erp.test` | Staf Pembelian | Endpoint Pembelian |
| `cashier@erp.test` | Kasir | Pembayaran & penerimaan |
| `viewer@erp.test` | Pembaca Laporan | Hanya endpoint `view` |

> Gunakan akun dengan peran terbatas untuk menguji respons **403 Forbidden** saat permission tidak terpenuhi.

---

## Catatan Keamanan

- **Jangan** menyimpan token di tempat yang tidak aman atau membagikannya. Token setara dengan kredensial login.
- Token tidak kedaluwarsa otomatis kecuali dicabut via logout (atau dihapus admin). Cabut token yang tidak terpakai.
- Selalu gunakan **HTTPS** di production agar token tidak bocor di jaringan.
