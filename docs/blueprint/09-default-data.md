# 09 — Default Data

All data is derived from reading the seeder files in `database/seeders/`. Every value listed below is the exact value written to the database during `php artisan db:seed`.

---

## Seeding Order and Dependencies

```
CurrencySeeder          — no dependencies
BranchSeeder            — no dependencies
WarehouseSeeder         — no dependencies
AccountSeeder           — no dependencies
TaxRateSeeder           — no dependencies
RolePermissionSeeder    — no dependencies
UserAccountSeeder       — depends on RolePermissionSeeder (roles must exist)
SettingSeeder           — no dependencies
ContactSeeder           — depends on CurrencySeeder (currency_code references)
ItemSeeder              — depends on AccountSeeder + TaxRateSeeder (account/tax FK references)
TransactionSeeder       — depends on ContactSeeder + ItemSeeder + AccountSeeder
```

All seeders use `firstOrCreate` or `updateOrCreate` — safe for repeated runs.

---

## 1. Currencies (43 currencies)

### Southeast Asia
| Name | Code | Symbol |
|------|------|--------|
| Indonesian Rupiah | IDR | Rp |
| Singapore Dollar | SGD | S$ |
| Malaysian Ringgit | MYR | RM |
| Thai Baht | THB | ฿ |
| Philippine Peso | PHP | ₱ |
| Vietnamese Dong | VND | ₫ |

### East Asia
| Name | Code | Symbol |
|------|------|--------|
| Japanese Yen | JPY | ¥ |
| Chinese Yuan | CNY | ¥ |
| South Korean Won | KRW | ₩ |
| Hong Kong Dollar | HKD | HK$ |
| Taiwan Dollar | TWD | NT$ |

### South Asia
| Name | Code | Symbol |
|------|------|--------|
| Indian Rupee | INR | ₹ |
| Pakistani Rupee | PKR | ₨ |
| Bangladeshi Taka | BDT | ৳ |
| Sri Lankan Rupee | LKR | Rs |

### Americas
| Name | Code | Symbol |
|------|------|--------|
| US Dollar | USD | $ |
| Canadian Dollar | CAD | C$ |
| Mexican Peso | MXN | MX$ |
| Brazilian Real | BRL | R$ |
| Argentine Peso | ARS | AR$ |
| Colombian Peso | COP | CO$ |
| Chilean Peso | CLP | CL$ |

### Europe
| Name | Code | Symbol |
|------|------|--------|
| Euro | EUR | € |
| British Pound | GBP | £ |
| Swiss Franc | CHF | CHF |
| Swedish Krona | SEK | kr |
| Norwegian Krone | NOK | kr |
| Danish Krone | DKK | kr |
| Polish Zloty | PLN | zł |
| Czech Koruna | CZK | Kč |
| Turkish Lira | TRY | ₺ |
| Russian Ruble | RUB | ₽ |

### Middle East
| Name | Code | Symbol |
|------|------|--------|
| Saudi Riyal | SAR | ﷼ |
| UAE Dirham | AED | د.إ |
| Qatari Riyal | QAR | ر.ق |
| Kuwaiti Dinar | KWD | د.ك |
| Bahraini Dinar | BHD | د.ب |

### Oceania
| Name | Code | Symbol |
|------|------|--------|
| Australian Dollar | AUD | A$ |
| New Zealand Dollar | NZD | NZ$ |

### Africa
| Name | Code | Symbol |
|------|------|--------|
| South African Rand | ZAR | R |
| Nigerian Naira | NGN | ₦ |
| Egyptian Pound | EGP | E£ |
| Kenyan Shilling | KES | KSh |

### Exchange Rates

Rates are stored as **1 IDR = X foreign currency** (inverse format to fit `decimal(13,9)`).

Base rates used (1 foreign unit = X IDR):

| Currency | IDR per unit | Stored rate (1/X) |
|----------|-------------|-------------------|
| USD | 15,850 | 0.000063091 |
| EUR | 17,200 | 0.000058140 |
| GBP | 20,100 | 0.000049751 |
| JPY | 105.50 | 0.009478673 |
| SGD | 11,900 | 0.000084034 |
| AUD | 10,250 | 0.000097561 |
| CNY | 2,180 | 0.000458716 |
| KWD | 51,600 | 0.000019380 |
| BHD | 42,050 | 0.000023782 |

