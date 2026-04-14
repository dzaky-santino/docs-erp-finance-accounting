# 04 — Business Flows

All flows are derived from reading the service classes in `app/Services/`, controller classes in `app/Http/Controllers/`, and event/listener classes.

---

## 1. User Authentication Flow

### Login (Web / Inertia)
1. User navigates to `/auth/login` — server renders the login page via Inertia
2. User submits email + password via `useForm().post('/auth/login')`
3. `LoginRequest` validates: email (required, email format, max:255), password (required, max:255)
4. `AuthController@login` calls `Auth::attempt($credentials, remember: true)`
5. **Success:** Session regenerated → redirect to `/dashboard`
6. **Failure:** `ValidationException` thrown with message "These credentials do not match our records." → redirect to login route (explicit, not `back()`, due to URL masking)
7. Login page reads errors from `usePage().props.errors` and displays inline

### Login (API)
1. Client sends `POST /api/login` with email + password
2. `AuthController@apiLogin` validates and calls `Auth::attempt()`
3. **Success:** Returns JSON `{ user, token }` — token is a Sanctum personal access token
4. **Failure:** Returns 422 JSON with validation errors

### Logout
1. `POST /auth/logout` → `AuthController@logout`
2. Session invalidated, token regenerated
3. Redirect to `/auth/login`

---

## 2. Invoice Lifecycle Flow

### Create Draft
1. User fills invoice form: customer, date, due date, line items (item, quantity, rate, tax, discount)
2. `StoreInvoiceRequest` validates all fields including permission check (`sale-invoice.create`)
3. `SaleInvoiceService::create()`:
   - Validates invoice_no uniqueness (excluding soft-deleted, `whereNull('deleted_at')`)
   - Calculates total: sum(quantity * rate) - discount + adjustment
   - Creates `SaleInvoice` record with `balance` = total, `payment_amount` = 0
   - Creates polymorphic `ItemEntry` records for each line item
4. Controller redirects to invoice show page with flash success

### Deliver Invoice
1. User clicks "Deliver" on invoice show page
2. `SaleInvoiceService::deliver()`:
   - Validates not already delivered (`DocumentAlreadyDeliveredException`)
   - Sets `delivered_at = now()`
   - **Creates GL entries:**
     - DEBIT: Accounts Receivable (12001) — full local amount (amount * exchange_rate)
     - CREDIT: Each line item's sell_account — amount ex-tax per line
     - CREDIT: Tax Payable (22001) — tax amount per line (quantity * rate * tax_rate)
   - Dispatches `InvoiceDelivered` event
3. Invoice status computed as "Delivered" / "Unpaid"

### Record Payment
1. User creates a Payment Receive selecting this invoice
2. `PaymentReceiveService::create()`:
   - Validates deposit account is Cash/Bank/OtherCurrentAsset
   - Validates invoice is delivered (not draft)
   - Validates payment amount ≤ invoice balance due
   - Creates `PaymentReceive` + `PaymentReceiveEntry`
   - Increments `SaleInvoice.payment_amount`
   - **Creates GL entries:**
     - DEBIT: Deposit account (Cash/Bank)
     - CREDIT: Accounts Receivable
   - Dispatches `PaymentReceived` event
3. `UpdateInvoiceBalance` listener calls `recalculateBalance()` to sync

### Status Transitions
```
Draft → Delivered → Unpaid → PartiallyPaid → Paid
                                    ↘ WrittenOff
```
- **Draft:** `delivered_at` is null
- **Delivered/Unpaid:** `delivered_at` set, `payment_amount + credited_amount = 0`
- **PartiallyPaid:** `payment_amount + credited_amount > 0` but `< balance`
- **Paid:** `payment_amount + credited_amount + writtenoff_amount >= balance`
- **WrittenOff:** `writtenoff_at` is set

### Write-Off
1. `SaleInvoiceService::writeOff()`:
   - Calculates remaining = balance - payment_amount - credited_amount
   - Sets `writtenoff_amount`, `writtenoff_at`, `writtenoff_expense_account_id`
   - **GL entries:** DEBIT write-off expense account, CREDIT Accounts Receivable

### Cancel Write-Off
1. `SaleInvoiceService::cancelWriteOff()`:
   - Deletes write-off GL entries (where `transaction_type = InvoiceWriteOff`)
   - Clears writtenoff fields

