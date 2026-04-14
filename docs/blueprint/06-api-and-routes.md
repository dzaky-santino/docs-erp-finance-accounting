# 06 — API and Routes

All routes are derived from reading `routes/web.php` and `routes/api.php`.

---

## Route Architecture

The system has two route files serving different purposes:

- **`routes/web.php`** — Serves the Inertia SPA. GET routes render pages via `Inertia::render()`. POST/PUT/DELETE routes handle form submissions from the frontend and are also mirrored from the API controllers.
- **`routes/api.php`** — Serves external API consumers using Bearer token authentication. Returns JSON responses.

All authenticated web routes use the `auth:sanctum` middleware (session-based). All API routes use `auth:sanctum` middleware (token-based).

### URL Masking
The browser address bar always shows `/` regardless of the current page. This is implemented in `resources/js/app.tsx` by monkey-patching `window.history.pushState/replaceState`. Components use `usePage().url` for the actual path. Controllers use explicit `redirect()->route()` instead of `redirect()->back()` because the Referer header is always `/`.

### Inertia vs API Response Pattern
Every controller method checks for the `X-Inertia` header:
- **Inertia request:** Returns `redirect()->route('target')->with('success', 'message')`
- **API request:** Returns JSON response with appropriate status code

---

## Authentication Routes

### Web Routes (no middleware)

| Method | URI | Handler | Purpose |
|--------|-----|---------|---------|
| GET | `/auth/login` | Closure → Inertia::render('auth/login') | Login page |
| POST | `/auth/login` | AuthController@login | Process login |
| GET | `/auth/register` | Redirect to `/auth/login` | Legacy redirect |

### Web Routes (auth:sanctum)

| Method | URI | Handler | Purpose |
|--------|-----|---------|---------|
| POST | `/auth/logout` | AuthController@logout | Destroy session |

### API Routes

| Method | URI | Middleware | Handler | Purpose |
|--------|-----|-----------|---------|---------|
| POST | `/api/login` | — | AuthController@apiLogin | Get API token |
| POST | `/api/logout` | auth:sanctum | AuthController@logout | Revoke token |
| GET | `/api/me` | auth:sanctum | AuthController@me | Current user info |

---

## Dashboard Routes

| Method | URI | Handler | Purpose |
|--------|-----|---------|---------|
| GET | `/dashboard` | Closure → Inertia::render | Dashboard with receivables, payables, cash balance, charts |

**Data passed:** receivables total, payables total, cash balance, monthly income/expenses (last 12 months), recent invoices (5), recent bills (5).

---

## Finance — Invoices

### Web Page Routes

| Method | URI | Handler | Purpose |
|--------|-----|---------|---------|
| GET | `/finance/invoices` | Closure → Inertia::render | Invoice list with filters |
| GET | `/finance/invoices/create` | Closure → Inertia::render | Create form with customers, items, accounts, taxes |
| GET | `/finance/invoices/{id}/edit` | Closure → Inertia::render | Edit form |
| GET | `/finance/invoices/{id}` | Closure → Inertia::render | Show detail |
| GET | `/finance/invoices/{id}/print` | Closure → Inertia::render | Print view |

### Web Form Routes (auth:sanctum)

| Method | URI | Handler | Purpose |
|--------|-----|---------|---------|
| POST | `/api/invoices` | SaleInvoiceController@store | Create invoice |
| PUT | `/api/invoices/{id}` | SaleInvoiceController@update | Update invoice |
| DELETE | `/api/invoices/{id}` | SaleInvoiceController@destroy | Delete invoice |
| POST | `/api/invoices/{id}/deliver` | SaleInvoiceController@deliver | Mark as delivered |
| POST | `/api/invoices/{id}/write-off` | SaleInvoiceController@writeOff | Write off balance |
| POST | `/api/invoices/{id}/cancel-write-off` | SaleInvoiceController@cancelWriteOff | Cancel write-off |
| POST | `/api/invoices/{id}/duplicate` | SaleInvoiceController@duplicate | Duplicate as draft |

### API Routes (auth:sanctum)