*(42 rates total, one for each non-IDR currency, dated on seeding day)*

---

## 2. Branch (1 record)

| Name | Code | Primary |
|------|------|---------|
| Head Office | HO | Yes |

---

## 3. Warehouse (1 record)

| Name | Code | Primary |
|------|------|---------|
| Main Warehouse | WH-MAIN | Yes |

---

## 4. Chart of Accounts (95 accounts)

PSAK-compliant Indonesian chart of accounts. All accounts have `is_predefined = true` and `is_active = true`.

### Assets (1xxxx) — 34 accounts

#### Cash & Bank (7 accounts)
| Code | Name | Type | System |
|------|------|------|--------|
| 11001 | Cash on Hand | cash | Yes |
| 11002 | Petty Cash | cash | No |
| 11101 | Bank BCA - IDR | bank | Yes |
| 11102 | Bank Mandiri - IDR | bank | No |
| 11103 | Bank BNI - IDR | bank | No |
| 11104 | Bank BRI - IDR | bank | No |
| 11105 | Bank BCA - USD | bank | No |

#### Accounts Receivable (2 accounts)
| Code | Name | Type | System |
|------|------|------|--------|
| 12001 | Accounts Receivable | accounts-receivable | Yes |
| 12002 | Allowance for Doubtful Accounts | accounts-receivable | No |

#### Inventory (3 accounts)
| Code | Name | Type | System |
|------|------|------|--------|
| 13001 | Inventory - Finished Goods | inventory | Yes |
| 13002 | Inventory - Raw Materials | inventory | No |
| 13003 | Inventory - Work in Progress | inventory | No |

#### Other Current Assets (7 accounts)
| Code | Name | Type |
|------|------|------|
| 14001 | Undeposited Funds | other-current-asset |
| 14002 | Prepaid Expenses | other-current-asset |
| 14003 | Prepaid Insurance | other-current-asset |
| 14004 | Prepaid Rent | other-current-asset |
| 14005 | PPN Masukan (Input VAT) | other-current-asset |
| 14006 | Advances to Employees | other-current-asset |
| 14007 | Deposits Paid | other-current-asset |

#### Fixed Assets (12 accounts)
| Code | Name | Type |
|------|------|------|
| 15001 | Land | fixed-asset |
| 15002 | Buildings | fixed-asset |
| 15003 | Accumulated Depreciation - Buildings | fixed-asset |
| 15004 | Vehicles | fixed-asset |
| 15005 | Accumulated Depreciation - Vehicles | fixed-asset |
| 15006 | Office Equipment | fixed-asset |
| 15007 | Accumulated Depreciation - Office Equipment | fixed-asset |
| 15008 | Computer Equipment | fixed-asset |
| 15009 | Accumulated Depreciation - Computer Equipment | fixed-asset |
| 15010 | Furniture and Fixtures | fixed-asset |
| 15011 | Accumulated Depreciation - Furniture | fixed-asset |

#### Non-Current Assets (3 accounts)
| Code | Name | Type |
|------|------|------|
| 16001 | Goodwill | non-current-asset |
| 16002 | Security Deposits | non-current-asset |
| 16003 | Deferred Tax Assets | non-current-asset |

### Liabilities (2xxxx) — 19 accounts

#### Accounts Payable (1 account)
| Code | Name | Type | System |
|------|------|------|--------|
| 21001 | Accounts Payable | accounts-payable | Yes |

#### Tax Payable (5 accounts)
| Code | Name | Type | System |
|------|------|------|--------|
| 22001 | PPN Keluaran (Output VAT) | tax-payable | Yes |
| 22002 | PPh 21 Payable | tax-payable | No |
| 22003 | PPh 23 Payable | tax-payable | No |
| 22004 | PPh 4(2) Payable | tax-payable | No |
| 22005 | PPh 25/29 Payable | tax-payable | No |

#### Credit Card (1 account)
| Code | Name | Type |
|------|------|------|
| 23001 | Credit Card - BCA | credit-card |

#### Other Current Liabilities (7 accounts)
| Code | Name | Type |
|------|------|------|
| 24001 | Accrued Expenses | other-current-liability |
| 24002 | Accrued Salaries and Wages | other-current-liability |
| 24003 | Revenue Received in Advance | other-current-liability |
| 24004 | Customer Deposits | other-current-liability |
| 24005 | BPJS Ketenagakerjaan Payable | other-current-liability |
| 24006 | BPJS Kesehatan Payable | other-current-liability |
| 24007 | Opening Balance Liabilities | other-current-liability |

