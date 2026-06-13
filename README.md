# Road Eagles MC Thailand — Club Management App

A private progressive web app (PWA) for managing club operations, finances, merchandise, events and membership for Road Eagles MC Thailand.

## Overview

Single-file HTML application with no server-side dependencies. All data is stored in [Supabase](https://supabase.com) and synced across devices. The app runs in any modern mobile browser and can be installed as a PWA on Android or iOS.

## Features

- **Dashboard** — account balances with month opening figures, stock alerts, tab net position
- **Ledger** — full income/expense tracking across Club and Charity accounts, paid/unpaid toggle
- **Members** — member management, role assignment, annual fee tracking and reset (full fee regardless of join date)
- **Merchandise** — catalogue with member/visitor pricing, stock levels, colour/size variants
- **Tab Ledger** — open merch tabs (edit/delete with stock restore), unpaid transactions, PO outstanding balances
- **Purchase Orders** — full lifecycle with multi-line-item support, production cost estimates, payments, assembly groups (BOM), status override for admin
- **Events** — event management with income/expense tracking
- **Suppliers** — full supplier records
- **Reports**
  - Monthly Statement (month/year selector, opening/closing balances, category subtotals)
  - Annual Summary (opening/closing balances, category subtotals)
  - Club Meeting Report (by month, full year, or custom date range; opening/closing balances; period activity summary)
  - Membership, Merchandise Sales, Stock, Stock Value
  - Donations (split by Club and Charity account)
  - Tab Debts, Purchase Ledger (with committed PO balances), Cost of Goods, Analysis, Audit Trail
  - All reports export to PDF with period label included
- **Categories & Config** — income/expense categories, product categories, event types, sizes, colours, roles, assembly groups
- **Backup & Restore** — JSON export/import with configurable filename prefix and reminder system

## Access & Roles

| Role | Access |
|------|--------|
| Admin | Full access including settings, delete, audit trail, backup, PO status override |
| Officer | Standard operational access |
| Title Only | Display role, no app login |

Access is controlled by 4-digit PIN. Managed by Admin in More → Settings.

## Getting Started

### New Device Setup

1. Open the app URL in Chrome or Brave
2. The app automatically syncs from Supabase before the PIN screen appears — no setup needed
3. Enter your 4-digit PIN
4. Credentials are hardcoded in the app as a fallback

### Updating to a New Version

1. In Chrome: Settings → Privacy → Clear browsing data → **Cached images and files only**
2. Reload the page — the new version loads from GitHub
3. Do **not** clear site data or cookies

> **Note:** Due to file size, uploading via the GitHub web interface may be slow. Use Git on desktop for faster uploads.

### If Login is Blocked

If the app shows a connection error on startup, check your internet connection and tap **↺ Try Again**. Login is intentionally blocked when Supabase cannot be reached — this prevents stale local data being used.

## File Structure

```
road-eagles-mc-final.html   — Live production app
remc-test.html              — Test system (separate Supabase database)
README.md                   — This file
```

## Technical Details

- **Storage**: Supabase PostgreSQL via REST API (`club_data` table, `main` row)
- **Local cache**: Browser localStorage (`remct4` key)
- **Credentials**: Hardcoded in app JS as fallback, also stored in `remct_sync` localStorage key
- **Sync**: On every save, data pushes to Supabase. On app open, data pulls from Supabase before PIN entry is permitted
- **Offline**: App blocks login when Supabase is unreachable — intentional by design

## Test System

The test app (`remc-test.html`) connects to a separate Supabase project (`bgajjoog`) and uses separate localStorage keys (`remct4_test`, `remct_sync_test`). Credentials are hardcoded and locked — live and test data are completely independent.

To initialise the test database, run in the test project SQL editor:
```sql
INSERT INTO club_data (id, data, updated_at)
VALUES ('test_main', '{}'::jsonb, now())
ON CONFLICT (id) DO NOTHING;
```

## Supabase Maintenance

Run annually on both live and test projects (before October 2026):

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON public.club_data TO anon;
ALTER TABLE public.club_data ENABLE ROW LEVEL SECURITY;
SELECT * FROM pg_policies WHERE tablename = 'club_data';
```

## Backup

- Admin only: More → Settings → Backup & Restore
- Configure filename prefix (default: `RE-Backup`) — saved as `<prefix>-YYYYMMDD-HH:MM:SS.json`
- Set reminder interval in days (0 = disabled). Tap **Save & Sync** to push both settings to all devices
- Restore accepts any valid backup JSON file regardless of filename

## Purchase Orders

POs support multiple stock line items per order — useful for ordering multiple shirt sizes or product variants from one supplier in a single PO.

- Add stock items with **+ Add Stock Item** — select product, variant, size, colour and quantity
- Production costs are used to calculate an **estimated total** which can be applied with one tap
- Full lifecycle: Draft → Ordered → Part Paid → Part Rcvd → Received
- Assembly Groups (BOM) link multiple POs to one finished product — stock only added on assembly, not on individual component receipt

## Key Business Rules

- **Membership fees**: Full annual fee applies to all members regardless of join date (no pro-rata)
- **Assembly group POs**: Stock is NOT added when individual components are received — only added via the Assemble step once all components arrive
- **PO status**: A PO with outstanding balance after stock receipt shows as "Part Rcvd" and remains visible in the Tab Ledger until fully paid
- **Tab deletion**: Deleting a tab entry restores stock levels. No financial transaction is created or reversed (tabs are unpaid by definition)
- **Donations**: Split by account — Club donations and Charity donations are reported separately

## Version History

| Version | Notes |
|---------|-------|
| v3.25 | Multi-line-item POs, production cost estimates, baseline v24 |
| v3.22 | Meeting report activity summary grouped by product |
| v3.19 | Remove pro-rata fees, PDF period label fix, baseline v21 |
| v3.17 | Delete tab entries with stock restore |
| v3.14 | Assembly group POs no longer add stock on receipt |
| v3.10 | Admin PO status override, part_received status |
| v3.08 | PO variant stored by ID not index, migration for existing POs |
| v3.00 | Assembly groups config, dynamic stock tabs, tab ledger edit, report opening/closing balances |
| v2.93 | Startup fetch gate, hardcoded credentials, login blocked offline |
| v2.88 | Data loss fix — manualSync only pushes on empty remote |
| v2.76 | Roles system with app access flags |

---

*Road Eagles MC Thailand — Private use only*
