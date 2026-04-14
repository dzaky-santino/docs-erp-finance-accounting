# ERP Finance & Accounting — System Blueprint

**Generated:** 2026-04-02
**Source:** Derived from the actual codebase at commit history on the `main` branch
**Version:** Laravel 13.1.1 + Inertia.js v2 + React 19

---

## Document Index

| # | Document | Description |
|---|----------|-------------|
| 01 | [System Overview](01-system-overview.md) | Architecture, tech stack, modules, and key design decisions |
| 02 | [Database Schema](02-database-schema.md) | Complete schema for all 42+ tables with columns, types, keys, and indexes |
| 03 | [Entity Relationships](03-entity-relationships.md) | All 46 Eloquent models with relationships and the core entity graph |
| 04 | [Business Flows](04-business-flows.md) | Step-by-step flows for every major operation including GL entry logic |
| 05 | [Module Pages Inventory](05-module-pages-inventory.md) | Every frontend page with purpose, data, actions, and access control |
| 06 | [API and Routes](06-api-and-routes.md) | All web and API routes with methods, controllers, and middleware |
| 07 | [Roles and Permissions](07-roles-and-permissions.md) | Complete RBAC system with all 79 permissions across 3 roles |
| 08 | [Integration Guide](08-integration-guide.md) | Developer setup, embedding patterns, and deployment considerations |
| 09 | [Default Data](09-default-data.md) | All seeded data: 42 currencies, 135 accounts, 12 tax rates, demo users |

---

## Recommended Reading Order

### For a Project Manager or Stakeholder
1. **01 — System Overview** — Understand what the system does and how it is built
2. **04 — Business Flows** — Understand every business process the system supports

### For a Backend Developer
1. **01 — System Overview** — Architecture and layered design
2. **02 — Database Schema** — Complete table definitions and constraints
3. **03 — Entity Relationships** — Eloquent models and how they connect
4. **04 — Business Flows** — Service-layer logic and GL entry patterns
5. **06 — API and Routes** — All endpoints and middleware
6. **08 — Integration Guide** — Setup and deployment

### For a Frontend Developer
1. **01 — System Overview** — Inertia.js bridge and URL masking
2. **05 — Module Pages Inventory** — Every page and its functionality
3. **06 — API and Routes** — Route structure and data flow
4. **08 — Integration Guide** — Dev environment setup

### For a DBA
1. **02 — Database Schema** — Complete schema with all constraints
2. **03 — Entity Relationships** — Foreign keys and relationship graph
3. **09 — Default Data** — Seeded data and initialization order

### For a Security Reviewer
1. **06 — API and Routes** — Authentication middleware and route protection
2. **07 — Roles and Permissions** — RBAC matrix and policy enforcement
3. **08 — Integration Guide** — Session handling and deployment security

---

## Notes

- This blueprint reflects the actual codebase at time of generation. It is not a design document — every fact was derived by reading the source code.
- The system is a single-tenant modular monolith. Multi-tenancy is not currently implemented.
- All monetary values use `decimal(15,5)` precision. Exchange rates use `decimal(13,9)`.
- The chart of accounts follows PSAK (Indonesian Financial Accounting Standards) conventions.
