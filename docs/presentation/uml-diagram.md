# UML Diagrams — ERP Finance & Accounting

Semua diagram menggunakan sintaks **Mermaid** dan dapat di-render di GitHub, GitLab, Notion, atau tools Markdown lainnya.

---

## DIAGRAM 1 — Use Case Diagram

Aktor dan use case yang tersedia per modul.

```mermaid
flowchart TD
    Admin([👤 Admin])
    Staff([👤 Staff])
    Viewer([👤 Viewer])

    subgraph AUTH["🔐 Autentikasi"]
        UC1[Login ke Sistem]
        UC2[Logout]
    end

    subgraph DASHBOARD["📊 Dashboard"]
        UC3[Lihat Ringkasan Keuangan]
    end

    subgraph AR["💰 Finance — Piutang A/R"]
        UC4[Buat / Edit Invoice]
        UC5[Kirim Invoice ke Pelanggan]
        UC6[Write-Off Invoice]
        UC7[Hapus Invoice]
        UC8[Duplikasi Invoice]
        UC9[Buat / Edit Estimasi]
        UC10[Konversi Estimasi ke Invoice]
        UC11[Buat Nota Kredit]
        UC12[Terapkan Nota Kredit ke Invoice]
        UC13[Refund Nota Kredit]
        UC14[Catat Penerimaan Pembayaran]
        UC15[Hapus Penerimaan Pembayaran]
    end

    subgraph AP["🧾 Finance — Hutang A/P"]
        UC16[Buat / Edit Tagihan Bill]
        UC17[Buka Tagihan]
        UC18[Duplikasi Tagihan]
        UC19[Hapus Tagihan]
        UC20[Buat Kredit Vendor]
        UC21[Terapkan Kredit Vendor ke Tagihan]
        UC22[Catat Pembayaran Tagihan]
        UC23[Hapus Pembayaran Tagihan]
    end

    subgraph ACC["📒 Akuntansi"]
        UC24[Lihat Bagan Akun]
        UC25[Buat / Edit Akun]
        UC26[Hapus Akun]
        UC27[Buat / Edit Jurnal Manual]
        UC28[Posting Jurnal ke Buku Besar]
        UC29[Hapus Jurnal]
        UC30[Buat / Edit Pengeluaran]
        UC31[Posting Pengeluaran]
        UC32[Hapus Pengeluaran]
    end

    subgraph REPORT["📈 Laporan Keuangan"]
        UC33[Lihat Neraca]
        UC34[Lihat Laba Rugi]
        UC35[Lihat Neraca Saldo]
        UC36[Lihat Arus Kas]
        UC37[Lihat Aging Piutang]
        UC38[Lihat Aging Hutang]
        UC39[Lihat Ringkasan Pajak]
    end

    subgraph SETTINGS["⚙️ Pengaturan"]
        UC40[Kelola Pengaturan Organisasi]
        UC41[Kelola Mata Uang dan Kurs]
        UC42[Kelola Tarif Pajak]
        UC43[Kelola Item / Produk]
        UC44[Kelola Kontak Pelanggan dan Vendor]
        UC45[Kelola Pengguna]
    end

    %% Admin → semua use case
    Admin --> UC1 & UC2 & UC3
    Admin --> UC4 & UC5 & UC6 & UC7 & UC8
    Admin --> UC9 & UC10 & UC11 & UC12 & UC13 & UC14 & UC15
    Admin --> UC16 & UC17 & UC18 & UC19 & UC20 & UC21 & UC22 & UC23
    Admin --> UC24 & UC25 & UC26 & UC27 & UC28 & UC29 & UC30 & UC31 & UC32
    Admin --> UC33 & UC34 & UC35 & UC36 & UC37 & UC38 & UC39
    Admin --> UC40 & UC41 & UC42 & UC43 & UC44 & UC45

    %% Staff → buat/edit, tidak bisa hapus atau pengaturan
    Staff --> UC1 & UC2 & UC3
    Staff --> UC4 & UC5 & UC8 & UC9 & UC10 & UC11 & UC12 & UC13 & UC14
    Staff --> UC16 & UC17 & UC18 & UC20 & UC21 & UC22
    Staff --> UC24 & UC27 & UC28 & UC30 & UC31
    Staff --> UC33 & UC34 & UC35 & UC36 & UC37 & UC38 & UC39

    %% Viewer → lihat saja
    Viewer --> UC1 & UC2 & UC3
    Viewer --> UC33 & UC34 & UC35 & UC36 & UC37 & UC38 & UC39
    Viewer --> UC24
```

