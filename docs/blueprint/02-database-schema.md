# 02 — Database Schema

All schema definitions are derived from reading every migration file in `database/migrations/`.

**Conventions across all tables:**
- Primary key: `id` (unsigned big integer, auto-increment) unless noted
- Timestamps: `created_at`, `updated_at` (nullable timestamps)
- Audit columns: `created_by`, `updated_by`, `deleted_by` (unsigned big integer, nullable) — present on every business table
- Soft deletes: `deleted_at` (nullable timestamp) — present on financial models

---

## 1. System & Framework Tables

### `sessions`
Session storage for authenticated users.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | string | No | — | Session identifier (PK) |
| user_id | unsignedBigInteger | Yes | — | FK to users.id |
| ip_address | string(45) | Yes | — | Client IP |
| user_agent | text | Yes | — | Browser user agent |
| payload | longText | No | — | Serialized session data |
| last_activity | integer | No | — | Unix timestamp of last activity |

**Indexes:** user_id, last_activity

### `cache` / `cache_locks`
Application cache storage.

| Column | Type | Description |
|--------|------|-------------|
| key | string (PK) | Cache key |
| value | mediumText | Cached value |
| expiration | bigInteger | Expiry timestamp |

### `jobs` / `job_batches` / `failed_jobs`
Queue worker tables (Laravel default).

### `password_reset_tokens`
| Column | Type | Description |
|--------|------|-------------|
| email | string (PK) | User email |
| token | string | Reset token |
| created_at | timestamp | Token creation time |

### `personal_access_tokens`
Sanctum API tokens.

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| id | bigInteger | No | PK |
| tokenable_type | string | No | Polymorphic model type |
| tokenable_id | unsignedBigInteger | No | Polymorphic model ID |
| name | string | No | Token name |
| token | string(64) | No | Hashed token (unique) |
| abilities | text | Yes | JSON abilities |
| last_used_at | timestamp | Yes | Last usage time |
| expires_at | timestamp | Yes | Expiry time |

---

## 2. Permission Tables (Spatie)

### `permissions`
| Column | Type | Description |
|--------|------|-------------|
| id | bigInteger | PK |
| name | string | Permission name (e.g., `sale-invoice.create`) |
| guard_name | string | Auth guard (default: `web`) |

**Unique:** (name, guard_name)

### `roles`
| Column | Type | Description |
|--------|------|-------------|
| id | bigInteger | PK |
| name | string | Role name (admin, staff, viewer) |
| guard_name | string | Auth guard |

**Unique:** (name, guard_name)

### `model_has_roles`
Pivot assigning roles to users (polymorphic).

| Column | Type | Description |
|--------|------|-------------|
| role_id | unsignedBigInteger | FK to roles.id (cascade delete) |
| model_type | string | `App\Models\User` |
| model_id | unsignedBigInteger | User ID |

### `model_has_permissions`
Direct permission assignments (polymorphic).

### `role_has_permissions`
Pivot assigning permissions to roles.

| Column | Type | Description |
|--------|------|-------------|
| permission_id | unsignedBigInteger | FK to permissions.id (cascade delete) |
| role_id | unsignedBigInteger | FK to roles.id (cascade delete) |

---

## 3. Core Tables

### `users`
System user accounts.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| name | string | No | — | Display name |
| email | string | No | — | Login email (unique) |
| email_verified_at | timestamp | Yes | — | Email verification date |
| password | string | No | — | Bcrypt-hashed password |
| remember_token | string | Yes | — | Remember-me token |
| created_by | unsignedBigInteger | Yes | — | Audit: creator |
| updated_by | unsignedBigInteger | Yes | — | Audit: last editor |
| deleted_by | unsignedBigInteger | Yes | — | Audit: deleter |
| deleted_at | timestamp | Yes | — | Soft delete |

**Unique:** email. **Soft deletes:** Yes.

### `currencies`
Supported currencies.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| name | string | No | — | Currency name (e.g., "Indonesian Rupiah") |
| code | string(4) | No | — | ISO code (e.g., "IDR") — unique |
| symbol | string(10) | No | — | Display symbol (e.g., "Rp") |

**Unique:** code. **Indexes:** name, code. **Soft deletes:** Yes.

### `exchange_rates`
Daily exchange rate history.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| currency_code | string(4) | No | — | ISO currency code |
| exchange_rate | decimal(13,9) | No | — | Rate relative to base currency |
| date | date | No | — | Effective date |

**Indexes:** currency_code, date. **Soft deletes:** No.