| Method | URI | Handler |
|--------|-----|---------|
| GET | `/api/invoices` | SaleInvoiceController@index |
| POST | `/api/invoices` | SaleInvoiceController@store |
| GET | `/api/invoices/{id}` | SaleInvoiceController@show |
| PUT | `/api/invoices/{id}` | SaleInvoiceController@update |
| DELETE | `/api/invoices/{id}` | SaleInvoiceController@destroy |
| POST | `/api/invoices/{id}/deliver` | SaleInvoiceController@deliver |
| POST | `/api/invoices/{id}/write-off` | SaleInvoiceController@writeOff |
| POST | `/api/invoices/{id}/cancel-write-off` | SaleInvoiceController@cancelWriteOff |
| POST | `/api/invoices/{id}/duplicate` | SaleInvoiceController@duplicate |

---

## Finance — Estimates

### Web Page Routes

| Method | URI | Purpose |
|--------|-----|---------|
| GET | `/finance/estimates` | Estimate list |
| GET | `/finance/estimates/create` | Create form |
| GET | `/finance/estimates/{id}/edit` | Edit form |
| GET | `/finance/estimates/{id}` | Show detail |

### API Routes (auth:sanctum)

| Method | URI | Handler |
|--------|-----|---------|
| GET | `/api/estimates` | SaleEstimateController@index |
| POST | `/api/estimates` | SaleEstimateController@store |
| GET | `/api/estimates/{id}` | SaleEstimateController@show |
| PUT | `/api/estimates/{id}` | SaleEstimateController@update |
| DELETE | `/api/estimates/{id}` | SaleEstimateController@destroy |
| POST | `/api/estimates/{id}/deliver` | SaleEstimateController@deliver |
| POST | `/api/estimates/{id}/approve` | SaleEstimateController@approve |
| POST | `/api/estimates/{id}/reject` | SaleEstimateController@reject |
| POST | `/api/estimates/{id}/convert` | SaleEstimateController@convertToInvoice |

---

## Finance — Credit Notes

### Web Page Routes

| Method | URI | Purpose |
|--------|-----|---------|
| GET | `/finance/credit-notes` | List |
| GET | `/finance/credit-notes/create` | Create form |
| GET | `/finance/credit-notes/{id}` | Show with unpaid invoices for application |

### API Routes (auth:sanctum)

| Method | URI | Handler |
|--------|-----|---------|
| GET | `/api/credit-notes` | CreditNoteController@index |
| POST | `/api/credit-notes` | CreditNoteController@store |
| GET | `/api/credit-notes/{id}` | CreditNoteController@show |
| DELETE | `/api/credit-notes/{id}` | CreditNoteController@destroy |
| POST | `/api/credit-notes/{id}/open` | CreditNoteController@open |
| POST | `/api/credit-notes/{id}/apply` | CreditNoteController@applyToInvoices |
| DELETE | `/api/credit-notes/applications/{id}` | CreditNoteController@deleteApplication |
| POST | `/api/credit-notes/{id}/refund` | CreditNoteController@refund |

---

## Finance — Payment Receives

### Web Page Routes

| Method | URI | Purpose |
|--------|-----|---------|
| GET | `/finance/payment-receives` | List |
| GET | `/finance/payment-receives/create` | Create form (supports `?invoice_id=` param) |

### API Routes (auth:sanctum)

| Method | URI | Handler |
|--------|-----|---------|
| GET | `/api/payment-receives` | PaymentReceiveController@index |
| POST | `/api/payment-receives` | PaymentReceiveController@store |
| GET | `/api/payment-receives/{id}` | PaymentReceiveController@show |
| DELETE | `/api/payment-receives/{id}` | PaymentReceiveController@destroy |

---

## Finance — Bills

### Web Page Routes

| Method | URI | Purpose |
|--------|-----|---------|
| GET | `/finance/bills` | Bill list |
| GET | `/finance/bills/create` | Create form |
| GET | `/finance/bills/{id}/edit` | Edit form |
| GET | `/finance/bills/{id}` | Show detail |
| GET | `/finance/bills/{id}/print` | Print view |

### API Routes (auth:sanctum)