---

## DIAGRAM 2 — Entity Relationship Diagram (ERD)

Entitas utama dan relasi antar tabel database.

```mermaid
erDiagram
    users {
        bigint id PK
        string name
        string email UK
        string password
        bigint created_by
        bigint updated_by
        timestamp deleted_at
    }

    roles {
        bigint id PK
        string name UK
        string guard_name
    }

    permissions {
        bigint id PK
        string name UK
        string guard_name
    }

    currencies {
        bigint id PK
        string name
        string code UK
        string symbol
        timestamp deleted_at
    }

    exchange_rates {
        bigint id PK
        bigint currency_id FK
        decimal rate
        date date
    }

    accounts {
        bigint id PK
        string name
        string account_type
        string code
        bigint parent_account_id FK
        boolean is_active
        boolean is_predefined
        boolean is_system_account
        decimal balance
        string currency_code
        timestamp deleted_at
    }

    contacts {
        bigint id PK
        string contact_service
        string display_name
        string email
        decimal balance
        string currency_code
        timestamp deleted_at
    }

    tax_rates {
        bigint id PK
        string name
        decimal rate
        boolean is_active
        timestamp deleted_at
    }

    items {
        bigint id PK
        string name
        string type
        decimal sell_price
        decimal cost_price
        bigint sell_account_id FK
        bigint cost_account_id FK
        bigint inventory_account_id FK
        timestamp deleted_at
    }

    sale_invoices {
        bigint id PK
        bigint customer_id FK
        date invoice_date
        date due_date
        string invoice_no
        decimal balance
        decimal payment_amount
        decimal credited_amount
        string currency_code
        decimal exchange_rate
        date delivered_at
        date writtenoff_at
        timestamp deleted_at
    }

    sale_estimates {
        bigint id PK
        bigint customer_id FK
        date estimate_date
        date expiry_date
        string estimate_number
        decimal amount
        date delivered_at
        date approved_at
        date converted_at
        timestamp deleted_at
    }

    payment_receives {
        bigint id PK
        bigint customer_id FK
        bigint deposit_account_id FK
        date payment_date
        decimal amount
        string currency_code
        decimal exchange_rate
        timestamp deleted_at
    }

    payment_receive_entries {
        bigint id PK
        bigint payment_receive_id FK
        bigint invoice_id FK
        decimal amount
    }

    credit_notes {
        bigint id PK
        bigint customer_id FK
        string credit_note_number
        date credit_note_date
        decimal amount
        decimal invoices_amount
        decimal refunded_amount
        date opened_at
        timestamp deleted_at
    }

    bills {
        bigint id PK
        bigint vendor_id FK
        string bill_number
        date bill_date
        date due_date
        decimal amount
        decimal payment_amount
        decimal credited_amount
        string currency_code
        decimal exchange_rate
        date opened_at
        timestamp deleted_at
    }

    bill_payments {
        bigint id PK
        bigint vendor_id FK
        bigint payment_account_id FK
        date payment_date
        decimal amount
        string currency_code
        timestamp deleted_at
    }

    bill_payment_entries {
        bigint id PK
        bigint bill_payment_id FK
        bigint bill_id FK
        decimal amount
    }

    vendor_credits {
        bigint id PK
        bigint vendor_id FK
        string vendor_credit_number
        decimal amount
        decimal bills_amount
        decimal refunded_amount
        date opened_at
        timestamp deleted_at
    }

    item_entries {
        bigint id PK
        string reference_type
        bigint reference_id
        bigint item_id FK
        string description
        decimal quantity
        decimal rate
        decimal amount
        bigint tax_rate_id FK
        bigint account_id FK
    }

    manual_journals {
        bigint id PK
        string journal_number
        date date
        decimal amount
        string currency_code
        date published_at
        timestamp deleted_at
    }

    manual_journal_entries {
        bigint id PK
        bigint manual_journal_id FK
        bigint account_id FK
        decimal debit
        decimal credit
        string description
    }

    expenses {
        bigint id PK
        bigint payment_account_id FK
        bigint payee_id FK
        date payment_date
        decimal total_amount
        string currency_code
        date published_at
        timestamp deleted_at
    }

    expense_categories {
        bigint id PK
        bigint expense_id FK
        bigint account_id FK
        decimal amount
        string description
    }

    account_transactions {
        bigint id PK
        bigint account_id FK
        decimal debit
        decimal credit
        string transaction_type
        string reference_type
        bigint reference_id
        string currency_code
        decimal exchange_rate
        date date
    }

    branches {
        bigint id PK
        string name
        string code
    }

    settings {
        bigint id PK
        string group
        string key
        string value
    }

    %% Relasi
    users ||--o{ roles : "model_has_roles"
    roles ||--o{ permissions : "role_has_permissions"
    currencies ||--o{ exchange_rates : "has many"
    accounts ||--o{ accounts : "parent → children"
    accounts ||--o{ account_transactions : "has many"
    contacts ||--o{ sale_invoices : "customer"
    contacts ||--o{ bills : "vendor"
    contacts ||--o{ payment_receives : "customer"
    contacts ||--o{ bill_payments : "vendor"
    contacts ||--o{ credit_notes : "customer"
    contacts ||--o{ vendor_credits : "vendor"
    sale_invoices ||--o{ item_entries : "polymorphic"
    sale_invoices ||--o{ payment_receive_entries : "has many"
    sale_estimates ||--o{ item_entries : "polymorphic"
    payment_receives ||--o{ payment_receive_entries : "has many"
    payment_receive_entries }o--|| sale_invoices : "belongs to"
    bills ||--o{ item_entries : "polymorphic"
    bills ||--o{ bill_payment_entries : "has many"
    bill_payments ||--o{ bill_payment_entries : "has many"
    bill_payment_entries }o--|| bills : "belongs to"
    items ||--o{ item_entries : "has many"
    tax_rates ||--o{ item_entries : "has many"
    manual_journals ||--o{ manual_journal_entries : "has many"
    manual_journal_entries }o--|| accounts : "belongs to"
    expenses ||--o{ expense_categories : "has many"
    expense_categories }o--|| accounts : "belongs to"
```