### `branches`
Company branch/business locations.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| name | string | No | — | Branch name |
| code | string | Yes | — | Short code |
| address | string | Yes | — | Street address |
| city | string | Yes | — | City |
| country | string | Yes | — | Country |
| phone_number | string | Yes | — | Phone |
| email | string | Yes | — | Email |
| website | string | Yes | — | Website URL |
| is_primary | boolean | No | false | Primary branch flag |

**Soft deletes:** Yes.

### `warehouses`
Inventory storage locations. Same structure as branches with `is_primary` flag.

**Soft deletes:** Yes.

### `settings`
Key-value configuration storage.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| user_id | unsignedBigInteger | Yes | — | FK to users.id (user-specific settings) |
| group | string | No | — | Setting group (e.g., "organization") |
| type | string | Yes | — | Value type hint |
| key | string | No | — | Setting key |
| value | text | Yes | — | Setting value |

**Indexes:** group, key, compound (group, key). **FK:** user_id → users.id.

---

## 4. Master Data Tables

### `accounts`
Chart of accounts with hierarchical parent-child structure.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| name | string | No | — | Account name |
| slug | string | Yes | — | URL-safe slug |
| account_type | string | No | — | AccountType enum value |
| code | string(20) | Yes | — | Account code (e.g., "11001") |
| description | text | Yes | — | Account description |
| parent_account_id | unsignedBigInteger | Yes | — | FK to accounts.id (self-ref) |
| is_active | boolean | No | true | Active status |
| index | unsignedInteger | No | 0 | Sort order |
| is_predefined | boolean | No | false | System-seeded account |
| balance | decimal(15,5) | No | 0 | Current balance |
| currency_code | string(4) | Yes | — | Multi-currency support |
| seeded_at | date | Yes | — | Seeding date |
| is_system_account | boolean | No | false | Core system account flag |

**FK:** parent_account_id → accounts.id (null on delete). **Indexes:** name, account_type, code, is_active, is_predefined, currency_code. **Soft deletes:** Yes.

### `contacts`
Unified customer and vendor directory.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| contact_service | string | No | — | ContactType: 'customer' or 'vendor' |
| display_name | string | No | — | Display name |
| salutation | string | Yes | — | Mr/Mrs/etc. |
| first_name | string | Yes | — | First name |
| last_name | string | Yes | — | Last name |
| company_name | string | Yes | — | Company name |
| email | string | Yes | — | Email address |
| work_phone | string | Yes | — | Work phone |
| personal_phone | string | Yes | — | Personal phone |
| website | string | Yes | — | Website URL |
| balance | decimal(15,5) | No | 0 | Outstanding balance |
| currency_code | string(4) | Yes | — | Preferred currency |
| opening_balance | decimal(15,5) | No | 0 | Opening balance amount |
| opening_balance_at | date | Yes | — | Opening balance date |
| opening_balance_branch_id | unsignedBigInteger | Yes | — | FK to branches.id |
| opening_balance_exchange_rate | decimal(13,9) | No | 1 | Opening balance rate |
| billing_address_1 | string | Yes | — | Billing line 1 |
| billing_address_2 | string | Yes | — | Billing line 2 |
| billing_address_city | string | Yes | — | Billing city |
| billing_address_country | string | Yes | — | Billing country |
| billing_address_email | string | Yes | — | Billing email |
| billing_address_postcode | string | Yes | — | Billing postcode |
| billing_address_phone | string | Yes | — | Billing phone |
| billing_address_state | string | Yes | — | Billing state |
| shipping_address_1 | string | Yes | — | Shipping line 1 |
| shipping_address_2 | string | Yes | — | Shipping line 2 |
| shipping_address_city | string | Yes | — | Shipping city |
| shipping_address_country | string | Yes | — | Shipping country |
| shipping_address_email | string | Yes | — | Shipping email |
| shipping_address_postcode | string | Yes | — | Shipping postcode |
| shipping_address_phone | string | Yes | — | Shipping phone |
| shipping_address_state | string | Yes | — | Shipping state |
| note | text | Yes | — | Internal notes |
| is_active | boolean | No | true | Active status |

**FK:** opening_balance_branch_id → branches.id. **Indexes:** contact_service. **Soft deletes:** Yes.

### `tax_rates`
Tax rate definitions.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| name | string | No | — | Tax name (e.g., "PPN 11%") |
| code | string | Yes | — | Short code (e.g., "PPN-11") |
| rate | decimal(8,4) | No | 0 | Tax percentage |
| description | string | Yes | — | Description |
| is_non_recoverable | boolean | No | false | Non-recoverable flag |
| is_compound | boolean | No | false | Compound tax flag |
| is_active | boolean | No | true | Active status |

**Soft deletes:** Yes.