| Method | URI | Handler |
|--------|-----|---------|
| GET | `/api/bills` | BillController@index |
| POST | `/api/bills` | BillController@store |
| GET | `/api/bills/{id}` | BillController@show |
| PUT | `/api/bills/{id}` | BillController@update |
| DELETE | `/api/bills/{id}` | BillController@destroy |
| POST | `/api/bills/{id}/open` | BillController@open |
| POST | `/api/bills/{id}/duplicate` | BillController@duplicate |

---

## Finance — Bill Payments

### Web Page Routes

| Method | URI | Purpose |
|--------|-----|---------|
| GET | `/finance/bill-payments` | List |
| GET | `/finance/bill-payments/create` | Create form (supports `?vendor_id=`, `?bill_id=`) |

### API Routes (auth:sanctum)

| Method | URI | Handler |
|--------|-----|---------|
| GET | `/api/bill-payments` | BillPaymentController@index |
| POST | `/api/bill-payments` | BillPaymentController@store |
| GET | `/api/bill-payments/{id}` | BillPaymentController@show |
| DELETE | `/api/bill-payments/{id}` | BillPaymentController@destroy |

---

## Finance — Vendor Credits

### Web Page Routes

| Method | URI | Purpose |
|--------|-----|---------|
| GET | `/finance/vendor-credits` | List |
| GET | `/finance/vendor-credits/create` | Create form |
| GET | `/finance/vendor-credits/{id}` | Show with unpaid bills |

### API Routes (auth:sanctum)

| Method | URI | Handler |
|--------|-----|---------|
| GET | `/api/vendor-credits` | VendorCreditController@index |
| POST | `/api/vendor-credits` | VendorCreditController@store |
| GET | `/api/vendor-credits/{id}` | VendorCreditController@show |
| DELETE | `/api/vendor-credits/{id}` | VendorCreditController@destroy |
| POST | `/api/vendor-credits/{id}/open` | VendorCreditController@open |
| POST | `/api/vendor-credits/{id}/apply` | VendorCreditController@applyToBills |
| DELETE | `/api/vendor-credits/applications/{id}` | VendorCreditController@deleteApplication |
| POST | `/api/vendor-credits/{id}/refund` | VendorCreditController@refund |

---

## Accounting — Chart of Accounts

### Web Page Routes

| Method | URI | Purpose |
|--------|-----|---------|
| GET | `/accounting/accounts` | Account tree |
| GET | `/accounting/accounts/create` | Create form |
| GET | `/accounting/accounts/{id}/edit` | Edit form |

### API Routes (auth:sanctum)

| Method | URI | Handler |
|--------|-----|---------|
| GET | `/api/accounts` | AccountController@index |
| POST | `/api/accounts` | AccountController@store |
| GET | `/api/accounts/{id}` | AccountController@show |
| PUT | `/api/accounts/{id}` | AccountController@update |
| DELETE | `/api/accounts/{id}` | AccountController@destroy |

---

## Accounting — Journals

### Web Page Routes

| Method | URI | Purpose |
|--------|-----|---------|
| GET | `/accounting/journals` | Journal list |
| GET | `/accounting/journals/create` | Create form |
| GET | `/accounting/journals/{id}` | Show detail |

### API Routes (auth:sanctum)

| Method | URI | Handler |
|--------|-----|---------|
| GET | `/api/journals` | ManualJournalController@index |
| POST | `/api/journals` | ManualJournalController@store |
| GET | `/api/journals/{id}` | ManualJournalController@show |
| PUT | `/api/journals/{id}` | ManualJournalController@update |
| DELETE | `/api/journals/{id}` | ManualJournalController@destroy |
| POST | `/api/journals/{id}/publish` | ManualJournalController@publish |

---

## Accounting — Expenses

### Web Page Routes

| Method | URI | Purpose |
|--------|-----|---------|
| GET | `/accounting/expenses` | Expense list |
| GET | `/accounting/expenses/create` | Create form |
| GET | `/accounting/expenses/{id}` | Show detail |

### API Routes (auth:sanctum)