---

## DIAGRAM 3 — Class Diagram (Model Layer)

Relasi antar Eloquent Model.

```mermaid
classDiagram
    class User {
        +string name
        +string email
        +getRoleNames()
        +getAllPermissions()
        +syncRoles()
    }

    class Contact {
        +string contact_service
        +string display_name
        +decimal balance
        +scopeCustomer()
        +scopeVendor()
    }

    class Account {
        +string name
        +string account_type
        +string code
        +boolean is_predefined
        +boolean is_system_account
        +scopeActive()
        +scopeBalanceSheet()
    }

    class SaleInvoice {
        +string invoice_no
        +decimal balance
        +decimal payment_amount
        +date delivered_at
        +getStatusAttribute()
        +isDraft()
        +balanceDue()
    }

    class SaleEstimate {
        +string estimate_number
        +date delivered_at
        +date approved_at
        +date converted_at
        +getStatusAttribute()
    }

    class PaymentReceive {
        +decimal amount
        +date payment_date
        +string currency_code
    }

    class PaymentReceiveEntry {
        +decimal amount
    }

    class CreditNote {
        +string credit_note_number
        +decimal amount
        +decimal invoices_amount
        +decimal refunded_amount
        +date opened_at
        +getStatusAttribute()
    }

    class Bill {
        +string bill_number
        +decimal amount
        +decimal payment_amount
        +date opened_at
        +getStatusAttribute()
        +isDraft()
        +balanceDue()
    }

    class BillPayment {
        +decimal amount
        +date payment_date
    }

    class BillPaymentEntry {
        +decimal amount
    }

    class VendorCredit {
        +string vendor_credit_number
        +decimal amount
        +decimal bills_amount
        +date opened_at
        +getStatusAttribute()
    }

    class ItemEntry {
        +string reference_type
        +bigint reference_id
        +decimal quantity
        +decimal rate
        +decimal amount
    }

    class ManualJournal {
        +string journal_number
        +decimal amount
        +date published_at
        +getStatusAttribute()
    }

    class ManualJournalEntry {
        +decimal debit
        +decimal credit
        +string description
    }

    class Expense {
        +decimal total_amount
        +date payment_date
        +date published_at
        +getStatusAttribute()
    }

    class ExpenseCategory {
        +decimal amount
        +string description
    }

    class AccountTransaction {
        +decimal debit
        +decimal credit
        +string transaction_type
        +string reference_type
        +bigint reference_id
        +date date
        +scopeOfType()
        +scopeBetweenDates()
    }

    class Currency {
        +string code
        +string symbol
    }

    class ExchangeRate {
        +decimal rate
        +date date
    }

    class TaxRate {
        +string name
        +decimal rate
        +boolean is_active
    }

    class Item {
        +string name
        +string type
        +decimal sell_price
        +decimal cost_price
    }

    class Branch {
        +string name
        +string code
    }

    %% User ↔ Roles (Spatie)
    User "1" --> "*" SaleInvoice : creates
    User "1" --> "*" Bill : creates

    %% Contact relationships
    Contact "1" --> "*" SaleInvoice : customer
    Contact "1" --> "*" SaleEstimate : customer
    Contact "1" --> "*" PaymentReceive : customer
    Contact "1" --> "*" CreditNote : customer
    Contact "1" --> "*" Bill : vendor
    Contact "1" --> "*" BillPayment : vendor
    Contact "1" --> "*" VendorCredit : vendor

    %% Invoice relationships
    SaleInvoice "1" --> "*" ItemEntry : entries (polymorphic)
    SaleInvoice "1" --> "*" PaymentReceiveEntry : paymentEntries
    PaymentReceive "1" --> "*" PaymentReceiveEntry : entries
    PaymentReceiveEntry "*" --> "1" SaleInvoice : invoice

    %% Bill relationships
    Bill "1" --> "*" ItemEntry : entries (polymorphic)
    Bill "1" --> "*" BillPaymentEntry : paymentEntries
    BillPayment "1" --> "*" BillPaymentEntry : entries
    BillPaymentEntry "*" --> "1" Bill : bill

    %% Credit Note relationships
    CreditNote "1" --> "*" ItemEntry : entries (polymorphic)

    %% Vendor Credit relationships
    VendorCredit "1" --> "*" ItemEntry : entries (polymorphic)

    %% Estimate relationships
    SaleEstimate "1" --> "*" ItemEntry : entries (polymorphic)
    SaleEstimate "1" --> "0..1" SaleInvoice : convertedTo

    %% Account relationships
    Account "1" --> "*" AccountTransaction : transactions
    Account "0..1" --> "*" Account : parent→children
    Account "1" --> "*" Item : sell/cost/inventory

    %% Journal relationships
    ManualJournal "1" --> "*" ManualJournalEntry : entries
    ManualJournalEntry "*" --> "1" Account : account

    %% Expense relationships
    Expense "1" --> "*" ExpenseCategory : categories
    ExpenseCategory "*" --> "1" Account : account
    Expense "*" --> "1" Account : paymentAccount

    %% Currency
    Currency "1" --> "*" ExchangeRate : rates

    %% Item & Tax
    Item "*" --> "1" TaxRate : sellTaxRate
    Item "*" --> "1" TaxRate : purchaseTaxRate
    ItemEntry "*" --> "1" Item : item
    ItemEntry "*" --> "1" TaxRate : taxRate
```