### `item_categories`
Product/service groupings.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| name | string | No | — | Category name |
| description | text | Yes | — | Description |
| user_id | unsignedBigInteger | Yes | — | FK to users.id |
| cost_account_id | unsignedBigInteger | Yes | — | FK to accounts.id |
| sell_account_id | unsignedBigInteger | Yes | — | FK to accounts.id |
| inventory_account_id | unsignedBigInteger | Yes | — | FK to accounts.id |
| cost_method | string | Yes | — | Costing method |

**Indexes:** name.

### `items`
Products and services master data.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| name | string | No | — | Item name |
| type | string | No | — | ItemType: 'service', 'inventory', 'non-inventory' |
| code | string | Yes | — | SKU/code |
| is_sellable | boolean | No | true | Can be sold |
| is_purchasable | boolean | No | true | Can be purchased |
| sell_price | decimal(15,5) | No | 0 | Default sell price |
| cost_price | decimal(15,5) | No | 0 | Default cost price |
| picture_uri | string | Yes | — | Image path |
| cost_account_id | unsignedBigInteger | Yes | — | FK to accounts.id |
| sell_account_id | unsignedBigInteger | Yes | — | FK to accounts.id |
| inventory_account_id | unsignedBigInteger | Yes | — | FK to accounts.id |
| sell_description | text | Yes | — | Sales description |
| purchase_description | text | Yes | — | Purchase description |
| quantity_on_hand | decimal(15,5) | No | 0 | Current stock quantity |
| is_landed_cost | boolean | No | false | Landed cost item flag |
| note | text | Yes | — | Notes |
| is_active | boolean | No | true | Active status |
| category_id | unsignedBigInteger | Yes | — | FK to item_categories.id |
| user_id | unsignedBigInteger | Yes | — | FK to users.id |
| sell_tax_rate_id | unsignedBigInteger | Yes | — | FK to tax_rates.id |
| purchase_tax_rate_id | unsignedBigInteger | Yes | — | FK to tax_rates.id |

**Indexes:** name, type, is_sellable, is_purchasable. **Soft deletes:** Yes.

---

## 5. Accounting Tables

### `manual_journals`
User-created journal entries.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| journal_number | string | Yes | — | Sequential number (e.g., "J-00001") |
| reference | string | Yes | — | External reference |
| journal_type | string | Yes | — | Journal type classification |
| amount | decimal(15,5) | No | 0 | Total amount (sum of credits) |
| currency_code | string(4) | Yes | — | Transaction currency |
| exchange_rate | decimal(13,9) | No | 1 | Exchange rate to base |
| date | date | No | — | Journal date |
| description | text | Yes | — | Description/memo |
| published_at | date | Yes | — | Publication date (null = draft) |
| user_id | unsignedBigInteger | Yes | — | FK to users.id |
| branch_id | unsignedBigInteger | Yes | — | FK to branches.id |

**Indexes:** journal_number, reference, journal_type, date, published_at. **Soft deletes:** Yes.

### `manual_journal_entries`
Individual debit/credit lines of a journal.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| credit | decimal(15,5) | No | 0 | Credit amount |
| debit | decimal(15,5) | No | 0 | Debit amount |
| index | unsignedInteger | No | 0 | Line order |
| account_id | unsignedBigInteger | No | — | FK to accounts.id |
| contact_id | unsignedBigInteger | Yes | — | FK to contacts.id |
| note | text | Yes | — | Line note |
| manual_journal_id | unsignedBigInteger | No | — | FK to manual_journals.id (cascade delete) |
| branch_id | unsignedBigInteger | Yes | — | FK to branches.id |
| project_id | unsignedBigInteger | Yes | — | FK to projects.id |

### `account_transactions`
The General Ledger. Every financial event records balanced debit/credit entries here.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| credit | decimal(15,5) | No | 0 | Credit amount (base currency) |
| debit | decimal(15,5) | No | 0 | Debit amount (base currency) |
| currency_code | string(4) | Yes | — | Original currency |
| exchange_rate | decimal(13,9) | No | 1 | Exchange rate used |
| transaction_type | string | No | — | TransactionType enum |
| reference_type | string | Yes | — | Source model class |
| reference_id | unsignedBigInteger | Yes | — | Source record ID |
| account_id | unsignedBigInteger | No | — | FK to accounts.id |
| contact_type | string | Yes | — | Contact type |
| contact_id | unsignedBigInteger | Yes | — | FK to contacts.id |
| transaction_number | string | Yes | — | Document number |
| reference_number | string | Yes | — | External reference |
| item_id | unsignedBigInteger | Yes | — | FK to items.id |
| item_quantity | unsignedInteger | Yes | — | Item quantity |
| note | text | Yes | — | Transaction note |
| user_id | unsignedBigInteger | Yes | — | FK to users.id |
| index_group | unsignedInteger | Yes | — | Grouping index for multi-line entries |
| index | unsignedInteger | No | 0 | Line order within group |
| date | date | No | — | Transaction date |
| is_costable | boolean | No | false | Affects inventory cost |
| branch_id | unsignedBigInteger | Yes | — | FK to branches.id |
| project_id | unsignedBigInteger | Yes | — | FK to projects.id |
| tax_rate_id | unsignedBigInteger | Yes | — | FK to tax_rates.id |
| tax_rate | decimal(8,4) | No | 0 | Applied tax rate |

