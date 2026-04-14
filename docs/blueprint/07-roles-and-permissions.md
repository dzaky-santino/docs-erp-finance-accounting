# 07 — Roles and Permissions

All data is derived from reading `database/seeders/RolePermissionSeeder.php`, all 9 policy files in `app/Policies/`, and all 28 form request `authorize()` methods in `app/Http/Requests/`.

---

## RBAC Implementation

The system uses **Spatie Laravel Permission** for role-based access control.

- **Guard:** `web`
- **Permission format:** `{module}.{action}`
- **Roles:** 3 (admin, staff, viewer)
- **Modules:** 20
- **Total permissions:** 79

---

## Modules and Actions

| Module | create | edit | delete | view | writeoff |
|--------|:------:|:----:|:------:|:----:|:--------:|
| account | x | x | x | x | |
| sale-invoice | x | x | x | x | x |
| sale-estimate | x | x | x | x | |
| sale-receipt | x | x | x | x | |
| payment-receive | x | x | x | x | |
| bill | x | x | x | x | |
| bill-payment | x | x | x | x | |
| credit-note | x | x | x | x | |
| vendor-credit | x | x | x | x | |
| expense | x | x | x | x | |
| manual-journal | x | x | x | x | |
| item | x | x | x | x | |
| contact | x | x | x | x | |
| tax-rate | x | x | x | x | |
| cashflow | x | x | x | x | |
| inventory-adjustment | x | x | x | x | |
| warehouse-transfer | x | x | x | x | |
| project | x | x | x | x | |
| report | | | | x | |
| setting | | x | | x | |

---

## Role: admin

**Description:** Full access to everything.

Has **all 79 permissions** across all 20 modules. Can create, edit, delete, and view every resource. Can write off invoices. Can edit system settings and manage users.

User management (Settings > Users) is additionally gated by `$user->hasRole('admin')` in form request authorization, since no `user` module is defined in the permission set.

---

## Role: staff

**Description:** Create, edit, and view on Finance & Accounting modules. View-only on everything else. Cannot delete any record.

### Full access (create + edit + view):

| Module | Permissions |
|--------|------------|
| sale-invoice | create, edit, view |
| sale-estimate | create, edit, view |
| sale-receipt | create, edit, view |
| payment-receive | create, edit, view |
| bill | create, edit, view |
| bill-payment | create, edit, view |
| credit-note | create, edit, view |
| vendor-credit | create, edit, view |
| expense | create, edit, view |
| manual-journal | create, edit, view |
| cashflow | create, edit, view |

### View-only:

| Module | Permissions |
|--------|------------|
| account | view |
| item | view |
| contact | view |
| tax-rate | view |
| inventory-adjustment | view |
| warehouse-transfer | view |
| project | view |
| report | view |
| setting | view |

### Denied:

- All `*.delete` permissions
- `sale-invoice.writeoff`
- `setting.edit`
- User management (requires admin role)

**Total staff permissions:** 42

---

## Role: viewer

**Description:** Read-only access across the entire system.

Has only `*.view` on every module that has a view action.

| Module | Permission |
|--------|-----------|
| account | view |
| sale-invoice | view |
| sale-estimate | view |
| sale-receipt | view |
| payment-receive | view |
| bill | view |
| bill-payment | view |
| credit-note | view |
| vendor-credit | view |
| expense | view |
| manual-journal | view |
| item | view |
| contact | view |
| tax-rate | view |
| cashflow | view |
| inventory-adjustment | view |
| warehouse-transfer | view |
| project | view |
| report | view |
| setting | view |

**Total viewer permissions:** 20

---

## Policy Classes

All 9 policies follow an identical pattern. Each method checks a single permission via `$user->can('module.action')`.