---

## DIAGRAM 4 — Sequence Diagram: Alur Invoice Lengkap

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant Controller as InvoiceController
    participant Service as SaleInvoiceService
    participant DB as Database

    Note over User,DB: === 1. Buat Invoice Draft ===
    User->>Browser: Isi form invoice (customer, items, tanggal)
    Browser->>Controller: POST /invoices (X-Inertia header)
    Controller->>Service: create(validated_data)
    Service->>DB: Validasi invoice_no unik (whereNull deleted_at)
    Service->>DB: INSERT sale_invoices (balance=total, payment_amount=0)
    Service->>DB: INSERT item_entries (polymorphic per baris item)
    Service-->>Controller: return $invoice
    Controller-->>Browser: redirect → invoices/{id} + flash "Invoice created"
    Browser-->>User: Tampilkan halaman invoice + toast sukses

    Note over User,DB: === 2. Kirim / Deliver Invoice ===
    User->>Browser: Klik tombol "Deliver"
    Browser->>Controller: POST /invoices/{id}/deliver
    Controller->>Service: deliver($id)
    Service->>DB: Cek invoice belum delivered (DocumentAlreadyDeliveredException?)
    Service->>DB: UPDATE sale_invoices SET delivered_at = now()
    Note right of Service: Buat GL Entries (double-entry)
    Service->>DB: INSERT account_transactions (DEBIT Piutang Usaha 12001)
    Service->>DB: INSERT account_transactions (CREDIT Pendapatan per baris)
    Service->>DB: INSERT account_transactions (CREDIT PPN Keluaran 22001)
    Service-->>Controller: return $invoice
    Controller-->>Browser: redirect → invoices/{id} + flash "Delivered"
    Browser-->>User: Status invoice = Unpaid

    Note over User,DB: === 3. Terima Pembayaran ===
    User->>Browser: Buat Payment Receive → pilih invoice ini
    Browser->>Controller: POST /payment-receives
    Controller->>Service: PaymentReceiveService.create(data)
    Service->>DB: Validasi deposit account = Cash/Bank
    Service->>DB: Validasi payment amount ≤ invoice balance due
    Service->>DB: INSERT payment_receives
    Service->>DB: INSERT payment_receive_entries (invoice_id, amount)
    Service->>DB: UPDATE sale_invoices SET payment_amount += amount
    Note right of Service: GL Entries pembayaran
    Service->>DB: INSERT account_transactions (DEBIT Kas/Bank)
    Service->>DB: INSERT account_transactions (CREDIT Piutang Usaha 12001)
    Service->>DB: Fire PaymentReceived event
    Service-->>Controller: return $payment
    Controller-->>Browser: redirect + flash "Payment recorded"
    
    Note over User,DB: === 4. Status Invoice Menjadi Paid ===
    DB-->>Browser: balance - payment_amount - credited_amount <= 0
    Browser-->>User: Status invoice = PAID ✓