**FK:** account_id → accounts.id, item_id → items.id, user_id → users.id, branch_id → branches.id, tax_rate_id → tax_rates.id. **Indexes:** transaction_type, reference_type, reference_id, contact_type, contact_id, transaction_number, reference_number, index_group, date, created_at. **Soft deletes:** No.

---

## 6. Finance A/R Tables

### `sale_estimates`
Price quotations for customers.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| amount | decimal(15,5) | No | 0 | Total amount |
| currency_code | string(4) | Yes | — | Currency |
| exchange_rate | decimal(13,9) | No | 1 | Exchange rate |
| customer_id | unsignedBigInteger | No | — | FK to contacts.id |
| estimate_date | date | No | — | Estimate date |
| expiration_date | date | Yes | — | Expiry date |
| reference | string | Yes | — | External reference |
| estimate_number | string | Yes | — | Sequential number |
| note | text | Yes | — | Notes |
| terms_conditions | text | Yes | — | Terms |
| send_to_email | string | Yes | — | Recipient email |
| delivered_at | date | Yes | — | Delivery date |
| approved_at | date | Yes | — | Approval date |
| rejected_at | date | Yes | — | Rejection date |
| user_id | unsignedBigInteger | Yes | — | FK to users.id |
| converted_to_invoice_id | unsignedBigInteger | Yes | — | FK to sale_invoices.id |
| converted_to_invoice_at | date | Yes | — | Conversion date |
| warehouse_id | unsignedBigInteger | Yes | — | FK to warehouses.id |
| branch_id | unsignedBigInteger | Yes | — | FK to branches.id |
| discount | decimal(10,2) | Yes | — | Document-level discount |
| discount_type | string | Yes | — | 'percentage' or 'amount' |
| adjustment | decimal(10,2) | Yes | — | Manual adjustment |

**Soft deletes:** Yes.

### `sale_invoices`
Customer invoices.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| customer_id | unsignedBigInteger | No | — | FK to contacts.id |
| invoice_date | date | No | — | Invoice date |
| due_date | date | Yes | — | Payment due date |
| invoice_no | string | Yes | — | Sequential number |
| reference_no | string | Yes | — | External reference |
| invoice_message | text | Yes | — | Customer message |
| terms_conditions | text | Yes | — | Terms |
| balance | decimal(15,5) | No | 0 | Original total amount |
| payment_amount | decimal(15,5) | No | 0 | Total payments received |
| currency_code | string(4) | Yes | — | Currency |
| exchange_rate | decimal(13,9) | No | 1 | Exchange rate |
| credited_amount | decimal(15,5) | No | 0 | Applied credit notes |
| delivered_at | date | Yes | — | Delivery date (null = draft) |
| user_id | unsignedBigInteger | Yes | — | FK to users.id |
| warehouse_id | unsignedBigInteger | Yes | — | FK to warehouses.id |
| branch_id | unsignedBigInteger | Yes | — | FK to branches.id |
| writtenoff_amount | decimal(15,5) | No | 0 | Written-off amount |
| writtenoff_at | date | Yes | — | Write-off date |
| writtenoff_expense_account_id | unsignedBigInteger | Yes | — | FK to accounts.id |
| is_inclusive_tax | boolean | No | false | Tax-inclusive pricing |
| tax_amount_withheld | decimal(15,5) | No | 0 | Withheld tax |
| project_id | unsignedBigInteger | Yes | — | Project reference |
| discount | decimal(10,2) | Yes | — | Document discount |
| discount_type | string | Yes | — | 'percentage' or 'amount' |
| adjustment | decimal(10,2) | Yes | — | Manual adjustment |

**Soft deletes:** Yes.