#### Long-Term Liabilities (3 accounts)
| Code | Name | Type |
|------|------|------|
| 25001 | Bank Loan - BCA | long-term-liability |
| 25002 | Bank Loan - Mandiri | long-term-liability |
| 25003 | Lease Liability | long-term-liability |

#### Non-Current Liabilities (2 accounts)
| Code | Name | Type |
|------|------|------|
| 26001 | Employee Benefit Obligation | non-current-liability |
| 26002 | Deferred Tax Liabilities | non-current-liability |

### Equity (3xxxx) — 6 accounts

| Code | Name | Type | System |
|------|------|------|--------|
| 31001 | Share Capital | equity | No |
| 31002 | Additional Paid-In Capital | equity | No |
| 32001 | Retained Earnings | equity | Yes |
| 32002 | Current Year Earnings | equity | No |
| 33001 | Opening Balance Equity | equity | No |
| 33002 | Owner's Drawing | equity | No |

### Revenue (4xxxx) — 10 accounts

#### Income (5 accounts)
| Code | Name | Type | System |
|------|------|------|--------|
| 41001 | Sales Revenue - Products | income | Yes |
| 41002 | Sales Revenue - Services | income | No |
| 41003 | Sales Revenue - Subscriptions | income | No |
| 41004 | Sales Discounts | income | No |
| 41005 | Sales Returns and Allowances | income | No |

#### Other Income (5 accounts)
| Code | Name | Type |
|------|------|------|
| 42001 | Interest Income | other-income |
| 42002 | Foreign Exchange Gain | other-income |
| 42003 | Purchase Discounts | other-income |
| 42004 | Gain on Sale of Assets | other-income |
| 42005 | Miscellaneous Income | other-income |

### Cost of Goods Sold (5xxxx) — 4 accounts

| Code | Name | Type | System |
|------|------|------|--------|
| 51001 | Cost of Goods Sold | cost-of-goods-sold | Yes |
| 51002 | Cost of Services | cost-of-goods-sold | No |
| 51003 | Freight and Delivery Costs | cost-of-goods-sold | No |
| 51004 | Purchase Returns and Allowances | cost-of-goods-sold | No |

### Operating Expenses (6xxxx) — 22 accounts

| Code | Name | Type |
|------|------|------|
| 61001 | Salaries and Wages | expense |
| 61002 | Employee Benefits | expense |
| 61003 | BPJS Ketenagakerjaan Expense | expense |
| 61004 | BPJS Kesehatan Expense | expense |
| 61005 | THR (Holiday Allowance) | expense |
| 61006 | Training and Development | expense |
| 62001 | Rent Expense | expense |
| 62002 | Office Supplies | expense |
| 62003 | Utilities (Electricity, Water, Gas) | expense |
| 62004 | Telephone and Internet | expense |
| 62005 | Postage and Courier | expense |
| 62006 | Cleaning and Maintenance | expense |
| 62007 | Security Services | expense |
| 62008 | Printing and Stationery | expense |
| 63001 | Legal and Professional Fees | expense |
| 63002 | Accounting and Audit Fees | expense |
| 63003 | Consulting Fees | expense |
| 63004 | Notary Fees | expense |
| 64001 | Advertising and Promotion | expense |
| 64002 | Entertainment | expense |
| 64003 | Travel Expenses | expense |
| 64004 | Transportation | expense |
| 65001 | Depreciation Expense | expense |
| 65002 | Amortization Expense | expense |
| 66001 | Insurance Expense | expense |
| 66002 | Bank Fees and Charges | expense |
| 66003 | Interest Expense | expense |
| 67001 | Software Subscriptions | expense |
| 67002 | IT Maintenance | expense |
| 67003 | Domain and Hosting | expense |

### Other Expenses (7xxxx) — 6 accounts

| Code | Name | Type |
|------|------|------|
| 71001 | Foreign Exchange Loss | other-expense |
| 71002 | Bad Debt Expense | other-expense |
| 71003 | Loss on Sale of Assets | other-expense |
| 71004 | Penalties and Fines | other-expense |
| 71005 | Donations and Contributions | other-expense |
| 71006 | Miscellaneous Expenses | other-expense |

