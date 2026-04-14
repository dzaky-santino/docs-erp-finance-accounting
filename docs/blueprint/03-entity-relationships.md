# 03 — Entity Relationships

All relationships are derived from reading the 46 Eloquent model files in `app/Models/`.

Every model uses the `Auditable` trait (auto-populates `created_by`, `updated_by`, `deleted_by`).

---

## Model Relationship Reference

### User
**Table:** `users` | **Traits:** SoftDeletes, Auditable, HasApiTokens, HasRoles (Spatie)

No explicit relationships defined. Implicitly referenced by many models via `user_id` foreign keys and by Spatie's `model_has_roles` and `model_has_permissions` pivot tables.

---

### Account
**Table:** `accounts` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| parentAccount | belongsTo | Account | parent_account_id | Parent in the account hierarchy |
| childAccounts | hasMany | Account | parent_account_id | Children in the hierarchy |
| accountTransactions | hasMany | AccountTransaction | account_id | All GL entries for this account |

**Scopes:** `active()`, `ofType(AccountType)`, `default()`, `balanceSheet()`, `incomeStatement()`

**Casts:** `account_type` → AccountType enum, `balance` → decimal:5

---

### Contact
**Table:** `contacts` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| openingBalanceBranch | belongsTo | Branch | opening_balance_branch_id | Branch for opening balance |
| saleInvoices | hasMany | SaleInvoice | customer_id | Invoices where contact is customer |
| bills | hasMany | Bill | vendor_id | Bills where contact is vendor |
| saleEstimates | hasMany | SaleEstimate | customer_id | Estimates for this customer |

**Scopes:** `customer()`, `vendor()`, `active()`

**Casts:** `contact_service` → ContactType enum

---

### Currency
**Table:** `currencies` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| exchangeRates | hasMany | ExchangeRate | currency_code → code | Exchange rate history |

**Scopes:** `byCode(string)`

---

### ExchangeRate
**Table:** `exchange_rates` | **Traits:** Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| currency | belongsTo | Currency | currency_code → code | Parent currency |

**Scopes:** `latestRate(string)`, `onDate(string)`

---

### Item
**Table:** `items` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| costAccount | belongsTo | Account | cost_account_id | COGS/expense account |
| sellAccount | belongsTo | Account | sell_account_id | Revenue account |
| inventoryAccount | belongsTo | Account | inventory_account_id | Inventory asset account |
| category | belongsTo | ItemCategory | category_id | Product category |
| sellTaxRate | belongsTo | TaxRate | sell_tax_rate_id | Default sales tax |
| purchaseTaxRate | belongsTo | TaxRate | purchase_tax_rate_id | Default purchase tax |
| warehouseStock | belongsToMany | Warehouse | item_warehouse_quantities | Stock by warehouse (pivot: quantity_on_hand) |
| inventoryTransactions | hasMany | InventoryTransaction | item_id | Stock movement history |

**Scopes:** `active()`, `ofType(ItemType)`, `sellable()`, `purchasable()`, `inventory()`

**Casts:** `type` → ItemType enum

---

### TaxRate
**Table:** `tax_rates` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| taxTransactions | hasMany | TaxRateTransaction | tax_rate_id | Tax application records |

---

### SaleInvoice
**Table:** `sale_invoices` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| customer | belongsTo | Contact | customer_id | Invoice customer |
| warehouse | belongsTo | Warehouse | warehouse_id | Fulfillment warehouse |
| branch | belongsTo | Branch | branch_id | Issuing branch |
| creator | belongsTo | User | user_id | Created by user |
| writeOffAccount | belongsTo | Account | writtenoff_expense_account_id | Write-off expense account |
| itemEntries | morphMany | ItemEntry | reference_type/reference_id | Line items (polymorphic) |
| paymentEntries | hasMany | PaymentReceiveEntry | invoice_id | Payment allocations |
| creditNoteApplications | hasMany | CreditNoteAppliedInvoice | invoice_id | Applied credit notes |

**Scopes:** `delivered()`, `draft()`, `unpaid()`, `overdue()`