### `payment_receives`
Customer payment records.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| customer_id | unsignedBigInteger | No | — | FK to contacts.id |
| payment_date | date | No | — | Payment date |
| amount | decimal(15,5) | No | 0 | Total payment amount |
| currency_code | string(4) | Yes | — | Currency |
| exchange_rate | decimal(13,9) | No | 1 | Exchange rate |
| reference_no | string | Yes | — | Reference number |
| deposit_account_id | unsignedBigInteger | No | — | FK to accounts.id (Cash/Bank) |
| payment_receive_no | string | Yes | — | Sequential number |
| statement | text | Yes | — | Statement memo |
| user_id | unsignedBigInteger | Yes | — | FK to users.id |
| branch_id | unsignedBigInteger | Yes | — | FK to branches.id |

**Soft deletes:** Yes.

### `payment_receive_entries`
Payment distribution to invoices.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| payment_receive_id | unsignedBigInteger | No | — | FK to payment_receives.id (cascade delete) |
| invoice_id | unsignedBigInteger | No | — | FK to sale_invoices.id |
| payment_amount | decimal(15,5) | No | 0 | Amount applied to this invoice |
| index | unsignedInteger | No | 0 | Line order |

### `credit_notes`
Customer credit notes.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| customer_id | unsignedBigInteger | No | — | FK to contacts.id |
| credit_note_date | date | No | — | Credit note date |
| credit_note_number | string | Yes | — | Sequential number |
| reference_no | string | Yes | — | External reference |
| amount | decimal(15,5) | No | 0 | Total credit amount |
| refunded_amount | decimal(15,5) | No | 0 | Cash refunded |
| invoices_amount | decimal(15,5) | No | 0 | Applied to invoices |
| currency_code | string(4) | Yes | — | Currency |
| exchange_rate | decimal(13,9) | No | 1 | Exchange rate |
| note | text | Yes | — | Notes |
| terms_conditions | text | Yes | — | Terms |
| opened_at | date | Yes | — | Open date (null = draft) |
| discount | decimal(10,2) | Yes | — | Document discount |
| discount_type | string | Yes | — | Discount type |
| adjustment | decimal(10,2) | Yes | — | Adjustment |

**Soft deletes:** Yes.

### `credit_note_applied_invoices`
Pivot: credit note applications to invoices.

| Column | Type | Description |
|--------|------|-------------|
| id | bigInteger | PK |
| amount | decimal(15,5) | Applied amount |
| credit_note_id | unsignedBigInteger | FK to credit_notes.id (cascade delete) |
| invoice_id | unsignedBigInteger | FK to sale_invoices.id |

### `refund_credit_note_transactions`
Cash refunds from credit notes.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| date | date | No | — | Refund date |
| credit_note_id | unsignedBigInteger | No | — | FK to credit_notes.id (cascade delete) |
| amount | decimal(15,5) | No | 0 | Refund amount |
| currency_code | string(4) | Yes | — | Currency |
| exchange_rate | decimal(13,9) | No | 1 | Exchange rate |
| reference_no | string | Yes | — | Reference |
| from_account_id | unsignedBigInteger | No | — | FK to accounts.id (payment source) |
| description | text | Yes | — | Description |
| branch_id | unsignedBigInteger | Yes | — | FK to branches.id |

**Soft deletes:** Yes.

---

## 7. Finance A/P Tables

### `bills`
Vendor purchase bills.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| vendor_id | unsignedBigInteger | No | — | FK to contacts.id |
| bill_number | string | Yes | — | Sequential number |
| bill_date | date | No | — | Bill date |
| due_date | date | Yes | — | Payment due date |
| reference_no | string | Yes | — | Vendor reference |
| status | string | No | 'draft' | BillStatus enum |
| note | text | Yes | — | Notes |
| amount | decimal(15,5) | No | 0 | Total amount |
| currency_code | string(4) | Yes | — | Currency |
| exchange_rate | decimal(13,9) | No | 1 | Exchange rate |
| payment_amount | decimal(15,5) | No | 0 | Total payments made |
| landed_cost_amount | decimal(15,5) | No | 0 | Landed costs |
| allocated_cost_amount | decimal(15,5) | No | 0 | Allocated costs |
| credited_amount | decimal(15,5) | No | 0 | Applied vendor credits |
| opened_at | date | Yes | — | Open date (null = draft) |
| is_inclusive_tax | boolean | No | false | Tax-inclusive pricing |
| tax_amount_withheld | decimal(15,5) | No | 0 | Withheld tax |
| discount | decimal(10,2) | Yes | — | Document discount |
| discount_type | string | Yes | — | Discount type |
| adjustment | decimal(10,2) | Yes | — | Adjustment |

**Soft deletes:** Yes.