### System Accounts

These accounts are flagged `is_system_account = true` and cannot be deleted:

| Code | Name | Purpose |
|------|------|---------|
| 11001 | Cash on Hand | Default cash account |
| 11101 | Bank BCA - IDR | Default bank account |
| 12001 | Accounts Receivable | AR system account |
| 13001 | Inventory - Finished Goods | Default inventory account |
| 21001 | Accounts Payable | AP system account |
| 22001 | PPN Keluaran (Output VAT) | Tax payable system account |
| 32001 | Retained Earnings | Equity system account |
| 41001 | Sales Revenue - Products | Default income account |
| 51001 | Cost of Goods Sold | Default COGS account |

---

## 5. Tax Rates (12 rates)

### VAT (PPN)
| Name | Code | Rate | Description |
|------|------|------|-------------|
| PPN 11% | PPN-11 | 11% | Value Added Tax standard rate |
| PPN 12% | PPN-12 | 12% | VAT new rate effective 2025 |
| PPN 1.1% | PPN-1.1 | 1.1% | PPN deemed rate for UMKM (small business) |

### Income Tax Withholding
| Name | Code | Rate | Description |
|------|------|------|-------------|
| PPh 21 - 5% | PPH21-5 | 5% | Employee tax bracket 1 (up to Rp 60M) |
| PPh 21 - 15% | PPH21-15 | 15% | Employee tax bracket 2 (Rp 60-250M) |
| PPh 23 - 2% | PPH23-2 | 2% | Withholding on services, rent, royalties |
| PPh 23 - 15% | PPH23-15 | 15% | Withholding on dividends and interest |
| PPh 4(2) - 10% | PPH42-10 | 10% | Final tax on building rental income |
| PPh 4(2) - 2.5% | PPH42-2.5 | 2.5% | Final tax on construction services |
| PPh 4(2) - 0.5% | PPH42-0.5 | 0.5% | Final tax for UMKM (PP 55/2022) |

### Exemptions
| Name | Code | Rate | Description |
|------|------|------|-------------|
| Tax Exempt | TAX-EXEMPT | 0% | Exempts goods/services from taxes |
| Non-Taxable | NON-TAXABLE | 0% | Items not subject to VAT (basic necessities) |

---

## 6. Roles and Permissions

| Role | Permissions | Description |
|------|------------|-------------|
| admin | All 79 | Full access to everything |
| staff | 42 | Create/edit/view on Finance & Accounting, view-only elsewhere, no delete |
| viewer | 20 | Read-only (*.view) on all modules |

See [07-roles-and-permissions.md](07-roles-and-permissions.md) for the complete permission matrix.

---

## 7. Demo Users (3 accounts)

| Name | Email | Password | Role |
|------|-------|----------|------|
| Administrator | admin@erp.test | password | admin |
| Staff User | staff@erp.test | password | staff |
| Viewer User | viewer@erp.test | password | viewer |

---

## 8. Organization Settings

### Organization Group
| Key | Value |
|-----|-------|
| accounting_basis | accrual |
| base_currency | IDR |
| language | id |
| date_format | dd/MM/yyyy |
| fiscal_year | january |
| timezone | Asia/Jakarta |

### Account Settings
| Key | Value |
|-----|-------|
| account_code_unique | 1 |

### Document Numbering

| Document | Prefix | Starting Number | Auto-Increment |
|----------|--------|----------------|----------------|
| Manual Journals | J- | 00001 | Yes |
| Sales Invoices | INV- | 00001 | Yes |
| Sales Receipts | REC- | 00001 | Yes |
| Sales Estimates | EST- | 00001 | Yes |
| Payment Receives | PAY- | 00001 | Yes |
| Bills | BILL- | 00001 | Yes |
| Bill Payments | BP- | 00001 | Yes |
| Credit Notes | CN- | 00001 | Yes |
| Vendor Credits | VC- | 00001 | Yes |
| Cashflow | CF- | 00001 | Yes |
| Warehouse Transfers | WT- | 00001 | Yes |

### Item Account Preferences
| Key | Value (Account Code) |
|-----|---------------------|
| preferred_sell_account | 50001 |
| preferred_cost_account | 40002 |
| preferred_inventory_account | 10008 |

