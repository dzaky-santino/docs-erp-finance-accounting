# 05 — Module Pages Inventory

All pages are derived from reading `resources/js/pages/` and the web route definitions in `routes/web.php`.

Every authenticated page uses `AppLayout` as a persistent layout (never unmounts during navigation). Auth pages use their own layout. Print pages use `PrintLayout`.

---

## 1. Authentication Pages

### Login
- **File:** `resources/js/pages/auth/login.tsx`
- **Route:** `GET /auth/login`
- **Layout:** Self-contained (no AppLayout)
- **Purpose:** Authenticate users into the system
- **Data displayed:** Application logo, name, tagline
- **Actions:** Email + password form with client-side validation (blur + submit), password visibility toggle, server-side error display from `usePage().props.errors`
- **Access:** Public (unauthenticated)
- **Notes:** Self-registration disabled. Register route redirects here.

---

## 2. Dashboard

### Dashboard Index
- **File:** `resources/js/pages/dashboard/index.tsx`
- **Route:** `GET /dashboard`
- **Purpose:** Overview of the organization's financial position
- **Data displayed:** Total receivables, total payables, cash balance, monthly income/expense chart (Recharts), recent invoices table, recent bills table
- **Actions:** Navigation links to invoices, bills, reports
- **Access:** All authenticated users

---

## 3. Finance — Invoices

### Invoice List
- **File:** `resources/js/pages/finance/invoices/index.tsx`
- **Route:** `GET /finance/invoices`
- **Purpose:** Browse and manage all sale invoices
- **Data displayed:** DataTable with invoice number, customer name, date, due date, amount, status (StatusBadge), balance due
- **Actions:** Create new, search, filter by status/customer/date range, row actions (view, edit, delete with ConfirmDialog)
- **Access:** `sale-invoice.view`

### Invoice Create
- **File:** `resources/js/pages/finance/invoices/create.tsx`
- **Route:** `GET /finance/invoices/create`
- **Purpose:** Create a new sale invoice
- **Data displayed:** Form with customer select, dates, invoice number, currency, exchange rate, LineItemsTable for line items (item, description, quantity, rate, discount, tax), document discount, adjustment, notes, terms
- **Actions:** Submit (POST), cancel
- **Access:** `sale-invoice.create`

### Invoice Edit
- **File:** `resources/js/pages/finance/invoices/edit.tsx`
- **Route:** `GET /finance/invoices/{id}/edit`
- **Purpose:** Edit an existing draft or delivered invoice
- **Data displayed:** Pre-filled form with all invoice data
- **Actions:** Submit (PUT), cancel
- **Access:** `sale-invoice.edit`

### Invoice Show
- **File:** `resources/js/pages/finance/invoices/show.tsx`
- **Route:** `GET /finance/invoices/{id}`
- **Purpose:** View invoice details and take status actions
- **Data displayed:** Full invoice header, line items table, payment history, credit note applications, status badge, totals
- **Actions:** Deliver, Write Off (with account selection), Cancel Write-Off, Duplicate, Edit, Delete, Print link
- **Access:** `sale-invoice.view`

### Invoice Print
- **File:** `resources/js/pages/finance/invoices/print.tsx`
- **Route:** `GET /finance/invoices/{id}/print`
- **Layout:** PrintLayout
- **Purpose:** Print-optimized invoice view
- **Data displayed:** Invoice header, line items, totals in print-friendly format
- **Actions:** Back button, Print button (both hidden in print via `.no-print`)
- **Access:** `sale-invoice.view`

---

## 4. Finance — Estimates

### Estimate List
- **File:** `resources/js/pages/finance/estimates/index.tsx`
- **Route:** `GET /finance/estimates`
- **Purpose:** Browse and manage sale estimates/quotations
- **Data displayed:** DataTable with estimate number, customer, date, expiration, amount, status
- **Actions:** Create, search, filter by status/customer, row actions
- **Access:** `sale-estimate.view`

### Estimate Create
- **File:** `resources/js/pages/finance/estimates/create.tsx`
- **Route:** `GET /finance/estimates/create`
- **Purpose:** Create a new price quotation
- **Data displayed:** Form with customer, dates, estimate number, line items
- **Actions:** Submit, cancel
- **Access:** `sale-estimate.create`

### Estimate Edit
- **File:** `resources/js/pages/finance/estimates/edit.tsx`
- **Route:** `GET /finance/estimates/{id}/edit`
- **Purpose:** Edit an existing estimate
- **Access:** `sale-estimate.edit`