### `bill_payments`
Vendor payment records.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| vendor_id | unsignedBigInteger | No | — | FK to contacts.id |
| amount | decimal(15,5) | No | 0 | Total payment |
| currency_code | string(4) | Yes | — | Currency |
| exchange_rate | decimal(13,9) | No | 1 | Exchange rate |
| payment_account_id | unsignedBigInteger | No | — | FK to accounts.id (Cash/Bank) |
| payment_number | string | Yes | — | Sequential number |
| payment_date | date | No | — | Payment date |
| payment_method | string | Yes | — | Payment method |
| reference | string | Yes | — | External reference |
| statement | text | Yes | — | Statement memo |

**Soft deletes:** Yes.

### `bill_payment_entries`
Payment distribution to bills.

| Column | Type | Description |
|--------|------|-------------|
| id | bigInteger | PK |
| bill_payment_id | unsignedBigInteger | FK to bill_payments.id (cascade delete) |
| bill_id | unsignedBigInteger | FK to bills.id |
| payment_amount | decimal(15,5) | Amount applied to this bill |
| index | unsignedInteger | Line order |

### `vendor_credits`
Vendor credit notes. Structure mirrors credit_notes but for the AP side.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| vendor_id | unsignedBigInteger | No | — | FK to contacts.id |
| amount | decimal(15,5) | No | 0 | Total credit amount |
| currency_code | string(4) | Yes | — | Currency |
| exchange_rate | decimal(13,9) | No | 1 | Exchange rate |
| vendor_credit_date | date | No | — | Credit date |
| vendor_credit_number | string | Yes | — | Sequential number |
| reference_no | string | Yes | — | Reference |
| refunded_amount | decimal(15,5) | No | 0 | Cash refunded |
| invoiced_amount | decimal(15,5) | No | 0 | Applied to bills |
| note | text | Yes | — | Notes |
| opened_at | date | Yes | — | Open date |

**Soft deletes:** Yes.

### `vendor_credit_applied_bills`
Pivot: vendor credit applications to bills.

| Column | Type | Description |
|--------|------|-------------|
| id | bigInteger | PK |
| amount | decimal(15,5) | Applied amount |
| vendor_credit_id | unsignedBigInteger | FK to vendor_credits.id (cascade delete) |
| bill_id | unsignedBigInteger | FK to bills.id |

### `refund_vendor_credit_transactions`
Cash refunds from vendor credits. Structure mirrors refund_credit_note_transactions.

**Soft deletes:** Yes.

### `expenses`
Direct expense/charge records.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| currency_code | string(4) | Yes | — | Currency |
| exchange_rate | decimal(13,9) | No | 1 | Exchange rate |
| description | text | Yes | — | Description |
| payment_account_id | unsignedBigInteger | No | — | FK to accounts.id (Cash/Bank) |
| payee_id | unsignedBigInteger | Yes | — | FK to contacts.id (vendor) |
| reference_no | string | Yes | — | Reference |
| total_amount | decimal(15,5) | No | 0 | Total expense amount |
| landed_cost_amount | decimal(15,5) | No | 0 | Landed costs |
| allocated_cost_amount | decimal(15,5) | No | 0 | Allocated costs |
| published_at | date | Yes | — | Publication date |
| payment_date | date | No | — | Expense date |

**Soft deletes:** Yes.

### `expense_categories`
Expense line item allocation to accounts.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| expense_account_id | unsignedBigInteger | No | — | FK to accounts.id |
| index | unsignedInteger | No | 0 | Line order |
| description | text | Yes | — | Line description |
| amount | decimal(15,5) | No | 0 | Line amount |
| allocated_cost_amount | decimal(15,5) | No | 0 | Allocated costs |
| is_landed_cost | boolean | No | false | Landed cost flag |
| expense_id | unsignedBigInteger | No | — | FK to expenses.id (cascade delete) |
| project_id | unsignedBigInteger | Yes | — | FK to projects.id |

---

## 8. Shared / Polymorphic Tables

### `item_entries`
Polymorphic line items shared across invoices, bills, estimates, credit notes, receipts, and vendor credits.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | bigInteger | No | auto | PK |
| reference_type | string | No | — | Parent model class |
| reference_id | unsignedBigInteger | No | — | Parent record ID |
| index | unsignedInteger | No | 0 | Line order |
| item_id | unsignedBigInteger | Yes | — | FK to items.id |
| description | text | Yes | — | Line description |
| discount | unsignedInteger | No | 0 | Discount value |
| discount_type | string | No | 'percentage' | Discount type |
| quantity | decimal(15,5) | No | 1 | Quantity |
| rate | decimal(15,5) | No | 0 | Unit price |
| sell_account_id | unsignedBigInteger | Yes | — | FK to accounts.id |
| cost_account_id | unsignedBigInteger | Yes | — | FK to accounts.id |
| is_landed_cost | boolean | No | false | Landed cost line |
| allocated_cost_amount | decimal(15,5) | No | 0 | Allocated cost |
| warehouse_id | unsignedBigInteger | Yes | — | FK to warehouses.id |
| project_id | unsignedBigInteger | Yes | — | FK to projects.id |
| is_inclusive_tax | boolean | No | false | Tax-inclusive price |
| tax_rate_id | unsignedBigInteger | Yes | — | FK to tax_rates.id |
| tax_rate | decimal(8,4) | No | 0 | Applied tax rate |

