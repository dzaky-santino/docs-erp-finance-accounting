# 01 — System Overview

## Project Identity

**System Name:** ERP Finance & Accounting

**Purpose:** A complete finance and accounting module for small-to-medium enterprises, providing double-entry bookkeeping, accounts receivable, accounts payable, expense tracking, multi-currency support, and financial reporting.

**Target Users:** Finance teams, accountants, and business owners who need to manage invoices, bills, payments, journal entries, and generate financial statements (Balance Sheet, Income Statement, Trial Balance, Cash Flow, Aging Reports, Tax Summary).

**Business Context:** Designed for Indonesian businesses following PSAK (Pernyataan Standar Akuntansi Keuangan) conventions. The default chart of accounts includes Indonesian tax accounts (PPN, PPh 21/23/4(2)), BPJS payroll liabilities, and IDR as the base currency. The system is equally usable for businesses in other jurisdictions by reconfiguring the chart of accounts and tax rates.

**Deployment Model:** The system is designed to be independently deployable as a standalone web application, or embedded into a larger ERP platform. It has no external service dependencies beyond a MySQL database and a web server.

---

## Technology Stack

### Backend

| Technology | Version | Purpose |
|-----------|---------|---------|
| PHP | ^8.3 | Server-side language |
| Laravel | ^13.0 | Web application framework |
| Laravel Sanctum | ^4.3 | Session-based and token-based authentication |
| Spatie Laravel Permission | ^7.2 | Role-based access control (RBAC) |
| Inertia.js (server) | ^2.0 | Server-side adapter bridging Laravel to React |
| PHPUnit | ^12.5 | Testing framework |

### Frontend

| Technology | Version | Purpose |
|-----------|---------|---------|
| React | ^19.0 | UI component library |
| TypeScript | ^5.7 | Type-safe JavaScript |
| Inertia.js (client) | ^2.0 | Client-side adapter for SPA navigation |
| Tailwind CSS | ^4.0 | Utility-first CSS framework |
| Vite | ^8.0 | Frontend build tool and dev server |
| Lucide React | ^0.460 | Icon library |
| Recharts | ^2.15 | Chart library for dashboard visualizations |
| Zod | ^3.24 | Schema validation |
| React Hook Form | ^7.54 | Form state management |
| TanStack React Query | ^5.62 | Server state management |

### Database

| Technology | Version | Purpose |
|-----------|---------|---------|
| MySQL | 8.0+ | Primary database engine |
| PostgreSQL | (alternative) | Supported via Laravel's database abstraction |

### Key Conventions

- All monetary columns: `decimal(15,5)`
- All exchange rate columns: `decimal(13,9)`
- All tax rate columns: `decimal(8,4)`
- Session driver: `database`
- Cache driver: `database`
- Queue driver: `database`

---

## System Architecture

### Modular Monolith

The application is a modular monolith — a single deployable unit with clearly separated internal modules:

- **Core** — Users, authentication, settings, currencies
- **Master Data** — Accounts (chart of accounts), contacts (customers/vendors), items, tax rates
- **Finance A/R** — Sale invoices, sale estimates, payment receives, credit notes
- **Finance A/P** — Bills, bill payments, vendor credits
- **Accounting** — Manual journal entries, expenses, general ledger (account transactions)
- **Reports** — Balance Sheet, Income Statement, Trial Balance, Cash Flow, Aging, Tax Summary

Each module has its own controllers, form requests, services, and frontend pages, but they share the same database and Eloquent models.

### Layered Architecture

Every request flows through four layers:

```
HTTP Request
    │
    ▼
Form Request (validation + authorization)
    │
    ▼
Controller (HTTP concerns, response formatting)
    │
    ▼
Service (business logic, GL entries, events)
    │
    ▼
Model (data access, relationships, scopes)
```

- **Form Requests** validate input and check permissions using `$this->user()->can('module.action')`.
- **Controllers** delegate all business logic to services. They handle Inertia vs. API response branching.
- **Services** contain all business rules: status transitions, balance calculations, GL entry creation, domain exception throwing.
- **Models** define table mappings, relationships, scopes, and casts. All 46 models use the `Auditable` trait.

### Inertia.js Bridge

Inertia.js eliminates the need for a separate REST API for the frontend. The flow is:

1. Laravel routes return `Inertia::render('PageName', $props)` instead of JSON or Blade views.
2. The Inertia middleware serializes the response as JSON with page component name and props.
3. On the client, Inertia's React adapter renders the correct page component with the props.
4. Form submissions use `useForm().post()` or `router.post()`, which sends requests with the `X-Inertia` header.
5. The server detects `X-Inertia` and returns a redirect with flash data instead of JSON.

This gives the user experience of an SPA (no full page reloads) while keeping all routing and data loading on the server.

### Authentication

- **Web sessions:** Laravel Sanctum with the `database` session driver. Login creates a session cookie; logout destroys it.
- **API tokens:** Sanctum personal access tokens for external API consumers. Issued via `POST /api/login`.
- **Self-registration is disabled.** User accounts are created only by administrators through Settings > Users.