### Duplicate
1. `SaleInvoiceService::duplicate()`:
   - Creates new draft invoice from existing data
   - Generates new invoice number via `getNextNumber()`
   - Copies all item entries
   - Excludes delivered_at, payment/credited/writtenoff amounts

---

## 3. Estimate to Invoice Conversion Flow

1. **Create Estimate:** User creates estimate with customer, date, line items
2. **Deliver:** `SaleEstimateService::deliver()` sets `delivered_at = now()`
3. **Approve:** `SaleEstimateService::approve()` validates delivered, sets `approved_at = now()`
4. **Convert:** `SaleEstimateService::convertToInvoice()`:
   - Validates not already converted (`EstimateAlreadyConvertedException`)
   - Maps estimate data to invoice format
   - Calls `SaleInvoiceService::create()` with the mapped data
   - Sets `converted_to_invoice_id` and `converted_to_invoice_at` on estimate
   - Estimate status becomes "Converted"
5. The new invoice starts in Draft status and follows the normal invoice lifecycle

**Alternative paths:**
- **Reject:** `SaleEstimateService::reject()` sets `rejected_at`, clears `approved_at`
- Estimates do NOT create GL entries at any stage

---

## 4. Payment Receive Flow

1. User navigates to Payment Receives > Create
2. Selects customer → system loads unpaid invoices for that customer
3. User allocates payment amounts to one or more invoices
4. `StorePaymentReceiveRequest` validates entries array, amounts, account
5. `PaymentReceiveService::create()`:
   - Validates deposit account type (must be Cash, Bank, or OtherCurrentAsset)
   - For each entry: validates invoice is delivered, payment ≤ invoice balance due
   - Creates `PaymentReceive` header
   - Creates `PaymentReceiveEntry` for each invoice allocation
   - Increments `SaleInvoice.payment_amount` for each invoice
   - **GL entries:**
     - DEBIT: Deposit account (Cash/Bank) — total payment * exchange_rate
     - CREDIT: Accounts Receivable — same amount
   - Dispatches `PaymentReceived` event
6. `UpdateInvoiceBalance` listener recalculates each affected invoice's balance

### Delete Payment
1. `PaymentReceiveService::delete()`:
   - Decrements `SaleInvoice.payment_amount` for each entry
   - Deletes GL entries
   - Deletes payment entries and payment record

---

## 5. Bill Lifecycle Flow

### Create Draft
1. `BillService::create()`:
   - Validates bill_number uniqueness
   - Calculates amount from entries
   - Creates `Bill` with `status = 'draft'`, `payment_amount = 0`
   - Creates `ItemEntry` records

### Open Bill
1. `BillService::open()`:
   - Validates not already opened (`DocumentAlreadyOpenedException`)
   - Sets `opened_at = now()`
   - **GL entries:**
     - CREDIT: Accounts Payable (21001) — full local amount
     - DEBIT: Each line item's cost_account — amount ex-tax per line
     - DEBIT: Tax Payable (22001) — tax amount per line
   - Dispatches `BillOpened` event

### Status Transitions
```
Draft → Opened → Unpaid → Overdue → PartiallyPaid → Paid
```
- **Draft:** `opened_at` is null
- **Opened/Unpaid:** `opened_at` set, no payments
- **Overdue:** Past due_date with outstanding balance
- **PartiallyPaid/Paid:** Based on payment_amount + credited_amount vs. amount

---

## 6. Bill Payment Flow

1. User selects vendor → system loads unpaid bills
2. Allocates payment amounts to bills
3. `BillPaymentService::create()`:
   - Validates payment account is Cash/Bank/OtherCurrentAsset
   - Validates each bill is opened and payment ≤ bill balance due
   - Creates `BillPayment` + `BillPaymentEntry` records
   - Increments `Bill.payment_amount`
   - **GL entries:**
     - DEBIT: Accounts Payable
     - CREDIT: Payment account (Cash/Bank)
   - Dispatches `BillPaymentRecorded` event
4. `UpdateBillBalance` listener recalculates each affected bill

---

## 7. Credit Note Flow

### Create
1. `CreditNoteService::create()`: Creates credit note with line items, calculates amount

### Open
2. `CreditNoteService::open()`:
   - Sets `opened_at = now()`
   - **GL entries:**
     - CREDIT: Accounts Receivable — total local amount
     - DEBIT: Each line item's sell_account — amount ex-tax