**Computed:** `status` (appended attribute derived from delivered_at, payment_amount, balance, writtenoff_at)

---

### SaleEstimate
**Table:** `sale_estimates` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| customer | belongsTo | Contact | customer_id | Estimate customer |
| warehouse | belongsTo | Warehouse | warehouse_id | Warehouse |
| branch | belongsTo | Branch | branch_id | Branch |
| creator | belongsTo | User | user_id | Created by |
| itemEntries | morphMany | ItemEntry | reference_type/reference_id | Line items |
| convertedInvoice | belongsTo | SaleInvoice | converted_to_invoice_id | Converted invoice |

**Computed:** `status` (from delivered_at, approved_at, rejected_at, converted_to_invoice_id)

---

### PaymentReceive
**Table:** `payment_receives` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| customer | belongsTo | Contact | customer_id | Paying customer |
| depositAccount | belongsTo | Account | deposit_account_id | Cash/Bank account |
| branch | belongsTo | Branch | branch_id | Branch |
| creator | belongsTo | User | user_id | Created by |
| paymentEntries | hasMany | PaymentReceiveEntry | payment_receive_id | Invoice allocations |

---

### PaymentReceiveEntry
**Table:** `payment_receive_entries` | **Traits:** Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| paymentReceive | belongsTo | PaymentReceive | payment_receive_id | Parent payment |
| invoice | belongsTo | SaleInvoice | invoice_id | Target invoice |

---

### CreditNote
**Table:** `credit_notes` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| customer | belongsTo | Contact | customer_id | Customer |
| warehouse | belongsTo | Warehouse | warehouse_id | Warehouse |
| branch | belongsTo | Branch | branch_id | Branch |
| creator | belongsTo | User | user_id | Created by |
| itemEntries | morphMany | ItemEntry | reference_type/reference_id | Line items |
| appliedInvoices | hasMany | CreditNoteAppliedInvoice | credit_note_id | Invoice applications |
| refundTransactions | hasMany | RefundCreditNote | credit_note_id | Cash refunds |

**Computed:** `status` (from opened_at, amount, refunded_amount, invoices_amount)

**Methods:** `remainingBalance()` = amount - refunded_amount - invoices_amount

---

### CreditNoteAppliedInvoice
**Table:** `credit_note_applied_invoices` | **Traits:** Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| creditNote | belongsTo | CreditNote | credit_note_id | Source credit note |
| invoice | belongsTo | SaleInvoice | invoice_id | Target invoice |

---

### RefundCreditNote
**Table:** `refund_credit_note_transactions` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| creditNote | belongsTo | CreditNote | credit_note_id | Source credit note |
| sourceAccount | belongsTo | Account | from_account_id | Refund payment account |
| branch | belongsTo | Branch | branch_id | Branch |

---

### Bill
**Table:** `bills` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| vendor | belongsTo | Contact | vendor_id | Bill vendor |
| warehouse | belongsTo | Warehouse | warehouse_id | Receiving warehouse |
| branch | belongsTo | Branch | branch_id | Branch |
| creator | belongsTo | User | user_id | Created by |
| itemEntries | morphMany | ItemEntry | reference_type/reference_id | Line items |
| paymentEntries | hasMany | BillPaymentEntry | bill_id | Payment allocations |
| vendorCreditApplications | hasMany | VendorCreditAppliedBill | bill_id | Applied vendor credits |

**Scopes:** `draft()`, `open()`, `unpaid()`, `overdue()`

**Methods:** `isDraft()`, `isOverdue()`, `balanceDue()`

---

### BillPayment
**Table:** `bill_payments` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| vendor | belongsTo | Contact | vendor_id | Payee vendor |
| paymentAccount | belongsTo | Account | payment_account_id | Cash/Bank account |
| branch | belongsTo | Branch | branch_id | Branch |
| creator | belongsTo | User | user_id | Created by |
| paymentEntries | hasMany | BillPaymentEntry | bill_payment_id | Bill allocations |

---