```

---

## DIAGRAM 5 — Sequence Diagram: Double-Entry Accounting Engine

```mermaid
sequenceDiagram
    participant Service as Service Layer
    participant GL as account_transactions
    participant AR as Akun Piutang (12001)
    participant Revenue as Akun Pendapatan (41001)
    participant Tax as Akun PPN Keluaran (22001)
    participant Cash as Akun Kas/Bank
    participant AP as Akun Hutang (21001)
    participant Expense as Akun Beban

    Note over Service,Expense: === Invoice Delivered (Penjualan) ===
    Service->>GL: INSERT DEBIT 1.000.000 → Akun Piutang (12001)
    Service->>GL: INSERT CREDIT 909.090  → Akun Pendapatan (41001)
    Service->>GL: INSERT CREDIT 90.910   → Akun PPN Keluaran (22001)
    Note right of GL: Total Debit = Total Credit ✓ (1.000.000)

    Note over Service,Expense: === Payment Received (Terima Pembayaran) ===
    Service->>GL: INSERT DEBIT 1.000.000 → Akun Kas/Bank
    Service->>GL: INSERT CREDIT 1.000.000 → Akun Piutang (12001)
    Note right of GL: Piutang berkurang, Kas bertambah ✓

    Note over Service,Expense: === Bill Opened (Tagihan Masuk) ===
    Service->>GL: INSERT DEBIT 500.000   → Akun Beban
    Service->>GL: INSERT DEBIT 55.000    → Akun PPN Masukan
    Service->>GL: INSERT CREDIT 555.000  → Akun Hutang (21001)
    Note right of GL: Total Debit = Total Credit ✓ (555.000)

    Note over Service,Expense: === Bill Payment (Bayar Tagihan) ===
    Service->>GL: INSERT DEBIT 555.000   → Akun Hutang (21001)
    Service->>GL: INSERT CREDIT 555.000  → Akun Kas/Bank
    Note right of GL: Hutang berkurang, Kas berkurang ✓

    Note over Service,Expense: === GL Reversal saat dokumen dihapus ===
    Service->>GL: DELETE WHERE reference_type='SaleInvoice' AND reference_id=42
    Note right of GL: Semua jurnal terkait invoice #42 dihapus