**Indexes:** reference_type, reference_id (compound).

### `tax_rate_transactions`
Tax application tracking.

| Column | Type | Description |
|--------|------|-------------|
| id | bigInteger | PK |
| tax_rate_id | unsignedBigInteger | FK to tax_rates.id |
| reference_type | string | Source model |
| reference_id | unsignedBigInteger | Source record ID |
| rate | decimal(8,4) | Applied rate |
| tax_account_id | unsignedBigInteger | FK to accounts.id |

---

## 9. Supporting Tables

### Inventory Tables

- **`inventory_transactions`** — Stock movement records (IN/OUT direction, item, quantity, rate)
- **`inventory_cost_lot_tracker`** — FIFO/lot cost tracking for inventory valuation
- **`inventory_adjustments`** — Manual stock adjustments (increment/decrement) with soft deletes
- **`inventory_adjustment_entries`** — Adjustment line items (item, quantity, cost, value)
- **`item_warehouse_quantities`** — Pivot: item stock by warehouse. Unique (item_id, warehouse_id).

### Cashflow Tables

- **`cashflow_transactions`** — Cash management entries (owner drawing, contribution, transfers)
- **`cashflow_transaction_lines`** — Cashflow entry line items

### Project Tables

- **`projects`** — Project master (name, contact, deadline, cost estimate, status)
- **`tasks`** — Project tasks with charge type and hour tracking
- **`time_entries`** — Time tracking entries linked to tasks and projects

### Document Tables

- **`documents`** — File metadata (key, mime_type, size, original_name)
- **`document_links`** — Polymorphic: links documents to any business entity

### Warehouse Transfer Tables

- **`warehouse_transfers`** — Inter-warehouse stock transfers with soft deletes
- **`warehouse_transfer_entries`** — Transfer line items (item, quantity, cost)

### Landed Cost Tables

- **`landed_costs`** — Cost allocation records linked to bills
- **`landed_cost_entries`** — Landed cost distribution entries

### Sale Receipts

- **`sale_receipts`** — Point-of-sale receipts (immediate payment without invoice)

---

## Complete Foreign Key Reference