### Apply to Invoices
3. `CreditNoteService::applyToInvoices()`:
   - Validates total applications ≤ remaining balance
   - Validates each application ≤ invoice balance due
   - Creates `CreditNoteAppliedInvoice` records
   - Increments `SaleInvoice.credited_amount` for each invoice
   - Increments `CreditNote.invoices_amount`

### Refund
4. `CreditNoteService::refund()`:
   - Validates amount ≤ remaining balance
   - Creates `RefundCreditNote` record
   - Increments `CreditNote.refunded_amount`
   - **GL entries:** DEBIT refund account (Cash/Bank), CREDIT Accounts Receivable

### Status Flow
```
Draft → Open → Closed
         ↓
    (Apply to Invoices OR Refund)
         ↓
   Closed (when amount = invoices_amount + refunded_amount)
```

---

## 8. Vendor Credit Flow

Mirrors the Credit Note flow but on the AP side:

1. **Create:** Line items with cost accounts
2. **Open:** GL entries: DEBIT Accounts Payable, CREDIT each item's cost_account
3. **Apply to Bills:** Creates `VendorCreditAppliedBill`, increments `Bill.credited_amount`
4. **Refund:** CREDIT Accounts Payable, DEBIT deposit account

---

## 9. Manual Journal Entry Flow

### Create
1. User enters balanced debit/credit lines with accounts
2. `ManualJournalService::create()`:
   - Validates total debits = total credits (`JournalNotBalancedException`)
   - Validates totals > 0 (`JournalAmountZeroException`)
   - Validates journal_number uniqueness
   - Creates `ManualJournal` with `amount = sum(credits)`
   - Creates `ManualJournalEntry` records

### Publish
3. `ManualJournalService::publish()`:
   - Sets `published_at = now()`
   - **GL entries:** One `AccountTransaction` per journal entry line, preserving exact debit/credit amounts, accounts, contacts, notes, and branch/project references

### Update Published Journal
4. If entries change: reverts existing GL entries, updates entries, recreates GL entries

---

## 10. Double-Entry Accounting Engine

### How GL Entries Work

Every service that creates GL entries follows this pattern:

```php
AccountTransaction::create([
    'debit'            => $amount,     // or 0 for credit side
    'credit'           => 0,            // or $amount for credit side
    'account_id'       => $accountId,
    'transaction_type' => TransactionType::SaleInvoice,
    'reference_type'   => 'SaleInvoice',
    'reference_id'     => $invoice->id,
    'date'             => $invoice->invoice_date,
    'currency_code'    => $invoice->currency_code,
    'exchange_rate'    => $invoice->exchange_rate,
    'index_group'      => 10,           // groups related entries
]);
```

### GL Entry Patterns by Transaction Type

| Transaction | Debit Account(s) | Credit Account(s) |
|------------|-------------------|-------------------|
| **Invoice Delivered** | Accounts Receivable (12001) | Sales Revenue (41001/41002) per line + Tax Payable (22001) |
| **Payment Received** | Deposit Account (Cash/Bank) | Accounts Receivable (12001) |
| **Invoice Write-Off** | Bad Debt / Expense Account | Accounts Receivable (12001) |
| **Credit Note Opened** | Sales Revenue (per line) | Accounts Receivable (12001) |
| **Credit Note Refund** | Refund Account (Cash/Bank) | Accounts Receivable (12001) |
| **Bill Opened** | Expense/COGS (per line) + Tax Payable (22001) | Accounts Payable (21001) |
| **Bill Payment** | Accounts Payable (21001) | Payment Account (Cash/Bank) |
| **Vendor Credit Opened** | Accounts Payable (21001) | Expense/COGS (per line) |
| **Vendor Credit Refund** | Accounts Payable (21001) | Deposit Account (Cash/Bank) |
| **Expense Published** | Expense Account(s) per category | Payment Account (Cash/Bank) |
| **Manual Journal** | As specified per entry | As specified per entry (must balance) |

### Multi-Currency GL Entries
All amounts stored in `account_transactions` are in the **base currency**. The conversion: `localAmount = documentAmount * exchangeRate`. The original currency and rate are also stored for reference.