### BillPaymentEntry
**Table:** `bill_payment_entries` | **Traits:** Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| billPayment | belongsTo | BillPayment | bill_payment_id | Parent payment |
| bill | belongsTo | Bill | bill_id | Target bill |

---

### VendorCredit
**Table:** `vendor_credits` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| vendor | belongsTo | Contact | vendor_id | Vendor |
| branch | belongsTo | Branch | branch_id | Branch |
| warehouse | belongsTo | Warehouse | warehouse_id | Warehouse |
| creator | belongsTo | User | user_id | Created by |
| itemEntries | morphMany | ItemEntry | reference_type/reference_id | Line items |
| appliedBills | hasMany | VendorCreditAppliedBill | vendor_credit_id | Bill applications |
| refundTransactions | hasMany | RefundVendorCredit | vendor_credit_id | Cash refunds |

**Computed:** `status`, **Methods:** `remainingBalance()`

---

### ManualJournal
**Table:** `manual_journals` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| creator | belongsTo | User | user_id | Created by |
| branch | belongsTo | Branch | branch_id | Branch |
| entries | hasMany | ManualJournalEntry | manual_journal_id | Debit/credit lines |

**Computed:** `status` (from published_at)

---

### ManualJournalEntry
**Table:** `manual_journal_entries` | **Traits:** Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| manualJournal | belongsTo | ManualJournal | manual_journal_id | Parent journal |
| account | belongsTo | Account | account_id | GL account |
| contact | belongsTo | Contact | contact_id | Associated contact |
| branch | belongsTo | Branch | branch_id | Branch |

---

### Expense
**Table:** `expenses` | **Traits:** SoftDeletes, Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| paymentAccount | belongsTo | Account | payment_account_id | Cash/Bank account |
| payee | belongsTo | Contact | payee_id | Vendor/payee |
| branch | belongsTo | Branch | branch_id | Branch |
| creator | belongsTo | User | user_id | Created by |
| expenseCategories | hasMany | ExpenseCategory | expense_id | Cost allocation lines |

**Computed:** `status` (from published_at)

---

### ExpenseCategory
**Table:** `expense_categories` | **Traits:** Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| expenseAccount | belongsTo | Account | expense_account_id | Expense GL account |
| expense | belongsTo | Expense | expense_id | Parent expense |

---

### ItemEntry
**Table:** `item_entries` | **Traits:** Auditable (polymorphic)

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| parentDocument | morphTo | — | reference_type/reference_id | Parent document (invoice, bill, etc.) |
| item | belongsTo | Item | item_id | Product/service item |
| sellAccount | belongsTo | Account | sell_account_id | Revenue account |
| costAccount | belongsTo | Account | cost_account_id | COGS account |
| taxRate | belongsTo | TaxRate | tax_rate_id | Applied tax rate |
| warehouse | belongsTo | Warehouse | warehouse_id | Warehouse |

**Methods:** `calculateSubtotal()`, `calculateTax()`

---

### AccountTransaction
**Table:** `account_transactions` | **Traits:** Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| account | belongsTo | Account | account_id | GL account |
| item | belongsTo | Item | item_id | Related item |
| taxRate | belongsTo | TaxRate | tax_rate_id | Applied tax rate |
| branch | belongsTo | Branch | branch_id | Branch |
| user | belongsTo | User | user_id | Creating user |

**Scopes:** `ofType(TransactionType)`, `forAccount(int)`, `betweenDates(string, string)`, `debit()`, `credit()`

---

### Setting
**Table:** `settings` | **Traits:** Auditable

| Relationship | Type | Related Model | Foreign Key | Description |
|-------------|------|--------------|-------------|-------------|
| user | belongsTo | User | user_id | User-specific settings |

**Scopes:** `group(string)`, `key(string)`

---

### Additional Models (Supporting)