```

---

## DIAGRAM 6 — State Diagram: Invoice Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Draft : Buat invoice baru

    Draft --> Unpaid : Deliver Invoice\n(delivered_at = now())\n[GL entries dibuat]

    Unpaid --> PartiallyPaid : Terima pembayaran parsial\n(0 < payment_amount < balance)

    PartiallyPaid --> Paid : Terima pembayaran penuh\n(payment_amount + credited_amount >= balance)

    Unpaid --> Paid : Terima pembayaran penuh langsung

    Unpaid --> WrittenOff : Write-Off invoice\n(writtenoff_at = now())\n[GL: DEBIT BadDebt, CREDIT AR]

    PartiallyPaid --> WrittenOff : Write-Off sisa piutang

    WrittenOff --> Unpaid : Cancel Write-Off\n[GL entries dihapus]

    note right of Draft
        delivered_at = NULL
        Bisa diedit atau dihapus
    end note

    note right of Unpaid
        delivered_at SET
        payment_amount = 0
        credited_amount = 0
    end note

    note right of PartiallyPaid
        payment_amount > 0
        tapi < balance
    end note

    note right of Paid
        payment_amount + credited_amount
        + writtenoff_amount >= balance
    end note
```

---

## DIAGRAM 7 — State Diagram: Bill Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Draft : Buat tagihan baru

    Draft --> Unpaid : Buka Tagihan (Open)\n(opened_at = now())\n[GL entries dibuat:\nDEBIT Beban, CREDIT Hutang]

    Unpaid --> Overdue : Tanggal jatuh tempo terlewati\n(due_date < today)\n[Status computed]

    Overdue --> PartiallyPaid : Bayar sebagian\n(0 < payment_amount < amount)

    Unpaid --> PartiallyPaid : Bayar sebagian

    PartiallyPaid --> Paid : Bayar penuh\n(payment_amount + credited_amount >= amount)

    Unpaid --> Paid : Bayar penuh langsung

    Overdue --> Paid : Bayar penuh meskipun terlambat

    note right of Draft
        opened_at = NULL
        Bisa diedit atau dihapus
    end note

    note right of Unpaid
        opened_at SET
        payment_amount = 0
        Belum melewati due_date
    end note

    note right of Overdue
        Computed: due_date < today
        dan masih ada saldo
    end note

    note right of Paid
        payment_amount + credited_amount
        >= amount
    end note