```
users.id                              ← sessions.user_id
users.id                              ← settings.user_id
users.id                              ← item_categories.user_id
users.id                              ← items.user_id
users.id                              ← sale_estimates.user_id
users.id                              ← sale_invoices.user_id
users.id                              ← sale_receipts.customer_id (via contacts)
users.id                              ← payment_receives.user_id
users.id                              ← bills.user_id
users.id                              ← bill_payments.user_id
users.id                              ← manual_journals.user_id
users.id                              ← expenses.user_id
users.id                              ← account_transactions.user_id
users.id                              ← inventory_adjustments.user_id
users.id                              ← cashflow_transactions.user_id

accounts.id                           ← accounts.parent_account_id (self-referential)
accounts.id                           ← item_categories.cost_account_id
accounts.id                           ← item_categories.sell_account_id
accounts.id                           ← item_categories.inventory_account_id
accounts.id                           ← items.cost_account_id
accounts.id                           ← items.sell_account_id
accounts.id                           ← items.inventory_account_id
accounts.id                           ← sale_invoices.writtenoff_expense_account_id
accounts.id                           ← sale_receipts.deposit_account_id
accounts.id                           ← payment_receives.deposit_account_id
accounts.id                           ← bill_payments.payment_account_id
accounts.id                           ← expenses.payment_account_id
accounts.id                           ← expense_categories.expense_account_id
accounts.id                           ← manual_journal_entries.account_id
accounts.id                           ← account_transactions.account_id
accounts.id                           ← item_entries.sell_account_id
accounts.id                           ← item_entries.cost_account_id
accounts.id                           ← cashflow_transactions.cashflow_account_id
accounts.id                           ← cashflow_transactions.credit_account_id
accounts.id                           ← cashflow_transaction_lines.cashflow_account_id
accounts.id                           ← cashflow_transaction_lines.credit_account_id
accounts.id                           ← refund_credit_note_transactions.from_account_id
accounts.id                           ← refund_vendor_credit_transactions.deposit_account_id
accounts.id                           ← inventory_adjustments.adjustment_account_id
accounts.id                           ← inventory_transactions.cost_account_id
accounts.id                           ← landed_costs.cost_account_id
accounts.id                           ← tax_rate_transactions.tax_account_id

contacts.id                           ← sale_estimates.customer_id
contacts.id                           ← sale_invoices.customer_id
contacts.id                           ← sale_receipts.customer_id
contacts.id                           ← payment_receives.customer_id
contacts.id                           ← credit_notes.customer_id
contacts.id                           ← bills.vendor_id
contacts.id                           ← bill_payments.vendor_id
contacts.id                           ← vendor_credits.vendor_id
contacts.id                           ← expenses.payee_id
contacts.id                           ← manual_journal_entries.contact_id
contacts.id                           ← projects.contact_id

branches.id                           ← contacts.opening_balance_branch_id
branches.id                           ← sale_estimates.branch_id
branches.id                           ← sale_invoices.branch_id
branches.id                           ← sale_receipts.branch_id
branches.id                           ← payment_receives.branch_id
branches.id                           ← bills.branch_id
branches.id                           ← bill_payments.branch_id
branches.id                           ← credit_notes.branch_id
branches.id                           ← vendor_credits.branch_id
branches.id                           ← expenses.branch_id
branches.id                           ← manual_journals.branch_id
branches.id                           ← manual_journal_entries.branch_id
branches.id                           ← account_transactions.branch_id
branches.id                           ← cashflow_transactions.branch_id
branches.id                           ← refund_credit_note_transactions.branch_id
branches.id                           ← refund_vendor_credit_transactions.branch_id

warehouses.id                         ← sale_estimates.warehouse_id
warehouses.id                         ← sale_invoices.warehouse_id
warehouses.id                         ← sale_receipts.warehouse_id
warehouses.id                         ← bills.warehouse_id
warehouses.id                         ← credit_notes.warehouse_id
warehouses.id                         ← vendor_credits.warehouse_id
warehouses.id                         ← item_entries.warehouse_id
warehouses.id                         ← inventory_adjustments.warehouse_id
warehouses.id                         ← inventory_transactions.warehouse_id
warehouses.id                         ← warehouse_transfers.from_warehouse_id
warehouses.id                         ← warehouse_transfers.to_warehouse_id
warehouses.id                         ← item_warehouse_quantities.warehouse_id

items.id                              ← item_entries.item_id
items.id                              ← account_transactions.item_id
items.id                              ← inventory_adjustment_entries.item_id
items.id                              ← inventory_transactions.item_id
items.id                              ← warehouse_transfer_entries.item_id
items.id                              ← item_warehouse_quantities.item_id

item_categories.id                    ← items.category_id

tax_rates.id                          ← items.sell_tax_rate_id
tax_rates.id                          ← items.purchase_tax_rate_id
tax_rates.id                          ← item_entries.tax_rate_id
tax_rates.id                          ← account_transactions.tax_rate_id
tax_rates.id                          ← tax_rate_transactions.tax_rate_id

sale_invoices.id                      ← payment_receive_entries.invoice_id
sale_invoices.id                      ← credit_note_applied_invoices.invoice_id

payment_receives.id                   ← payment_receive_entries.payment_receive_id

credit_notes.id                       ← credit_note_applied_invoices.credit_note_id
credit_notes.id                       ← refund_credit_note_transactions.credit_note_id

bills.id                              ← bill_payment_entries.bill_id
bills.id                              ← vendor_credit_applied_bills.bill_id
bills.id                              ← landed_costs.bill_id

bill_payments.id                      ← bill_payment_entries.bill_payment_id

vendor_credits.id                     ← vendor_credit_applied_bills.vendor_credit_id
vendor_credits.id                     ← refund_vendor_credit_transactions.vendor_credit_id

manual_journals.id                    ← manual_journal_entries.manual_journal_id

expenses.id                           ← expense_categories.expense_id

projects.id                           ← tasks.project_id
projects.id                           ← time_entries.project_id

tasks.id                              ← time_entries.task_id

documents.id                          ← document_links.document_id

cashflow_transactions.id              ← cashflow_transaction_lines.cashflow_transaction_id

inventory_adjustments.id              ← inventory_adjustment_entries.adjustment_id

warehouse_transfers.id                ← warehouse_transfer_entries.warehouse_transfer_id

landed_costs.id                       ← landed_cost_entries.landed_cost_id

permissions.id                        ← model_has_permissions.permission_id
permissions.id                        ← role_has_permissions.permission_id

roles.id                              ← model_has_roles.role_id
roles.id                              ← role_has_permissions.role_id
```