| Model | Table | Key Relationships |
|-------|-------|-------------------|
| Branch | branches | Referenced by many financial documents |
| Warehouse | warehouses | items (belongsToMany via pivot), referenced by documents |
| SaleReceipt | sale_receipts | customer, depositAccount, warehouse, branch, itemEntries (morphMany) |
| CashflowTransaction | cashflow_transactions | cashflowAccount, creditAccount, branch, user, lines (hasMany) |
| CashflowTransactionLine | cashflow_transaction_lines | cashflowTransaction, cashflowAccount, creditAccount |
| InventoryAdjustment | inventory_adjustments | adjustmentAccount, warehouse, branch, creator, entries (hasMany) |
| InventoryAdjustmentEntry | inventory_adjustment_entries | adjustment, item |
| InventoryTransaction | inventory_transactions | item, costAccount, warehouse, branch |
| Project | projects | contact, tasks (hasMany), timeEntries (hasMany) |
| Task | tasks | project, timeEntries (hasMany) |
| TimeEntry | time_entries | task, project |
| Document | documents | links (hasMany) |
| DocumentLink | document_links | document, ownerEntity (morphTo) |
| LandedCost | landed_costs | costAccount, bill, entries (hasMany) |
| LandedCostEntry | landed_cost_entries | landedCost |
| WarehouseTransfer | warehouse_transfers | sourceWarehouse, destinationWarehouse, entries (hasMany) |
| WarehouseTransferEntry | warehouse_transfer_entries | warehouseTransfer, item |
| VendorCreditAppliedBill | vendor_credit_applied_bills | vendorCredit, bill |
| RefundVendorCredit | refund_vendor_credit_transactions | vendorCredit, depositAccount, branch |
| TaxRateTransaction | tax_rate_transactions | taxRate, taxAccount |
| ItemCategory | item_categories | costAccount, sellAccount, inventoryAccount, items (hasMany) |

---

## High-Level Entity Relationship Summary

### User → Roles → Permissions
A `User` is assigned one or more `Roles` via Spatie's `model_has_roles` pivot. Each `Role` has many `Permissions` via `role_has_permissions`. Permission checks happen in Form Requests (`authorize()`) and Policies (`viewAny`, `view`, `create`, `update`, `delete`). The format is `module.action` (e.g., `sale-invoice.create`).

### Contact as Customer and Vendor
A single `Contact` model serves both roles, differentiated by `contact_service` ('customer' or 'vendor'). Customers are linked to invoices, estimates, payment receives, and credit notes. Vendors are linked to bills, bill payments, and vendor credits. This unified design allows a single contact to be both a customer and a vendor.

### Invoice → Items, Payments, Credits, GL
A `SaleInvoice` has polymorphic `ItemEntry` line items (shared with bills, estimates, etc.). It receives payments via `PaymentReceiveEntry` records and credit applications via `CreditNoteAppliedInvoice`. When delivered, the invoice service creates `AccountTransaction` entries debiting Accounts Receivable and crediting Sales Revenue + Tax Payable. The invoice's `balance`, `payment_amount`, and `credited_amount` fields track the outstanding balance.

### Bill → Items, Payments, Credits, GL
A `Bill` mirrors the invoice pattern on the AP side. It has polymorphic `ItemEntry` lines, receives payments via `BillPaymentEntry`, and credit applications via `VendorCreditAppliedBill`. When opened, GL entries debit Expense/COGS + Tax Payable and credit Accounts Payable.

### Account as the Center of Double-Entry
`Account` is the central entity in the accounting system. Every financial event creates balanced `AccountTransaction` records. The `account_transactions` table stores debit and credit amounts with the transaction type, reference to the source document (polymorphic via reference_type/reference_id), currency, exchange rate, and date. Financial reports are generated by querying and aggregating these transactions.

### Document → GL Entry Flow
All financial documents follow the same pattern:
1. **Create** as draft (no GL impact)
2. **Activate** (deliver/open/publish) → service creates `AccountTransaction` entries
3. **Pay** → payment service creates additional `AccountTransaction` entries
4. **Delete** → service reverts all GL entries, then deletes the document

The `reference_type` and `reference_id` on `AccountTransaction` link back to the originating document, enabling complete audit trails and GL reversal on deletion.