### Estimate Show
- **File:** `resources/js/pages/finance/estimates/show.tsx`
- **Route:** `GET /finance/estimates/{id}`
- **Purpose:** View estimate and take lifecycle actions
- **Actions:** Deliver, Approve, Reject, Convert to Invoice, Edit, Delete
- **Access:** `sale-estimate.view`

---

## 5. Finance — Credit Notes

### Credit Note List
- **File:** `resources/js/pages/finance/credit-notes/index.tsx`
- **Route:** `GET /finance/credit-notes`
- **Purpose:** Browse customer credit notes
- **Data displayed:** DataTable with credit note number, customer, date, amount, remaining balance, status
- **Access:** `credit-note.view`

### Credit Note Create
- **File:** `resources/js/pages/finance/credit-notes/create.tsx`
- **Route:** `GET /finance/credit-notes/create`
- **Purpose:** Issue a new credit note to a customer
- **Data displayed:** Form with customer, date, line items
- **Access:** `credit-note.create`

### Credit Note Show
- **File:** `resources/js/pages/finance/credit-notes/show.tsx`
- **Route:** `GET /finance/credit-notes/{id}`
- **Purpose:** View credit note details, apply to invoices, record refunds
- **Data displayed:** Credit note header, line items, applied invoices, refund history, remaining balance
- **Actions:** Open, Apply to Invoices (with invoice selection and amount allocation), Record Refund (with account selection), Remove Application, Delete
- **Access:** `credit-note.view` (actions require `credit-note.edit`)

---

## 6. Finance — Payment Receives

### Payment Receive List
- **File:** `resources/js/pages/finance/payment-receives/index.tsx`
- **Route:** `GET /finance/payment-receives`
- **Purpose:** Browse customer payments
- **Data displayed:** DataTable with payment number, customer, date, amount, deposit account
- **Access:** `payment-receive.view`

### Payment Receive Create
- **File:** `resources/js/pages/finance/payment-receives/create.tsx`
- **Route:** `GET /finance/payment-receives/create`
- **Purpose:** Record a customer payment and allocate to invoices
- **Data displayed:** Customer select, deposit account select, payment date, unpaid invoices table with allocation inputs
- **Actions:** Submit, cancel. Supports `?invoice_id=` query param for preselection.
- **Access:** `payment-receive.create`

---

## 7. Finance — Bills

### Bill List
- **File:** `resources/js/pages/finance/bills/index.tsx`
- **Route:** `GET /finance/bills`
- **Purpose:** Browse and manage vendor bills
- **Data displayed:** DataTable with bill number, vendor, date, due date, amount, status, balance
- **Actions:** Create, search, filter by status/vendor/date, row actions
- **Access:** `bill.view`

### Bill Create
- **File:** `resources/js/pages/finance/bills/create.tsx`
- **Route:** `GET /finance/bills/create`
- **Purpose:** Enter a new vendor bill
- **Data displayed:** Form with vendor, dates, bill number, line items
- **Access:** `bill.create`

### Bill Edit
- **File:** `resources/js/pages/finance/bills/edit.tsx`
- **Route:** `GET /finance/bills/{id}/edit`
- **Purpose:** Edit an existing bill
- **Access:** `bill.edit`

### Bill Show
- **File:** `resources/js/pages/finance/bills/show.tsx`
- **Route:** `GET /finance/bills/{id}`
- **Purpose:** View bill details and take actions
- **Actions:** Open, Duplicate, Edit, Delete, link to create bill payment
- **Access:** `bill.view`

### Bill Print
- **File:** `resources/js/pages/finance/bills/print.tsx`
- **Route:** `GET /finance/bills/{id}/print`
- **Layout:** PrintLayout
- **Access:** `bill.view`

---

## 8. Finance — Bill Payments

### Bill Payment List
- **File:** `resources/js/pages/finance/bill-payments/index.tsx`
- **Route:** `GET /finance/bill-payments`
- **Purpose:** Browse vendor payments
- **Data displayed:** DataTable with payment number, vendor, date, amount, payment account
- **Access:** `bill-payment.view`

### Bill Payment Create
- **File:** `resources/js/pages/finance/bill-payments/create.tsx`
- **Route:** `GET /finance/bill-payments/create`
- **Purpose:** Record a vendor payment and allocate to bills
- **Data displayed:** Vendor select, payment account, unpaid bills with allocation inputs
- **Actions:** Supports `?vendor_id=` and `?bill_id=` query params
- **Access:** `bill-payment.create`

---

