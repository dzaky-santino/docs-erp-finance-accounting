# 08 — Integration Guide

All information is derived from reading the actual source files: `composer.json`, `package.json`, configuration files, middleware, and the Inertia entry point.

---

## Developer Setup

### Prerequisites

- PHP 8.4+
- Composer 2.x
- Node.js 20+ with npm
- MySQL 8.0+ or PostgreSQL 15+

### Installation Steps

```bash
# 1. Clone the repository
git clone <repository-url>
cd erp-finance-accounting

# 2. Install PHP dependencies
composer install

# 3. Install frontend dependencies
npm install

# 4. Configure environment
cp .env.example .env
# Edit .env with database credentials

# 5. Generate application key
php artisan key:generate

# 6. Run database migrations
php artisan migrate

# 7. Seed default data
php artisan db:seed

# 8. Start development servers (two terminals)
npm run dev          # Terminal 1: Vite dev server
php artisan serve    # Terminal 2: Laravel dev server

# 9. Verify installation
php artisan test     # All 81 tests should pass
```

### Seeding Order

The `DatabaseSeeder` runs seeders in dependency order:

| Order | Seeder | Purpose |
|-------|--------|---------|
| 1 | CurrencySeeder | 43 world currencies + exchange rates vs IDR |
| 2 | BranchSeeder | 1 default branch (Head Office) |
| 3 | WarehouseSeeder | 1 default warehouse (Main Warehouse) |
| 4 | AccountSeeder | 95 PSAK-compliant chart of accounts |
| 5 | TaxRateSeeder | 12 Indonesian tax rates |
| 6 | RolePermissionSeeder | 3 roles + 79 permissions |
| 7 | UserAccountSeeder | 3 demo user accounts |
| 8 | SettingSeeder | Organization + document numbering settings |
| 9 | ContactSeeder | 14 demo contacts (8 customers + 6 vendors) |
| 10 | ItemSeeder | 16 demo items (8 services + 4 non-inventory + 4 inventory) |
| 11 | TransactionSeeder | Demo invoices, bills, payments, and journal entries |

All seeders use `firstOrCreate` or `updateOrCreate` for idempotent re-runs.

---

## Architecture Overview

### Request Flow

```
Browser (React SPA)
    ↓ Inertia XHR (X-Inertia header)
Laravel Router (web.php)
    ↓ auth:sanctum middleware (session-based)
HandleInertiaRequests middleware
    ↓ shares: auth, flash, settings
Form Request (validation + authorization)
    ↓ $request->validated()
Controller (HTTP layer)
    ↓ delegates to service
Service (business logic)
    ↓ model operations + GL entries
Eloquent Models (data layer)
    ↓ Auditable trait (created_by, updated_by)
Database (MySQL/PostgreSQL)
```

### Dual Route Architecture

The application has two route files serving different purposes:

| File | Middleware | Purpose |
|------|-----------|---------|
| `routes/web.php` | `web` + `auth:sanctum` (session) | Inertia SPA — page renders and form submissions |
| `routes/api.php` | `api` + `auth:sanctum` (token) | External API consumers using Bearer tokens |

All Inertia form submissions (POST/PUT/DELETE) go through web routes, not API routes. The web.php file mirrors every CRUD endpoint from api.php that the frontend uses.

### Authentication

- **Session-based** (web routes): Laravel Sanctum with session cookies. Used by the Inertia SPA.
- **Token-based** (API routes): Laravel Sanctum personal access tokens via `Authorization: Bearer {token}`. Used by external API consumers.
- **Self-registration disabled**: User accounts are created only by admins through Settings > Users.

---

## Inertia.js Bridge

### Shared Page Props

`HandleInertiaRequests.php` shares these props on every page:

```typescript
interface PageProps {
    auth: {
        user: {
            id: number;
            name: string;
            email: string;
            roles: Role[];
        };
    } | null;
    flash: {
        success: string | null;
        error: string | null;
    };
    settings: {
        base_currency: string;        // default: 'IDR'
        fiscal_year_start_month: string; // default: 'january'
        accounting_basis: string;     // default: 'accrual'
        organization_name: string;    // default: ''
        date_format: string;          // default: 'dd/MM/yyyy'
        timezone: string;             // default: 'UTC'
    };
}
```

### Page Resolution

Pages are resolved from `resources/js/pages/` using the module path:

```typescript
resolvePageComponent(`./pages/${name}.tsx`, import.meta.glob('./pages/**/*.tsx'))
```

Controllers render pages using `Inertia::render('module/page', $props)`.

### Layout System

Three layouts with different attachment patterns:

| Layout | Applied Via | Purpose |
|--------|------------|---------|
| AppLayout | Persistent `page.layout` static property | Main application shell — sidebar + header |
| AuthLayout | Inline JSX wrapper | Login page centered card |
| PrintLayout | Inline JSX wrapper | Print-only toolbar with back/print buttons |

AppLayout is a **persistent layout**: Inertia reuses the same instance across navigations, preserving sidebar state (accordion group, collapse toggle) without remounting.

---

## URL Masking

The application masks all URL paths in the browser address bar. The address bar always displays `/` regardless of the current page.

### Implementation (`resources/js/app.tsx`)