### GL Entry Reversal
When a document is deleted or updated, the service calls `revertGLEntries($documentId)` which deletes all `AccountTransaction` records matching the `reference_type` and `reference_id`.

---

## 11. Financial Reports Generation Flow

### Balance Sheet
1. `ReportService::balanceSheet($date)`:
   - Queries `account_transactions` where `date <= $date`
   - Groups by account, sums debits and credits
   - Calculates balance based on normal balance (debit accounts: debit - credit; credit accounts: credit - debit)
   - Groups accounts into Asset, Liability, Equity sections
   - Calculates net income and adds to equity total
   - Returns totals ensuring Assets = Liabilities + Equity

### Income Statement
1. `ReportService::incomeStatement($fromDate, $toDate)`:
   - Queries transactions within date range
   - Groups by income and expense root types
   - Returns total income, total expenses, net income (income - expenses)

### Trial Balance
1. `ReportService::trialBalance($fromDate, $toDate)`:
   - Shows all accounts with total debits and total credits for the period
   - Verifies total debits = total credits

### Cash Flow Statement
1. `ReportService::cashFlowStatement($fromDate, $toDate)`:
   - Filters transactions on Cash and Bank accounts
   - Classifies by transaction type into:
     - **Operating:** SaleInvoice, PaymentReceived, Bill, BillPayment, Expense
     - **Investing:** (asset-related transactions)
     - **Financing:** (equity/liability-related transactions)
   - Returns net cash flow per section

### Receivables Aging
1. `ReportService::receivablesAging($asOfDate, $customerId)`:
   - Queries delivered invoices with outstanding balance (balance - payment_amount - credited_amount > 0)
   - Calculates days past due using `DATEDIFF($asOfDate, due_date)` with parameterized bindings
   - Buckets: Current (≤0 days), 1-30, 31-60, 61-90, Over 90
   - Groups by contact

### Payables Aging
Same pattern as receivables but for bills where `opened_at` is not null.

### Tax Summary
1. `ReportService::taxSummary($fromDate, $toDate)`:
   - Tax collected: Credits to Tax Payable from SaleInvoice, SaleReceipt, CreditNote transactions
   - Tax paid: Debits to Tax Payable from Bill, VendorCredit, Expense transactions
   - Net tax payable: collected - paid

---

## 12. Chart of Accounts Setup Flow

### Create Account
1. `AccountService::create()`:
   - Validates name uniqueness (including soft-deleted via `withTrashed()`)
   - Validates code uniqueness (if provided, excluding soft-deleted)
   - If parent specified:
     - Validates parent type compatibility (child and parent must share the same root type: Asset, Liability, Equity, Income, or Expense)
     - Validates max depth (traverses parent chain, max 5 levels)
   - Inherits currency_code from parent if not specified
   - Generates slug from name
   - Sets `is_predefined = false`

### Account Hierarchy
```
Root Type: Asset
  └── Parent Type: Current Asset
       ├── Cash on Hand (11001) [Cash type]
       ├── Bank BCA - IDR (11101) [Bank type]
       └── Accounts Receivable (12001) [AR type]
           └── (child accounts possible, up to 5 levels deep)
```

### System Accounts
Certain accounts are flagged as `is_system_account = true` and `is_predefined = true`. These cannot be deleted (`PredefinedAccountException`). They are automatically used by services:
- **Accounts Receivable (12001)** — used by invoice and payment services
- **Accounts Payable (21001)** — used by bill and bill payment services
- **Tax Payable / PPN Keluaran (22001)** — used for tax GL entries
- **Sales Revenue - Products (41001)** — default sell account
- **COGS (51001)** — default cost account

### Delete Account
Cannot delete if:
- `is_predefined = true` → `PredefinedAccountException`
- Has GL transactions → `AccountHasTransactionsException`
- On delete: child accounts are unlinked (parent_account_id set to null)

---

## Events and Listeners

| Event | Listener | Action |
|-------|----------|--------|
| `InvoiceDelivered` | — | Fired but no listener (reserved for future use) |
| `PaymentReceived` | `UpdateInvoiceBalance` | Calls `SaleInvoiceService::recalculateBalance()` for each affected invoice |
| `BillOpened` | — | Fired but no listener (reserved for future use) |
| `BillPaymentRecorded` | `UpdateBillBalance` | Calls `BillService::recalculateBalance()` for each affected bill |