## 9. Finance — Vendor Credits

### Vendor Credit List
- **File:** `resources/js/pages/finance/vendor-credits/index.tsx`
- **Route:** `GET /finance/vendor-credits`
- **Purpose:** Browse vendor credit notes
- **Access:** `vendor-credit.view`

### Vendor Credit Create
- **File:** `resources/js/pages/finance/vendor-credits/create.tsx`
- **Route:** `GET /finance/vendor-credits/create`
- **Purpose:** Record a new vendor credit
- **Access:** `vendor-credit.create`

### Vendor Credit Show
- **File:** `resources/js/pages/finance/vendor-credits/show.tsx`
- **Route:** `GET /finance/vendor-credits/{id}`
- **Purpose:** View vendor credit, apply to bills, record refunds
- **Actions:** Open, Apply to Bills, Record Refund, Remove Application, Delete
- **Access:** `vendor-credit.view`

---

## 10. Accounting — Chart of Accounts

### Account List
- **File:** `resources/js/pages/accounting/accounts/index.tsx`
- **Route:** `GET /accounting/accounts`
- **Purpose:** View the complete chart of accounts in hierarchical tree structure
- **Data displayed:** Account tree with code, name, type, balance, active status
- **Actions:** Create, edit, delete (with ConfirmDialog)
- **Access:** `account.view`

### Account Create
- **File:** `resources/js/pages/accounting/accounts/create.tsx`
- **Route:** `GET /accounting/accounts/create`
- **Purpose:** Add a new account to the chart
- **Data displayed:** Form with name, code, type select, parent account select, description, currency
- **Access:** `account.create`

### Account Edit
- **File:** `resources/js/pages/accounting/accounts/edit.tsx`
- **Route:** `GET /accounting/accounts/{id}/edit`
- **Purpose:** Edit account details (type is immutable after creation)
- **Data displayed:** Pre-filled form, flag if account has GL transactions
- **Access:** `account.edit`

---

## 11. Accounting — Journal Entries

### Journal List
- **File:** `resources/js/pages/accounting/journals/index.tsx`
- **Route:** `GET /accounting/journals`
- **Purpose:** Browse manual journal entries
- **Data displayed:** DataTable with journal number, date, amount, status, description
- **Actions:** Create, search, filter by status/date
- **Access:** `manual-journal.view`

### Journal Create
- **File:** `resources/js/pages/accounting/journals/create.tsx`
- **Route:** `GET /accounting/journals/create`
- **Purpose:** Create a new manual journal entry
- **Data displayed:** Journal header (number, date, reference, description), debit/credit entry lines with account select, amount inputs, running total showing debit = credit balance
- **Access:** `manual-journal.create`

### Journal Show
- **File:** `resources/js/pages/accounting/journals/show.tsx`
- **Route:** `GET /accounting/journals/{id}`
- **Purpose:** View journal entry details
- **Data displayed:** Header info, entry lines with accounts and amounts, debit/credit totals
- **Actions:** Publish, Edit, Delete
- **Access:** `manual-journal.view`

---

## 12. Accounting — Expenses

### Expense List
- **File:** `resources/js/pages/accounting/expenses/index.tsx`
- **Route:** `GET /accounting/expenses`
- **Purpose:** Browse expense records
- **Data displayed:** DataTable with date, payee, amount, payment account, status
- **Access:** `expense.view`

### Expense Create
- **File:** `resources/js/pages/accounting/expenses/create.tsx`
- **Route:** `GET /accounting/expenses/create`
- **Purpose:** Record a new expense
- **Data displayed:** Form with payment account, payee (vendor), date, reference, expense category lines (account, description, amount)
- **Access:** `expense.create`

### Expense Show
- **File:** `resources/js/pages/accounting/expenses/show.tsx`
- **Route:** `GET /accounting/expenses/{id}`
- **Purpose:** View expense details
- **Actions:** Publish, Edit, Delete
- **Access:** `expense.view`

---

## 13. Reports

### Reports Index
- **File:** `resources/js/pages/reports/index.tsx`
- **Route:** `GET /reports`
- **Purpose:** Report selection hub with cards for each available report
- **Access:** `report.view`

### Balance Sheet
- **File:** `resources/js/pages/reports/balance-sheet.tsx`
- **Route:** `GET /reports/balance-sheet`
- **Purpose:** Assets = Liabilities + Equity at a point in time
- **Data displayed:** Date picker, account groups with balances, totals
- **Access:** `report.view`