### Role-Based Access Control

Three roles implemented via Spatie Laravel Permission:

| Role | Description |
|------|-------------|
| **admin** | Full access to all 79 permissions across all modules |
| **staff** | Create/edit on finance modules, view-only on settings and master data, no delete |
| **viewer** | Read-only access across the entire system |

Permission format: `{module}.{action}` (e.g., `sale-invoice.create`, `bill.delete`, `report.view`).

### Double-Entry Accounting Engine

The `account_transactions` table is the General Ledger (GL) core. Every financial event generates balanced debit/credit entries:

| Event | Debit Account | Credit Account |
|-------|--------------|----------------|
| Invoice delivered | Accounts Receivable | Sales Revenue + Tax Payable |
| Payment received | Cash/Bank | Accounts Receivable |
| Bill opened | Expense/COGS + Tax Payable | Accounts Payable |
| Bill payment | Accounts Payable | Cash/Bank |
| Credit note opened | Sales Revenue | Accounts Receivable |
| Vendor credit opened | Accounts Payable | Expense/COGS |
| Expense published | Expense accounts | Cash/Bank |
| Manual journal | As specified | As specified (must balance) |

GL entries are created by service classes when documents transition to their "active" state (delivered, opened, published). They are reverted when documents are deleted or updated.

---

## Modules Overview

### Core Module
User authentication, session management, and organization settings. Provides the login page, user management (admin-only), and global configuration (base currency, fiscal year, date format, timezone, accounting basis). The `HandleInertiaRequests` middleware shares auth, flash, and settings data on every page.

### Master Data Module
Chart of Accounts with hierarchical parent-child structure (max 5 levels), 19 account types, and predefined system accounts. Contacts serve dual purpose as both customers and vendors. Items support three types (service, inventory, non-inventory) with linked sell/cost/inventory accounts. Tax rates include Indonesian PPN and PPh rates.

### Finance — Accounts Receivable Module
Sale estimates (quotations) with delivery, approval, rejection, and conversion to invoices. Sale invoices with full lifecycle (Draft > Delivered > Unpaid > Partially Paid > Paid), write-off capability, and duplication. Payment receives for recording customer payments distributed across multiple invoices. Credit notes for customer credits with application to invoices or cash refund.

### Finance — Accounts Payable Module
Purchase bills with full lifecycle (Draft > Opened > Unpaid > Overdue > Partially Paid > Paid), duplication, and landed cost tracking. Bill payments for recording vendor payments distributed across multiple bills. Vendor credits for vendor refunds with application to bills or cash refund.

### Accounting Module
Manual journal entries with balanced debit/credit validation and draft/published workflow. Direct expense recording with multi-category allocation and draft/published workflow. The account_transactions table serves as the unified general ledger for all financial documents.

### Reports Module
Seven financial reports generated from GL data: Balance Sheet (point-in-time), Income Statement (date range), Trial Balance (date range), Cash Flow Statement (date range), Receivables Aging (bucket analysis), Payables Aging (bucket analysis), and Tax Summary (collected vs. paid).

### Settings Module
Organization configuration, currency management with exchange rate history, tax rate management, item/product catalog, contact directory, and user administration.

---

## Key Design Decisions

### Soft Deletes on All Financial Records
Every financial model uses Laravel's `SoftDeletes` trait. Deleted records are marked with a `deleted_at` timestamp but remain in the database. This preserves audit trails, prevents broken references in GL entries, and allows data recovery. Unique constraints are scoped to exclude soft-deleted records using `whereNull('deleted_at')`.

### Audit Trail on Every Table
All 46 models include `created_by`, `updated_by`, and `deleted_by` columns populated automatically by the `Auditable` trait via model boot events. This provides a complete record of who created, modified, and deleted every record in the system.

### URL Masking
The browser address bar always displays `/` regardless of the actual page. This is implemented by monkey-patching `window.history.pushState` and `window.history.replaceState` in `app.tsx` to always write `/` as the URL. Components that need the actual route path use `usePage().url` from Inertia's internal state. This design decision has implications: `redirect()->back()` cannot be used in controllers (the Referer header is always `/`), so all controllers use explicit `redirect()->route()` targets.

### Inertia Instead of Separate API
The system uses Inertia.js to serve the React frontend directly from Laravel, avoiding the complexity of maintaining a separate API for the SPA. Controllers detect the `X-Inertia` header to branch between redirect responses (for the SPA) and JSON responses (for external API consumers). This means every controller method serves both the web interface and the API.

### Multi-Currency Support
Every financial document stores a `currency_code` and `exchange_rate`. GL entries are recorded in both the document currency and the base currency (via multiplication by the exchange rate). The `exchange_rates` table stores daily rate history. Cash, Bank, and Credit Card account types support multi-currency balances.

### Indonesian Business Context
The default configuration targets Indonesian businesses: IDR as base currency, PSAK-compliant chart of accounts, Indonesian tax rates (PPN 11%/12%, PPh 21/23/4(2)), BPJS payroll accounts, Jakarta timezone, and `dd/MM/yyyy` date format. All of these are configurable through settings and seeders.