---

## 9. Demo Contacts (14 contacts)

### Customers (8)

| Display Name | Contact Person | City | Currency |
|-------------|---------------|------|----------|
| PT Maju Bersama Indonesia | Budi Santoso | Jakarta Selatan | IDR |
| CV Teknologi Nusantara | Dewi Anggraini | Jakarta Selatan | IDR |
| PT Solusi Digital Pratama | Andi Wijaya | Surabaya | IDR |
| PT Karya Mandiri Sejahtera | Siti Rahayu | Bandung | IDR |
| PT Global Mitra Abadi | Rizki Pratama | Jakarta Selatan | IDR |
| PT Indo Kreatif Media | Mega Putri | Jakarta Selatan | IDR |
| Hendra Kurniawan | Hendra Kurniawan | Jakarta Barat | IDR |
| PT Sinar Jaya Logistik | Agus Suryanto | Surabaya | IDR |

### Vendors (6)

| Display Name | Contact Person | City | Currency |
|-------------|---------------|------|----------|
| PT Sumber Makmur Perkasa | Wahyu Hidayat | Jakarta Pusat | IDR |
| CV Aneka Supplies | Lina Marlina | Jakarta Utara | IDR |
| PT Jaya Komputer Indonesia | Roni Susanto | Bandung | IDR |
| PT Abadi Furniture | Yanti Kusuma | Surabaya | IDR |
| PT Telekomunikasi Mandiri | Dimas Prayogo | Jakarta Selatan | IDR |
| CV Berkat Jaya Rent | Eko Prasetyo | Jakarta Barat | IDR |

---

## 10. Demo Items (16 items)

### Services (8 items)

| Code | Name | Sell Price (IDR) | Tax |
|------|------|-----------------|-----|
| SVC-001 | IT Consulting - Senior | 2,500,000 | PPN 11% |
| SVC-002 | IT Consulting - Junior | 1,500,000 | PPN 11% |
| SVC-003 | Web Development | 35,000,000 | PPN 11% |
| SVC-004 | Mobile App Development | 50,000,000 | PPN 11% |
| SVC-005 | System Maintenance | 5,000,000 | PPN 11% |
| SVC-006 | UI/UX Design | 15,000,000 | PPN 11% |
| SVC-007 | Cloud Server Hosting | 3,000,000 | PPN 11% |
| SVC-008 | Training and Workshop | 7,500,000 | PPN 11% |

### Non-Inventory Products (4 items)

| Code | Name | Sell Price (IDR) | Cost Price (IDR) | Tax |
|------|------|-----------------|-----------------|-----|
| PRD-001 | Software License - ERP Module | 25,000,000 | 10,000,000 | PPN 11% |
| PRD-002 | Software License - CRM | 18,000,000 | 7,500,000 | PPN 11% |
| PRD-003 | Domain Registration | 250,000 | 150,000 | PPN 11% |
| PRD-004 | SSL Certificate | 1,500,000 | 750,000 | PPN 11% |

### Inventory Items (4 items)

| Code | Name | Sell Price (IDR) | Cost Price (IDR) | QoH | Tax |
|------|------|-----------------|-----------------|-----|-----|
| INV-001 | Laptop - Business Series | 15,000,000 | 11,000,000 | 25 | PPN 11% |
| INV-002 | Network Switch 24-Port | 4,500,000 | 3,200,000 | 15 | PPN 11% |
| INV-003 | UPS 1500VA | 3,500,000 | 2,500,000 | 20 | PPN 11% |
| INV-004 | Server Rack 42U | 12,000,000 | 8,500,000 | 5 | PPN 11% |

### Account Mappings

| Item Type | Sell Account | Cost Account | Inventory Account |
|-----------|-------------|-------------|-------------------|
| Services | 41002 (Sales Revenue - Services) | 51002 (Cost of Services) | — |
| Non-Inventory | 41001 (Sales Revenue - Products) | 51001 (COGS) | — |
| Inventory | 41001 (Sales Revenue - Products) | 51001 (COGS) | 13001 (Inventory - Finished Goods) |

---

## 11. Demo Transactions

The `TransactionSeeder` creates sample financial documents including invoices, bills, payments, and journal entries to provide a realistic demo environment. This seeder runs last because it depends on contacts, items, and accounts being already seeded.