### Income Statement
- **File:** `resources/js/pages/reports/income-statement.tsx`
- **Route:** `GET /reports/income-statement`
- **Purpose:** Revenue - Expenses = Net Income for a period
- **Data displayed:** Date range picker, income accounts, expense accounts, totals, net income
- **Access:** `report.view`

### Trial Balance
- **File:** `resources/js/pages/reports/trial-balance.tsx`
- **Route:** `GET /reports/trial-balance`
- **Purpose:** Verify debits = credits across all accounts
- **Data displayed:** All accounts with debit and credit totals
- **Access:** `report.view`

### Cash Flow Statement
- **File:** `resources/js/pages/reports/cash-flow.tsx`
- **Route:** `GET /reports/cash-flow`
- **Purpose:** Cash inflows and outflows by activity type
- **Data displayed:** Operating, investing, financing sections with net totals
- **Access:** `report.view`

### Receivables Aging
- **File:** `resources/js/pages/reports/receivables-aging.tsx`
- **Route:** `GET /reports/receivables-aging`
- **Purpose:** Outstanding customer balances by age bucket
- **Data displayed:** As-of date, optional customer filter, aging grid (Current, 1-30, 31-60, 61-90, 90+)
- **Access:** `report.view`

### Payables Aging
- **File:** `resources/js/pages/reports/payables-aging.tsx`
- **Route:** `GET /reports/payables-aging`
- **Purpose:** Outstanding vendor balances by age bucket
- **Access:** `report.view`

### Tax Summary
- **File:** `resources/js/pages/reports/tax-summary.tsx`
- **Route:** `GET /reports/tax-summary`
- **Purpose:** Tax collected vs. tax paid for a period
- **Data displayed:** Date range, tax collected, tax paid, net tax payable
- **Access:** `report.view`

---

## 14. Settings

### Organization Settings
- **File:** `resources/js/pages/settings/organization.tsx`
- **Route:** `GET /settings/organization`
- **Purpose:** Configure organization-wide settings
- **Data displayed:** Form with organization name, base currency, fiscal year start, accounting basis, date format, timezone
- **Actions:** Save settings
- **Access:** `setting.edit` (admin)

### Currencies
- **File:** `resources/js/pages/settings/currencies.tsx`
- **Route:** `GET /settings/currencies`
- **Purpose:** Manage currencies and exchange rates
- **Data displayed:** Currency list with code, name, symbol; exchange rate form
- **Actions:** Create, edit (name/symbol), delete, set exchange rate
- **Access:** Admin role

### Tax Rates
- **File:** `resources/js/pages/settings/tax-rates.tsx`
- **Route:** `GET /settings/tax-rates`
- **Purpose:** Manage tax rate definitions
- **Data displayed:** Tax rate list with name, code, rate, active status
- **Actions:** Create, edit, delete, toggle active
- **Access:** Admin role

### Items
- **File:** `resources/js/pages/settings/items.tsx`
- **Route:** `GET /settings/items`
- **Purpose:** Manage product and service catalog
- **Data displayed:** Item list with name, code, type, sell price, cost price, active status
- **Actions:** Create, edit, delete
- **Access:** `item.view` / `item.create` / `item.edit`

### Contacts
- **File:** `resources/js/pages/settings/contacts.tsx`
- **Route:** `GET /settings/contacts`
- **Purpose:** Manage customers and vendors
- **Data displayed:** Contact list with type filter (customer/vendor), display name, company, email, phone, balance
- **Actions:** Create, edit, delete, filter by type
- **Access:** `contact.view` / `contact.create` / `contact.edit`

### Users
- **File:** `resources/js/pages/settings/users.tsx`
- **Route:** `GET /settings/users`
- **Purpose:** Manage user accounts and role assignments
- **Data displayed:** User list with name, email, role(s)
- **Actions:** Create user (name, email, password with visibility toggle, role select), edit, delete
- **Access:** Admin role only (checked via `hasRole('admin')`)

---

## Page Count Summary

| Module | Pages |
|--------|-------|
| Authentication | 1 |
| Dashboard | 1 |
| Finance — Invoices | 5 |
| Finance — Estimates | 4 |
| Finance — Credit Notes | 3 |
| Finance — Payment Receives | 2 |
| Finance — Bills | 5 |
| Finance — Bill Payments | 2 |
| Finance — Vendor Credits | 3 |
| Accounting — Chart of Accounts | 3 |
| Accounting — Journal Entries | 3 |
| Accounting — Expenses | 3 |
| Reports | 8 |
| Settings | 6 |
| **Total** | **49** |