| Policy | Model | Module Prefix |
|--------|-------|--------------|
| AccountPolicy | Account | `account` |
| SaleInvoicePolicy | SaleInvoice | `sale-invoice` |
| SaleEstimatePolicy | SaleEstimate | `sale-estimate` |
| CreditNotePolicy | CreditNote | `credit-note` |
| VendorCreditPolicy | VendorCredit | `vendor-credit` |
| BillPolicy | Bill | `bill` |
| ManualJournalPolicy | ManualJournal | `manual-journal` |
| ExpensePolicy | Expense | `expense` |
| ContactPolicy | Contact | `contact` |

### Standard Policy Methods

Every policy implements these 5 methods:

| Method | Permission Checked |
|--------|--------------------|
| `viewAny(User $user)` | `{module}.view` |
| `view(User $user, Model $model)` | `{module}.view` |
| `create(User $user)` | `{module}.create` |
| `update(User $user, Model $model)` | `{module}.edit` |
| `delete(User $user, Model $model)` | `{module}.delete` |

Policies are registered in `AppServiceProvider`.

---

## Form Request Authorization

Form requests check permissions in their `authorize()` method. The permission names match the seeded `module.action` format exactly.

### Settings Module

| Form Request | Permission |
|-------------|-----------|
| StoreAccountRequest | `account.create` |
| UpdateAccountRequest | `account.edit` |
| StoreContactRequest | `contact.create` |
| UpdateContactRequest | `contact.edit` |
| StoreCurrencyRequest | `account.create` |
| UpdateCurrencyRequest | `account.edit` |
| SetExchangeRateRequest | `account.edit` |
| StoreTaxRateRequest | `tax-rate.create` |
| UpdateTaxRateRequest | `tax-rate.edit` |
| StoreItemRequest | `item.create` |
| UpdateItemRequest | `item.edit` |
| UpdateSettingRequest | `setting.edit` |
| StoreUserRequest | `hasRole('admin')` |
| UpdateUserRequest | `hasRole('admin')` |

### Finance Module

| Form Request | Permission |
|-------------|-----------|
| StoreInvoiceRequest | `sale-invoice.create` |
| UpdateInvoiceRequest | `sale-invoice.edit` |
| StoreEstimateRequest | `sale-estimate.create` |
| UpdateEstimateRequest | `sale-estimate.edit` |
| StoreBillRequest | `bill.create` |
| UpdateBillRequest | `bill.edit` |
| StorePaymentReceiveRequest | `payment-receive.create` |
| StoreBillPaymentRequest | `bill-payment.create` |
| StoreCreditNoteRequest | `credit-note.create` |
| ApplyCreditNoteRequest | `credit-note.edit` |
| RefundCreditNoteRequest | `credit-note.edit` |
| StoreVendorCreditRequest | `vendor-credit.create` |
| ApplyVendorCreditRequest | `vendor-credit.edit` |
| RefundVendorCreditRequest | `vendor-credit.edit` |

### Accounting Module

| Form Request | Permission |
|-------------|-----------|
| StoreJournalRequest | `manual-journal.create` |
| UpdateJournalRequest | `manual-journal.edit` |
| StoreExpenseRequest | `expense.create` |
| UpdateExpenseRequest | `expense.edit` |

---

## Controller-Level Authorization

Controllers that do not use policies check permissions directly in their methods:

- **Status change actions** (deliver, open, approve, publish, write-off): Checked via form request or inline `$this->authorize()` using the `edit` permission of the corresponding module
- **Delete actions**: Checked via policy `delete` method or inline permission check
- **Report actions**: `ReportController` checks `report.view`

---

## Demo User Accounts

Created by `UserAccountSeeder.php` (runs after `RolePermissionSeeder`):

| Name | Email | Password | Role |
|------|-------|----------|------|
| Administrator | admin@erp.test | password | admin |
| Staff User | staff@erp.test | password | staff |
| Viewer User | viewer@erp.test | password | viewer |

All accounts use `firstOrCreate` for idempotent re-runs. Passwords are hashed via Laravel's `User` model mutator.