```

---

## DIAGRAM 8 — Component Diagram: Arsitektur Sistem

```mermaid
flowchart TB
    subgraph CLIENT["🌐 Browser (Client)"]
        React["React 19\n+ TypeScript"]
        Inertia_Client["Inertia.js Client Adapter"]
        TailwindUI["Tailwind CSS\nUI Components"]
        React --> Inertia_Client
        React --> TailwindUI
    end

    subgraph SERVER["🖥️ Laravel Server"]
        subgraph MIDDLEWARE["Middleware Layer"]
            Sanctum["Laravel Sanctum\n(Auth Session/Token)"]
            InertiaMiddleware["HandleInertiaRequests\n(Share auth, flash, settings)"]
            Permission["Spatie Permission\nMiddleware"]
        end

        subgraph HTTP["HTTP Layer"]
            FormRequest["Form Requests\n(Validation + Authorization)"]
            Controllers["Controllers\n(Inertia/JSON branching)"]
        end

        subgraph BUSINESS["Business Layer"]
            Services["Service Classes\n(17 services)\nBusiness rules, GL entries"]
            Events["Events & Listeners\n(PaymentReceived, BillOpened)"]
        end

        subgraph DATA["Data Layer"]
            Models["Eloquent Models\n(46 models)\nSoftDeletes + Auditable"]
            Cache["Laravel Cache\n(Settings, Permissions)"]
        end

        MIDDLEWARE --> HTTP
        FormRequest --> Controllers
        Controllers --> Services
        Services --> Events
        Services --> Models
        Models --> Cache
    end

    subgraph DB["🗄️ MySQL Database"]
        Tables["43+ Tables\nTransactions, Accounts,\nContacts, Users, GL"]
    end

    subgraph AUTH_LAYER["🔐 Authentication"]
        Session["Session Table\n(database driver)"]
        Token["personal_access_tokens\n(API consumers)"]
    end

    Inertia_Client <-->|"X-Inertia header\nJSON page props"| InertiaMiddleware
    Models <-->|"Eloquent ORM\nQueries"| Tables
    Sanctum <--> Session
    Sanctum <--> Token

    style CLIENT fill:#dbeafe,stroke:#3b82f6
    style SERVER fill:#f0fdf4,stroke:#22c55e
    style DB fill:#fef3c7,stroke:#f59e0b
    style AUTH_LAYER fill:#fce7f3,stroke:#ec4899
```

---

## DIAGRAM 9 — Activity Diagram: Alur Login dan Otorisasi

```mermaid
flowchart TD
    Start([Pengguna buka browser]) --> OpenLogin
    OpenLogin[Buka URL aplikasi /] --> CheckSession

    CheckSession{Session\naktif?}
    CheckSession -->|Ya| LoadDashboard[Redirect ke /dashboard]
    CheckSession -->|Tidak| ShowLogin[Tampilkan halaman Login]

    ShowLogin --> FillForm[Isi Email + Password]
    FillForm --> SubmitForm[Submit form]

    SubmitForm --> ValidateInput{Validasi\nformat input}
    ValidateInput -->|Gagal| ShowError1[Tampilkan error validasi\ndi form]
    ShowError1 --> FillForm

    ValidateInput -->|Lolos| CheckCredentials{Periksa credentials\ndi database}
    CheckCredentials -->|Tidak cocok| ShowError2[Tampilkan:\n'These credentials do not\nmatch our records.']
    ShowError2 --> FillForm

    CheckCredentials -->|Cocok| CreateSession[Buat session database\nRegenerate session ID]
    CreateSession --> LoadPermissions[Load permissions user\nfrom cache/database]
    LoadPermissions --> CachePermissions[Simpan ke cache\nuser.permissions.id\nTTL: 30 menit]

    CachePermissions --> CheckDashboardAccess[Request /dashboard]
    CheckDashboardAccess --> PermMiddleware{permission middleware\ncheck}

    PermMiddleware -->|Tidak punya akses| Show403[Tampilkan halaman 403]
    PermMiddleware -->|Lolos| BuildNavbar[Bangun sidebar navigation\nberdasarkan permissions]

    BuildNavbar --> RenderDashboard[Render Dashboard\ndengan data keuangan]
    RenderDashboard --> UserActive([Pengguna aktif di aplikasi])

    LoadDashboard --> CheckDashboardAccess

    style Start fill:#22c55e,color:#fff
    style UserActive fill:#22c55e,color:#fff
    style Show403 fill:#ef4444,color:#fff
    style ShowError1 fill:#f97316,color:#fff
    style ShowError2 fill:#f97316,color:#fff
```

---

*Semua diagram di atas menggunakan sintaks Mermaid dan dapat di-render langsung di:*
- *GitHub / GitLab Markdown*
- *Notion*
- *VS Code dengan ekstensi Mermaid*
- *mermaid.live (online editor)*