```
1. Monkey-patch window.history.pushState → always passes '/' as URL
2. Monkey-patch window.history.replaceState → always passes '/' as URL
3. popstate listener → replaces URL with '/' on back/forward
4. Initial page load → masked before Inertia boots
```

### Implications

- `window.location.pathname` always returns `/`
- Components must use `usePage().url` for the actual route path
- `redirect()->back()` is unreliable (Referer header always sends `/`)
- All controllers use explicit `redirect()->route()` instead of `redirect()->back()`

---

## Flash Messages and Toast

### End-to-End Flow

```
Controller →with('success', 'msg')  →  redirect()->route('target')
    ↓
HandleInertiaRequests.share()  →  reads session flash into Inertia props
    ↓
AppLayout.tsx  →  router.on('success') event listener reads flash
    ↓
showToast()  →  ToastContainer renders auto-dismissing notification (4s)
```

The `router.on('success')` pattern (not `useEffect`) is used because AppLayout is persistent — a `useEffect` with `[flash?.success]` would not re-fire for consecutive identical flash messages.

Toast variants: `success` (green), `error` (red), `info` (blue).

---

## Controller Response Pattern

Every controller method handles both Inertia and API requests:

```php
public function store(StoreRequest $request): JsonResponse|RedirectResponse
{
    $record = $this->service->create($request->validated());

    if ($request->header('X-Inertia')) {
        return redirect()->route('target.show', $record->id)
            ->with('success', 'Record created successfully.');
    }

    return response()->json($record, 201);
}
```

### Standard Redirect Targets

| Operation | Target |
|-----------|--------|
| Store (has show page) | Show page of new record |
| Store (no show page) | Module index page |
| Update | Show page or index page |
| Destroy | Module index page |
| Status action | Show page of affected record |
| Duplicate | Show page of new duplicate |

---

## Audit Trail

Every table has three audit columns populated by the `Auditable` trait (`app/Concerns/Auditable.php`):

| Column | Type | Set On |
|--------|------|--------|
| `created_by` | unsignedBigInteger, nullable | Model creating event |
| `updated_by` | unsignedBigInteger, nullable | Model creating + updating events |
| `deleted_by` | unsignedBigInteger, nullable | Model deleting event (before SoftDeletes) |

When no user is authenticated (seeding, console, queue), columns remain `null`.

---

## Soft Delete + Unique Constraint Handling

### Problem

Soft-deleted records occupy unique database indexes (currencies.code, users.email) and application-level unique validation.

### Solution

Two patterns depending on whether a DB-level unique index exists:

**DB-level unique (currencies, users) — Restore-or-Create:**

```
1. Check if non-deleted record exists → throw DuplicateNumberException
2. Check if trashed record exists → restore + update + clear deleted_by
3. Otherwise → create new
```

**Validation-only unique (invoices, bills, etc.) — Allow reuse:**

```
Form Request: Rule::unique('table', 'column')->whereNull('deleted_at')
Service: Model::where('column', $value)->whereNull('deleted_at')->exists()
```

---

## Testing

### Running Tests

```bash
php artisan test                         # All 81 tests
php artisan test --filter=InvoiceTest    # Specific test class
php artisan test --testsuite=Feature     # Feature tests only
php artisan test --testsuite=Unit        # Unit tests only
```

### Test Structure

```
tests/
├── Feature/           — 62 tests across 9 files
│   ├── Auth/          — Authentication + authorization (9 tests)
│   ├── Finance/       — Invoice, payment, credit, bill flows (30 tests)
│   ├── Accounting/    — Journal entries + chart of accounts (13 tests)
│   └── Reports/       — All 7 report types (9 tests)
└── Unit/              — 18 tests across 3 files
    └── Calculations/  — Totals, currency conversion, aging buckets
```

### Factories

17 factories in `database/factories/` support all models used in tests. Key factories: `SaleInvoiceFactory`, `BillFactory`, `ItemEntryFactory`, `PaymentReceiveFactory`, `ManualJournalFactory`.

---

## Key PHP Dependencies

| Package | Purpose |
|---------|---------|
| `laravel/framework` | Core framework (v13.x) |
| `inertiajs/inertia-laravel` | Server-side Inertia adapter |
| `laravel/sanctum` | Authentication (session + token) |
| `spatie/laravel-permission` | Role-based access control |
| `tightenco/ziggy` | Laravel route → JavaScript |

## Key Frontend Dependencies

| Package | Purpose |
|---------|---------|
| `@inertiajs/react` | React adapter for Inertia.js |
| `react` + `react-dom` | UI framework (v19) |
| `tailwindcss` | Utility-first CSS (v4) |
| `lucide-react` | Icon library |
| `@headlessui/react` | Accessible UI primitives |

---

## Environment Configuration

Key `.env` variables:

| Variable | Purpose | Example |
|----------|---------|---------|
| `APP_URL` | Application URL | `http://localhost:8000` |
| `DB_CONNECTION` | Database driver | `mysql` or `pgsql` |
| `DB_DATABASE` | Database name | `erp_finance` |
| `SANCTUM_STATEFUL_DOMAINS` | Cookie auth domains | `localhost:5173,localhost:8000` |