| Method | URI | Handler |
|--------|-----|---------|
| GET | `/api/expenses` | ExpenseController@index |
| POST | `/api/expenses` | ExpenseController@store |
| GET | `/api/expenses/{id}` | ExpenseController@show |
| PUT | `/api/expenses/{id}` | ExpenseController@update |
| DELETE | `/api/expenses/{id}` | ExpenseController@destroy |
| POST | `/api/expenses/{id}/publish` | ExpenseController@publish |

---

## Reports

### Web Page Routes

| Method | URI | Purpose |
|--------|-----|---------|
| GET | `/reports` | Reports index |
| GET | `/reports/balance-sheet` | Balance sheet page |
| GET | `/reports/income-statement` | Income statement page |
| GET | `/reports/trial-balance` | Trial balance page |
| GET | `/reports/cash-flow` | Cash flow statement page |
| GET | `/reports/receivables-aging` | Receivables aging page |
| GET | `/reports/payables-aging` | Payables aging page |
| GET | `/reports/tax-summary` | Tax summary page |

### API Routes (auth:sanctum)

| Method | URI | Handler |
|--------|-----|---------|
| GET | `/api/reports/balance-sheet` | ReportController@balanceSheet |
| GET | `/api/reports/income-statement` | ReportController@incomeStatement |
| GET | `/api/reports/trial-balance` | ReportController@trialBalance |
| GET | `/api/reports/cash-flow` | ReportController@cashFlowStatement |
| GET | `/api/reports/receivables-aging` | ReportController@receivablesAging |
| GET | `/api/reports/payables-aging` | ReportController@payablesAging |
| GET | `/api/reports/tax-summary` | ReportController@taxSummary |

---

## Settings

### Web Page Routes

| Method | URI | Purpose |
|--------|-----|---------|
| GET | `/settings/organization` | Organization settings page |
| GET | `/settings/currencies` | Currencies page |
| GET | `/settings/tax-rates` | Tax rates page |
| GET | `/settings/items` | Items page |
| GET | `/settings/contacts` | Contacts page |
| GET | `/settings/users` | Users page |

### API Routes (auth:sanctum)

| Method | URI | Handler |
|--------|-----|---------|
| GET/POST | `/api/contacts` | ContactController@index/store |
| GET/PUT/DELETE | `/api/contacts/{id}` | ContactController@show/update/destroy |
| GET/POST | `/api/currencies` | CurrencyController@index/store |
| GET/PUT/DELETE | `/api/currencies/{id}` | CurrencyController@show/update/destroy |
| POST | `/api/currencies/exchange-rate` | CurrencyController@setExchangeRate |
| GET/POST | `/api/tax-rates` | TaxRateController@index/store |
| GET/PUT/DELETE | `/api/tax-rates/{id}` | TaxRateController@show/update/destroy |
| POST | `/api/tax-rates/{id}/toggle-active` | TaxRateController@toggleActive |
| GET/POST | `/api/items` | ItemController@index/store |
| GET/PUT/DELETE | `/api/items/{id}` | ItemController@show/update/destroy |
| GET/PUT | `/api/settings` | SettingController@index/update |
| GET | `/api/settings/{key}` | SettingController@show |
| GET/POST | `/api/users` | UserController@index/store |
| GET/PUT/DELETE | `/api/users/{id}` | UserController@show/update/destroy |

---

## Root Route

| Method | URI | Purpose |
|--------|-----|---------|
| GET | `/` | Redirects to `/dashboard` if authenticated, `/auth/login` if not |

---

## Shared Inertia Data (HandleInertiaRequests)

Every Inertia response includes these shared props:

```typescript
{
  auth: {
    user: { id, name, email, roles: Role[] } | null
  },
  flash: {
    success: string | null,
    error: string | null
  },
  settings: {
    base_currency: string,        // default: 'USD' (seeded as 'IDR')
    fiscal_year_start_month: string,  // default: 'january'
    accounting_basis: string,     // default: 'accrual'
    organization_name: string,    // default: ''
    date_format: string,          // default: 'dd/MM/yyyy'
    timezone: string              // default: 'UTC'
  }
}
```
